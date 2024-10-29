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

- Gas fees / refunds
