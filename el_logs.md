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

    accumulate_log(evm, log_root, keccak256(abi.encode(log.address, LOG_ADDRESS_STORAGE_SLOT)))
```

Flow is then, for `eth_getLogs(address, topics, fromBlock, toBlock)`:

- Assumption: Client application knows `fromBlock` and `toBlock` hash
- Client application obtains `fromBlock` and `toBlock` headers and validates them via the known hashes
- Client application obtains `fromBlock`'s `parentBlock` header and validates it via `fromBlock`'s `parentHash`
- Client application obtains historical log accumulators at `parentBlock` and `toBlock` using `eth_getProof` using the storage slots matching the requested logs filter, and verifies the result against `parentBlock`'s and `toBlock`'s `stateRoot`
- Client application now processes `eth_getLogs` result, locally updating the log accumulator using `accumulate_log` compatible logic
- After applying all the logs, client application verifies that its local log accumulator matches the expected value of `toBlock`'s post-state. This ensures that all logs were received without gaps

For plain ETH transfers, additional logs have to be accumulated and synthesized in `eth_getLogs`, that do not necessarily have associated `Receipt`.

- Genesis balances (have to initialize the log accumulator contract with the synthetic genesis logs)
- Block rewards (maybe can look how Bitcoin does it and add a transaction with no sender signature)
- Withdrawals (similar to block rewards, they generate new coins into the EL)
- Internal transactions triggering ETH transfers (delegatecall / call --> maybe simply emit a regular `Log` into the `Receipt`, could it be wasteful on storage? alternatively just add into the accumulator without a `Log`)
- Gas fees / refunds (the `Receipt` already allows computing this together with the `BlockHeader`, so not needed to duplicate in the `Receipt`, but a synthetic `Log` has to be accumulated for `eth_getLogs` response -- or, just add a `Log` to the `Receipt` if storage is alright for that)
- 0-ETH calls (a transaction that sends 0 ETH to a contract should still have an entry in the transaction history, it's something that happened to the account)

All synthetic logs have to be computable from the full `ExecutionBlock` body (+ `Transaction` / `Receipt` / `Withdrawal` / `ExecutionRequests`). They also have to be included in `eth_getLogs` response. If a design is possible that creates `Receipt` for it, we may need additional fake transactions for newly minted coins, or additional block fields. The scheme still works as long as the synthetic logs are just accumulated in the contract and provided in `eth_getLogs` though.

The synthetic logs should have `address` set to a `SYSTEM_ADDRESS` that cannot be controlled by an external entity, as the `address` field is the only one that is not fakeable. If it makes sense, reuse same address as used for `WithdrawalRequest` / `ConsolidationRequest`, and have sensible topics that follow state of the art ERC standards. As in, the first topic should be based on an event signature, the account address has to be indexed, and the amount also needs to be included (un-indexed should be fine, just follow the same rules as the tokens for simplicity and consistency).

To support parallel transaction execution, the log accumulators should probably be updated _after_ all transactions have been executed. Otherwise transactions that share a topic (e.g., `Transfer`) become mutually exclusive and have to be sequentialized. This is in line with current mainnet where receipts only become available _after_ the state root has been updated, but may need additional bookkeeping to track the ETH transfers from internal calls / delegatecalls.
