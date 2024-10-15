# Network config

| Config | Value |
| - | - |
| Meta | https://ssz-devnet-0.ethpandaops.io |
| Faucet | https://faucet.ssz-devnet-0.ethpandaops.io |
| CL RPC | https://bn.bootnode-1.ssz-devnet-0.ethpandaops.io |
| EL RPC | https://rpc.bootnode-1.ssz-devnet-0.ethpandaops.io |
| config.yaml | https://config.ssz-devnet-0.ethpandaops.io/cl/config.yaml |
| genesis.ssz | https://config.ssz-devnet-0.ethpandaops.io/cl/genesis.ssz |
| genesis.json | https://config.ssz-devnet-0.ethpandaops.io/el/genesis.json |

# Light clients

Ethereum is accessed via two servers, one provides consensus data (what is the latest block), the other provides execution data (what are the transactions in the blocks, what are the balances and so on). Anyone can provide those servers, but they are too heavy to run on phones, browser extensions and so on. Those are called light clients and need to query someone else's server.

# Minimizing trust

Light clients historically have to trust the server to provide correct data to them. This is a problem because reputable providers have a tendency to have restrictive privacy policies (logging, associating requests pertaining to a set of wallets with the sender's IP address / timing). Fusaka-light extends server data with correctness proofs so that incorrect and incomplete data can be detected. This allows light clients to send their requests to any server offering the API, and to split requests pertaining to different wallets to separate servers, improving security and privacy for end users.

# Helios

The API used by wallets is the [Ethereum JSON-RPC API](https://ethereum.org/en/developers/docs/apis/json-rpc). To ensure that appropriate correctness proofs are obtained and verified, [Helios](https://github.com/a16z/helios) can be installed as a proxy between the wallet and the RPC server. When the wallet requests `eth_getBalance` for its account from Helios, Helios transforms this to an `eth_getProof` request, obtains and verifies the balance, and synthesizes the correct response to the wallet `eth_getBalance` request. The wallet can continue using convenience APIs, same as if it was talking to a trusted server, but without actually requiring a trusted server; by linking Helios, any server becomes a trusted server.

# Linking Helios

First step is to integrate Helios into the app in some way.

- [https://github.com/a16z/helios](https://github.com/a16z/helios)
- [https://github.com/rkreutz/HeliosKit](https://github.com/rkreutz/HeliosKit) (outdated but can be updated)
- [https://github.com/cawfree/react-native-helios](https://github.com/cawfree/react-native-helios)

RPC servers and network configuration files will be provided once available.

# Ethereum addresses

Ethereum addresses contain 20 bytes of information and are represented as a `0x`-prefixed hex string.

Allow user to type or paste such an address, or scan it from QR code.

The app should then start monitoring that address, displaying the balances and transaction history.

# ETH balance

To query ETH balance, send [`eth_getBalance`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getbalance) to the local Helios proxy.

The app should display the monitored account balance and allow refreshing it and/or do so periodically.

# Transaction history

To query transaction history, send [`eth_getLogs`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getlogs) to the local Helios proxy.

- In the `topics` argument, specify the monitored account address, zero prefixed to 32 bytes
- Use `fromBlock` / `toBlock` to indicate what range of blocks to sync
- Use Helios APIs to determine the latest block number. Note that blocks beyond the finalized block number may get invalidated / updated to point to different blocks.

In the response, each log has an `address`, 0-4 `topics` and `data`.

- `topics[0]` is `keccak256(event_signature)`. Reverse lookup via [https://www.4byte.directory](https://www.4byte.directory).
    - `topics[0]` for `Transfer(address,address,uint256)` is [`0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef`](https://www.4byte.directory/event-signatures/?bytes_signature=0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef)
- `address` indicates what address originated the event.
    - For plain ETH `Transfer`, `address` is [`0xfffffffffffffffffffffffffffffffffffffffe`](./el_logs.md).
    - For ERC20 token `Transfer`, `address` is the token contract address.
- `topics[1 ..< 4]` contain indexed arguments of the event.
    - For `Transfer`, `topics[1]` is the `from` address (zero prefixed to fill uint256)
        - For plain ETH, `topics[1]` is set to `0xfffffffffffffffffffffffffffffffffffffffe` when new ETH is minted to the account (block reward / withdrawal from validator)
    - For `Transfer`, `topics[2]` is the `to` address (zero prefixed to fill uint256)
        - For plain ETH, `topics[2]` is set to `0xfffffffffffffffffffffffffffffffffffffffe` when paying for transaction fees.
- `data` contains non-indexed arguments of the event.
    - For `Transfer`, this is a big-endian `uint256` containing the amount
        - For plain ETH, this is in Wei (10^-18 ETH)

The app should track the events associated with an account and list them. For known events such as `Transfer`, it should display them nicely.

# Transaction details

For each log, the `transactionHash` is provided in the JSON-RPC response. To query transaction details, send [`eth_getTransactionByHash`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_gettransactionbyhash) and [`eth_getTransactionReceipt`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_gettransactionreceipt) to the local Helios proxy.

The app should allow clicking through from a log to its corresponding transaction / receipt details so that more details can be examined.

# ERC20 tokens

When a new ERC20 token is discovered by looking at the `address` of a `Transfer` event, information about the token can be requested from that token contract by sending [`eth_call`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_call) to the local Helios proxy.

- Set `to` key of first parameter dictionary to the ERC20 token contract `address`
- Set `input` key of first parameter dictionary to `keccak256(function_abi)[0 ..< 4]` followed by any arguments
- Set second parameter to `latest`

[ERC-20](https://eips.ethereum.org/EIPS/eip-20) defines the function signatures.

- For `name()`, `input` is [`0x06fdde03`](https://www.4byte.directory/signatures/?bytes4_signature=0x06fdde03) and return is ABI-encoded `string`
- For `symbol()`, `input` is [`0x95d89b41`](https://www.4byte.directory/signatures/?bytes4_signature=0x95d89b41) and return is ABI-encoded `string`
- For `decimals()`, `input` is [`0x313ce567`](https://www.4byte.directory/signatures/?bytes4_signature=0x313ce567) and return is big endian `uint256`
- For `balanceOf(address)`, `input` is [`0x70a08231`](https://www.4byte.directory/signatures/?bytes4_signature=0x70a08231) followed by address zero prefixed to 32 bytes and return is big endian `uint256`

The app should provide a nice UI that interprets ERC20 contract data:

- The latest balance should be queried using `eth_call` on the token contract with `balanceOf(address)`.
- The transaction history should display token transfers based on `symbol()` and `decimals()`.

Example for USDC `name()`:

```sh
curl 'https://execution-api.eth-light.xyz/v1/mainnet' \
-X 'POST' \
-H 'Content-Type: application/json' \
--data-binary '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","input":"0x06fdde03"},"latest"]}' | jq '.'
```

```sh
{
  "jsonrpc": "2.0",
  "result": "0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000855534420436f696e000000000000000000000000000000000000000000000000",
  "id": 1
}
```

Then split into uint256 words:

```
0000000000000000000000000000000000000000000000000000000000000020
0000000000000000000000000000000000000000000000000000000000000008
55534420436f696e000000000000000000000000000000000000000000000000
```

- First is the offset (data starts in 32 bytes)
- Second is the string length (8 bytes)
- Third is UTF-8 data (`USD Coin`)
- [https://medium.com/@scourgedev/deep-dive-into-abi-encode-types-padding-and-disassembly-84472f1b4543](https://medium.com/@scourgedev/deep-dive-into-abi-encode-types-padding-and-disassembly-84472f1b4543)

Example for USDC `balanceOf(0x37305B1cD40574E4C5Ce33f8e8306Be057fD7341)`:

```sh
curl 'https://execution-api.eth-light.xyz/v1/mainnet' \
-X 'POST' \
-H 'Content-Type: application/json' \
--data-binary '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","input":"0x70a0823100000000000000000000000037305b1cd40574e4c5ce33f8e8306be057fd7341"},"latest"]}' | jq '.'
```

```sh
{
  "jsonrpc": "2.0",
  "result": "0x00000000000000000000000000000000000000000000000000066c2da4eb2860",
  "id": 1
}
```

Transforming to decimal gives 1807793156466784, and applying the `decimals()` result of 6 this becomes 1'807'793'156.466784 USDC, which matches [https://etherscan.io/tokenholdings?a=0x37305b1cd40574e4c5ce33f8e8306be057fd7341](https://etherscan.io/tokenholdings?a=0x37305b1cd40574e4c5ce33f8e8306be057fd7341).
