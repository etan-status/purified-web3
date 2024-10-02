| | | Tests | [Nimbus](https://github.com/status-im/nimbus-eth2) | [EthereumJS](https://github.com/ethereumjs/ethereumjs-monorepo) | [Devnet](./network_params_fusaka-light.yaml) | [Helios](https://github.com/a16z/helios) |
| - | - | :-: | :-: | :-: | :-: | :-: |
| 🐣 | **Consensus light client data**
|| [Altair light client](https://github.com/ethereum/consensus-specs/blob/dev/specs/altair/light-client/sync-protocol.md) | [🔗](https://github.com/ethereum/consensus-specs/tree/dev/tests/formats/light_client) | ✅ | n/a | ✅ | ✅ |
|| Trusted block root in consensus network config | ❌ | ❌ | n/a | ❌ | ❌ |
|| Historical light client data API | ❌  | ❌ | n/a | ❌ | ❌ |
|| [EIP-7658: Light client data backfill](https://eips.ethereum.org/EIPS/eip-7658) | ❌ | ❌ | n/a | ❌ | n/a |
| 🦒 | **Provable on-chain data**
|| [EIP-7495: SSZ StableContainer](https://eips.ethereum.org/EIPS/eip-7495) | [🔗](https://github.com/ethereum/consensus-specs/pull/3777) | ✅ | ✅ | ✅ | ❌ |
|| ↑ Tagged `Profile` support?
|| [EIP-7688: Forward compatible consensus data structures](https://eips.ethereum.org/EIPS/eip-7688) | [🔗](https://github.com/ethereum/consensus-specs/pull/3844) | ✅ | n/a | ✅ | ❌ |
| 🪓 | **MPT removal**
|| [EIP-6404: SSZ Transactions](https://eips.ethereum.org/EIPS/eip-6404) | [🔗](https://github.com/etan-status/latest_fork_tests/commit/eip-6404) | ✅ | ✅ | ✅ | ❌ |
|| [EIP-6466: SSZ Receipts Root](https://eips.ethereum.org/EIPS/eip-6466) | ❌ | n/a | ❌ | ❌ | ❌ |
|| ↑ [EIP-7706: Separate gas type for calldata](https://eips.ethereum.org/EIPS/eip-7706)
|| ↑ [Remove logs bloom from receipts](./el_receipt.md)
|| [SSZ Transaction / Receipt proofs](https://github.com/ethereum/EIPs/pull/8884) | [🔗](https://github.com/ethereum/EIPs/blob/737c2c2ec68715a07534318aa67a21bd907e81ec/EIPS/eip-%23%23%23%23.md#test-cases) | n/a | ✅ | ✅ | ❌ |
|| [EIP-6465: SSZ Withdrawals Root](https://eips.ethereum.org/EIPS/eip-6465) | ❌ | n/a | ❌ | ❌ | ❌ |
|| [SSZ Requests Root](https://eips.ethereum.org/EIPS/eip-7688) | ❌ | ❌ | ❌ | ❌ | ❌ |
|| [EIP-6493: SSZ Transaction Signature Scheme](https://eips.ethereum.org/EIPS/eip-6493) | ❌ | n/a | ❌ | ❌ | ❌ |
|| ↑ [EIP-7702: Set EOA account code](https://eips.ethereum.org/EIPS/eip-7702)
| 💳 | **<nobr>Verifiable transaction history</nobr>**
|| [EIP-7708: ETH transfers emit a log](https://eips.ethereum.org/EIPS/eip-7708) | ❌ | n/a | ❌ | ❌ | ❌ |
|| ↑ Block rewards
|| ↑ Withdrawals
|| ↑ Fees
|| ↑ 0 ETH transfers
|| [Parallel post-state roots (in state trie)](./el_logs.md) | ❌ | n/a | ❌ | ❌ | ❌ |
|| ↑ JSON-RPC API for verifiable logs enumeration
|| [Parallel post-state roots (full IVC)](https://notes.ethereum.org/@vbuterin/parallel_post_state_roots) | ❌ | n/a | ❌ | ❌ | ❌ |
| 🚀 | **CL/EL performance**
|| SSZ Block Header | ❌ | ❌ | ❌ | ❌ | ❌ |
|| ↑ [Remove logs bloom from block header](./el_block_hash.md)
|| ↑ Use block header root in ePBS
|| SSZ Engine API | ❌ | ❌ | ❌ | ❌ | n/a |
| ⚙️ | **API optimizations**
|| Single roundtrip `eth_call` with proofs | ❌ | n/a | ❌ | ❌ | ❌ |
|| [SSZ query language](https://hackmd.io/@etan-status/electra-lc#SSZ-query-language) | ❌ | ❌ | ❌ | ❌ | ❌ |

[Telegram](https://t.me/+ZJqjzyCQWB8xNzE0)
