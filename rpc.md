# Network config

| Config | Value |
| - | - |
| Meta | https://ssz-devnet-0.ethpandaops.io |
| CL RPC | https://bn.bootnode-1.ssz-devnet-0.ethpandaops.io |
| EL RPC | https://rpc.bootnode-1.ssz-devnet-0.ethpandaops.io |
| config.yaml | https://config.ssz-devnet-0.ethpandaops.io/cl/config.yaml |
| genesis.ssz | https://config.ssz-devnet-0.ethpandaops.io/cl/genesis.ssz |
| genesis.json | https://config.ssz-devnet-0.ethpandaops.io/el/genesis.json |

# General

The Fusaka-Light devnet builds on top of Prague/Electra. All features are enabled with `ELECTRA_FORK_EPOCH` / at the prague timestamp. This is the scope of initial devnet:

# [EIP-7495: SSZ StableContainer](https://eips.ethereum.org/EIPS/eip-7495)

This is a new SSZ type of which Merkleization must be supported to verify proofs on Fusaka-Light.

- **Merkleization limits**: Instead of determining tree size based on number of fields, the design space limit is indicated explicitly in the type definition, e.g., `StableContainer[16]` always is hashed as if there were 16 leaves, even if there are only 4 fields.
- **Merkleization mix-in**: In `StableContainer` all fields are optional. There is an additional mix-in for a `Bitvector[N]` that indicates which of the fields are actually in use.
- **Profile**: `Profile` describes a specific configuration of required / excluded fields within `StableContainer`. It is only relevant for type safety and for serialization, but does not affect Merkleization. Can be ignored for the initial devnet.
- **Implementations**: [https://stabilitynow.box](https://stabilitynow.box) contains a list of SSZ implementations in various programming languages that support `StableContainer`.

# [EIP-7688: Forward compatible consensus data structures](https://eips.ethereum.org/EIPS/eip-7688)

This adopts `StableContainer` for CL data structures and affects the proofs for light client data. Proofs are a bit longer, using a scheme similar to the one used for Electra, as generalized indices change.

- **Tests**: The light client tests were updated for this EIP and are linked in the [PR description](https://github.com/ethereum/consensus-specs/pull/3844). They follow the usual [test format](https://github.com/ethereum/consensus-specs/tree/dev/tests/formats/light_client), same as mainnet.

# [EIP-6404: SSZ Transactions](https://eips.ethereum.org/EIPS/eip-6404)

Transactions now use SSZ as their on-chain representation. To verify them, initialize a `Transaction` SSZ `StableContainer` from the JSON-RPC, and then `hash_tree_root` it to obtain `transactionRoot`.

The following JSON-RPC API methods are extended with an `inclusionProof` object in their response:

- eth_getTransactionByHash
- eth_getTransactionByBlockHashAndIndex
- eth_getTransactionByBlockNumberAndIndex

`inclusionProof` contains:

- `transactionsRoot`: The `transactionsRoot` from the execution block header corresponding to `blockHash`. Verifying client implementations MUST check that this value matches the block header data.
- `transactionRoot`: The [`hash_tree_root`](https://github.com/ethereum/consensus-specs/blob/ef434e87165e9a4c82a99f54ffd4974ae113f732/ssz/simple-serialize.md) of the transaction object. Verifying client implementations MUST check that JSON-RPC response data matches this root when converted to the [`Transaction`](./eip-6404.md#transaction-container) SSZ representation.
- `merkleBranch`: An [SSZ Merkle proof](https://github.com/ethereum/consensus-specs/blob/ef434e87165e9a4c82a99f54ffd4974ae113f732/ssz/merkle-proofs.md) that proofs `transactionRoot` to be located at `transactionIndex` within `transactionsRoot`. Verifying client implementations MUST check correctness of this proof. The proof can be verified using [`is_valid_merkle_branch`](https://github.com/ethereum/consensus-specs/blob/ef434e87165e9a4c82a99f54ffd4974ae113f732/specs/phase0/beacon-chain.md#is_valid_merkle_branch).

[https://eth-light.xyz](https://eth-light.xyz) offers a viewer to inspect the SSZ representation of any Ethereum mainnet transaction.

✅ Milestone reached: Verifiable transactions

# [EIP-6466: SSZ Receipts](https://eips.ethereum.org/EIPS/eip-6466)

A couple extras that are pending: [https://github.com/ethereum/EIPs/pull/8939/files](https://github.com/ethereum/EIPs/pull/8939/files)

Similar to the transactions, but for receipts.

- There is an extra key `authorities` in the JSON-RPC response that reflects the list of EIP-7702 SetCode authorities. For the successful ones, it will be the authority signer address. For the failed / skipped ones, it will be the zero address. The key is only there for SetCode transactions.
- `cumulative_gas_used` is dropped from the on-chain data; instead, `gas_used` of the individual transaction is there.
- `logs_bloom` is dropped without replacement

Proofs are on eth_getTransactionReceipt JSON-RPC, follow the same `inclusionProof` format as for transactions, but use `receiptRoot` / `receiptsRoot` / `merkleBranch`.

Helios needs to construct the SSZ `Receipt` `StableContainer` from JSON data, `hash_tree_root` it then compare with `receiptRoot`. Check that `receiptRoot` is at `transactionIndex` within `receiptsRoot`. And that `receiptsRoot` matches the `blockHash`.

[https://eth-light.xyz](https://eth-light.xyz) offers a viewer to inspect the SSZ representation of any Ethereum mainnet receipt.

✅ Milestone reached: Verifiable receipts

# [EIP-6465: SSZ Withdrawals Root](https://eips.ethereum.org/EIPS/eip-6465)

Withdrawals are changed to SSZ, this may affect block header validation if `withdrawals` are checked against `withdrawals_root`.

# [EIP-7668: Remove bloom filters](https://eips.ethereum.org/EIPS/eip-7668)

Bloom filters for logs are removed as they are not practically useable with their high false positivity rate.

# [SSZ Block Header](https://fusaka-light.box/el_block_hash.html)

The block hash is now computed as SSZ `hash_tree_root`. Logs Bloom and PoW fields are gone. Fees are now tracked as a multidimensional fee vector. This affects block header validation.

✅ Milestone reached: Verifiable block headers (beyond what is implicitly trusted from LC protocol)

# [EIP-7708: ETH transfers emit a log](https://eips.ethereum.org/EIPS/eip-7708)

There are new logs for plain ETH transfers, fee burns, priority fee payments, block rewards, withdrawals and 0-ETH calls. For basic transfers, the new logs take a similar amount of disk space as the Bloom filter used to fill. Further improvements could consolidate the fee burn + priority fee payment to improve parallel transaction execution.

# [Logs IVC](https://fusaka-light.box/el_logs.html)

This is a partial implementation of [Vitalik's idea](https://notes.ethereum.org/@vbuterin/parallel_post_state_roots). Instead of adding the external zk infrastructure, we put everything into EL state.

There is a system contract that stores all incremental log accumulator states in mappings. Mappings exist for lookups by address, by topic, and by address+topic. These are the same filters supported by eth_getLogs.

To verify, Helios needs to obtain the state of the accumulator state of the past, it can also cache this whenever it syncs logs. Or have the client application pass the old cached state, if historical state is difficult to query. This is done with eth_getProof and verified against a trusted old block's parent state.

Then, run eth_getLogs and locally update the accumulators with the processed logs. If the result matches against the on-chain accumulator state (once more, eth_getProof), the logs can be trusted to be correct without extras or missing ones.

✅ Milestone reached: Verifiable logs
