```python
class BlockMeta(Container):
    timestamp: uint64
    root: Root

class LogMeta(Container):
    block: BlockMeta
    transaction_root: Root  # All zero for system logs

class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]  # Should we change this to 4 topic fields and make the entire thing stablecontainer for extensibility?
    data: ByteList[MAX_LOG_DATA_SIZE]

class AccumulatedLog(Container):
    meta: LogMeta
    log_root: Root

LOG_CONTRACT_ADDRESS = ExecutionAddress()  # A contract with 3x `mapping` and no code

LOG_ADDRESS_STORAGE_SLOT = 0  # mapping(address => bytes32)
LOG_TOPICS_STORAGE_SLOT = 1  # mapping(bytes32 => bytes32)
LOG_ADDRESS_TOPICS_STORAGE_SLOT = 2  # mapping(bytes32 => bytes32)

def accumulate_log(evm: Evm, log_root: Bytes32, key: Bytes32):
    root = hashlib.sha256()
    root.update(log_root)  # Cachable across invocations with same `log_root`
    root.update(sload(evm.env.state, LOG_CONTRACT_ADDRESS, key))  # Cacheable account lookup in trie
    sstore(evm.env.state, LOG_CONTRACT_ADDRESS, key, root.digest())

def track_log(evm: Evm, log: AccumulatedLog) -> None:
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

`eth_getLogs` response data is extended with `blockTimestamp`.

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
- 0-ETH calls (a transaction that sends 0 ETH to a contract should still have an entry in the transaction history to reflect that the account was active)

---

Transaction fees are extended to emit logs:

- `address`: `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`)
- `topics[0]`: `keccak256('Fee(address,uint256)')`
- `topics[1]`: `from` address (zero prefixed to fill uint256)
- `data`: `amount` in Wei (uint256)

Logs are emitted on:

- Gas fees / refunds (logs for fees have their `to` set to `SYSTEM_ADDRESS`)

---

The concept of system logs is introduced, by extending the EL block header with a new field `systemLogsRoot`, which is defined as `hash_tree_root(List[Log, MAX_LOGS_PER_RECEIPT])`.

Logs are added to the system logs list under the following conditions:

When crediting priority fees:

- `address`: `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`)
- `topics[0]`: `keccak256('PriorityRewards(address,uint256)')`
- `topics[1]`: `to` address (zero prefixed to fill uint256)
- `data`: `amount` in Wei (uint256)

When crediting a withdrawal:

- `address`: `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`)
- `topics[0]`: `keccak256('Withdrawal(address,uint256)')`
- `topics[1]`: `to` address (zero prefixed to fill uint256)
- `data`: `amount` in Wei (uint256)

When generating a genesis block:

- `address`: `0xfffffffffffffffffffffffffffffffffffffffe` (`SYSTEM_ADDRESS`)
- `topics[0]`: `keccak256('Genesis(address,uint256)')`
- `topics[1]`: `to` address (zero prefixed to fill uint256)
- `data`: `amount` in Wei (uint256)

The CL `ExecutionPayload` structure is also extended with `system_logs_root`, which also affects light client header data.
