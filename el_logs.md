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
