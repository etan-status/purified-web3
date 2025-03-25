### 📽️ [EthereumZuri.ch 2025 talk](https://www.youtube.com/watch?v=uoCNpufygBE) ([Slides PDF](./Slides/2025-EthereumZuri.ch.pdf), [PPT](./Slides/2025-EthereumZuri.ch.pptx))
### 🔥 [Devnet with public RPC](https://ssz-devnet-0.ethpandaops.io)
### 📱 [Verifying wallet guide](./app.md)
### 🔮 [Web3 purifier guide](./rpc.md)
### 💬 [Discord](https://discord.gg/xUmjdjzMNY)
### 📠 [Telegram](https://t.me/+ZJqjzyCQWB8xNzE0)

|| EIP| Tests | [Nimbus](https://github.com/status-im/nimbus-eth2) | [EthJS](https://github.com/ethereumjs/ethereumjs-monorepo) | [Geth](https://github.com/ethereum/go-ethereum) | [Helios](https://github.com/a16z/helios/tree/ssz-devnet) |
| - | - | :-: | :-: | :-: | :-: | :-: |
| 🌲 | [**<nobr>SSZ (Simple Serialize)</nobr>**](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md)
|| [EIP-7916: SSZ ProgressiveList](https://eips.ethereum.org/EIPS/eip-7916) | ❌ | ❌ | ❌ | ✅ | ❌ |
|| [EIP-7495: SSZ StableContainer](https://eips.ethereum.org/EIPS/eip-7495) | [🔗](https://github.com/ethereum/consensus-specs/pull/3777) | ✅ | ✅ | ❌ | ✅ |
|| [EIP-7688: Forward compatible consensus data structures](https://eips.ethereum.org/EIPS/eip-7688) | [🔗](https://github.com/ethereum/consensus-specs/pull/3844) | ✅ | n/a | n/a | ✅ |
| 💳 | [**<nobr>PURGE → LOG reform</nobr>**](https://vitalik.eth.limo/general/2024/10/26/futures5.html)
|| [EIP-7668: Remove bloom filters](https://eips.ethereum.org/EIPS/eip-7668) (see SSZ receipts / blocks)
|| [EIP-7708: ETH transfers emit a log](https://eips.ethereum.org/EIPS/eip-7708) | ❌ | n/a | ❌ | ❌ | ❌ |
|| ↑ [Addon with more details](https://github.com/ethereum/EIPs/pull/9003/files)
|| [EIP-7799: System logs](https://eips.ethereum.org/EIPS/eip-7799) | ❌ | n/a | ✅ | ❌ | ❌ |
|| [EIP-7792: Verifiable logs](https://eips.ethereum.org/EIPS/eip-7792) (+ proper IVC with tee/snark) | ❌ | n/a | ✅ | ❌ | ❌ |
|| [EIP-7745: Two dimensional log filter data structure](https://eips.ethereum.org/EIPS/eip-7745) ([Video](https://github.com/etan-status/purified-web3/releases/download/2025-portal/2025-EIP-7745-Intro.mp4)) | ❌ | n/a | ❌ | ✅ | ❌ |
| 🪓 | [**<nobr>PURGE → Remove old tx types</nobr>**](https://vitalik.eth.limo/general/2024/10/26/futures5.html)
|| [EIP-7706: Separate gas type for calldata](https://eips.ethereum.org/EIPS/eip-7706) (multidim concept)
|| [EIP-6404: SSZ transactions](https://eips.ethereum.org/EIPS/eip-6404) | [🔗](https://github.com/etan-status/latest_fork_tests/commit/eip-6404) | ✅ | ✅ | ❌ | ✅ |
|| [EIP-6466: SSZ receipts](https://eips.ethereum.org/EIPS/eip-6466) | ❌ | n/a | ✅ | ❌ | ❌ |
|| ↑ [Addon for transaction / receipt proofs](https://github.com/ethereum/EIPs/pull/8884/files) | [🔗](https://github.com/ethereum/EIPs/blob/737c2c2ec68715a07534318aa67a21bd907e81ec/EIPS/eip-%23%23%23%23.md#test-cases) | n/a | ✅ | ❌ | ❌ |
|| [EIP-6493: SSZ transaction signature scheme](https://eips.ethereum.org/EIPS/eip-6493) | ❌ | n/a | ❌ | ❌ | ❌ |
| ☯️ | [**<nobr>PURGE → Serialization harmonization</nobr>**](https://vitalik.eth.limo/general/2024/10/26/futures5.html)
|| [EIP-6465: SSZ withdrawals root](https://eips.ethereum.org/EIPS/eip-6465) | ❌ | n/a | ✅ | ❌ | ✅ |
|| [EIP-7807: SSZ execution blocks](https://eips.ethereum.org/EIPS/eip-7807) (+ SSZ engine API) | ❌ | ❌ | ✅ | ❌ | ❌ |
| 🐣 | [**<nobr>VERGE → Basic light client support</nobr>**](https://vitalik.eth.limo/general/2024/10/23/futures4.html)
|| [Altair light client](https://github.com/ethereum/consensus-specs/blob/dev/specs/altair/light-client/sync-protocol.md) | [🔗](https://github.com/ethereum/consensus-specs/tree/dev/tests/formats/light_client) | ✅ | n/a | n/a | ✅ |
|| [EIP-7658: Light client data backfill](https://eips.ethereum.org/EIPS/eip-7658) | ❌ | ❌ | n/a | n/a | n/a |

[L1 R&D Workshop slides](./slides.pdf) \| [RLP -> SSZ converting explorer](https://eth-light.xyz) \| [SSZ StableContainer implementations](https://stabilitynow.box) \| [Kurtosis](./network_params_pureth.yaml) \| [Future ideas](./future.md)
