```python
class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]  # Should we change this to 4 topic fields and make the entire thing stablecontainer for extensibility?
    data: ByteList[MAX_LOG_DATA_SIZE]

LOG_CONTRACT_ADDRESS = ExecutionAddress()  # A contract with 3x `mapping` and no code

LOG_ADDRESS_STORAGE_SLOT = 0  # mapping(address => bytes32)
LOG_TOPICS_STORAGE_SLOT = 1  # mapping(bytes32 => bytes32)
LOG_ADDRESS_TOPICS_STORAGE_SLOT = 2  # mapping(bytes32 => bytes32)

def accumulate_log(evm: Evm, log_root: Bytes32, key: Bytes32):
    root = hashlib.sha256()
    root.update(log_root)  # Cachable across invocations with same `log_root`
    root.update(sload(evm.env.state, LOG_CONTRACT_ADDRESS, key))  # Cacheable account lookup in trie
    sstore(evm.env.state, LOG_CONTRACT_ADDRESS, key, root.digest())

def track_log(evm: Evm, log: Log) -> None:
    log_root = log.hash_tree_root()

    # Allow eth_getLogs proof via `address` filter
    key = keccak256(abi.encode(log.address, LOG_ADDRESS_STORAGE_SLOT))  # abi.encode(address) != htr...
    accumulate_log(evm, log_root, key)

    for topic in log.topics:
        # Allow eth_getLogs proof via `topics` filter
        key = keccak256(abi.encode(topic, LOG_TOPICS_STORAGE_SLOT))
        accumulate_log(evm, log_root, key)

        # Allow eth_getLogs proof via combined `address` + `topics` filter
        key = keccak256(abi.encode(log.address, topic))  # Partially cacheable hash / same prefix
        key = keccak256(abi.encode(key, LOG_ADDRESS_TOPICS_STORAGE_SLOT))
        accumulate_log(evm, log_root, key)
```

To support parallel transaction execution, the log accumulators should probably be updated _after_ all transactions have been executed. Otherwise transactions that share a topic (e.g., `Transfer`) become mutually exclusive and have to be sequentialized. This is in line with current mainnet where receipts only become available _after_ the state root has been updated, but may need additional bookkeeping to track the ETH transfers from internal calls / delegatecalls.

Flow is then, for `eth_getLogs(address, topics, fromBlock, toBlock)`:

- Assumption: Client application knows `fromBlock` and `toBlock` hash
- Client application obtains `fromBlock` and `toBlock` headers and validates them via the known hashes
- Client application obtains `fromBlock`'s `parentBlock` header and validates it via `fromBlock`'s `parentHash`
- Client application obtains historical log accumulators at `parentBlock` and `toBlock` using `eth_getProof` using the storage slots matching the requested logs filter, and verifies the result against `parentBlock`'s and `toBlock`'s `stateRoot`
- Client application now processes `eth_getLogs` result, locally updating the log accumulator using `accumulate_log` compatible logic
- After applying all the logs, client application verifies that its local log accumulator matches the expected value of `toBlock`'s post-state. This ensures that all logs were received without gaps

---

ETH balance changes are extended to emit logs:

- `address`: `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`)
- `topics[0]`: `keccak256('Transfer(address,address,uint256)')`
- `topics[1]`: `from` address (zero prefixed to fill uint256)
- `topics[2]`: `to` address (zero prefixed to fill uint256)
- `data`: `amount` in Wei (uint256)

Logs are emitted on:

- Regular ETH transfer from one account to another
- Sending ETH to a contract
- Internal transactions triggering ETH transfers (delegatecall / call)
- Gas fees / refunds (logs for fees have their `to` set to `SYSTEM_ADDRESS`)
- 0-ETH calls (a transaction that sends 0 ETH to a contract should still have an entry in the transaction history to reflect that the account was active)

---

System transactions are introduced to mint new ETH:

- When a `TransactionPayload` has `max_fees_per_gas` and `max_priority_fees_per_gas` set to `None`, then while executing the transaction, no gas fee is charged.

- When a `TransactionPayload` has `input_: Optional[ByteList[MAX_CALLDATA_SIZE]]` set to `None`, then while executing the transaction, only the balance is transferred but no code is called. Destination code is only called if `input_` is set to non-`None`, including `[]`.

- When an `ExecutionSignature` has no active fields (i.e., `secp256k1` set to `None`), the transaction is assumed to be a system transaction, originating from `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`).

- Before executing a transaction originating from `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`), the balance of `SYSTEM_ADDRESS` is set to `tx.payload.value`.

```python
class SystemExecutionSignature(Profile[ExecutionSignature]):
    pass  # No `secp256k1` signature

class MintTransactionPayload(Profile[TransactionPayload]):
    chain_id: ChainId
    nonce: uint64
    gas: uint64
    to: ExecutionAddress
    value: uint256

class MintTransaction(Container):
    payload: MintTransactionPayload
    signature: SystemExecutionSignature
```

ETH can then be minted with:

```python
MintTransaction(
    payload=MintTransactionPayload(
        chain_id=CHAIN_ID,
        nonce=SYSTEM_ADDRESS.nonce,
        gas=21000,
        to=DESTINATION_ADDRESS,
        value=amount,
    ),
    signature=SystemExecutionSignature(),
)
```

Note that `MintTransaction` transfers ETH and, therefore, also emits logs as defined above in its receipt.

The following changes are applied to block production:

- When generating a genesis block, a `MintTransaction` is put into the `transactions` list and processed for each initial balance, similar to https://etherscan.io/txs?block=0
- Instead of crediting block rewards, a `MintTransaction` is inserted into the `transactions` list and processed.
- Instead of processing a withdrawal, a `MintTransaction` is inserted into the `transactions` list and processed.

As part of `newPayload` validation on engine API, it is checked that inserted `MintTransaction` appear at their expected locations (for the `fee_recipient` and `withdrawals`), and that no extra `MintTransaction` is present. The CL could also do this check, for now let's keep it in the EL.
