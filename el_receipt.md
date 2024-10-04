```py
class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]
    data: ByteList[MAX_LOG_DATA_SIZE]

class Receipt(StableContainer[MAX_RECEIPT_FIELDS]):
    from_: Optional[ExecutionAddress]
    gas_used: Optional[uint64]
    contract_address: Optional[ExecutionAddress]
    logs: Optional[List[Log, MAX_LOGS_PER_RECEIPT]]

    # EIP-658
    status: Optional[boolean]

    # EIP-7702
    authorities: Optional[List[ExecutionAddress, MAX_AUTHORIZATION_LIST_SIZE]]
```

authorities:

- MAX_AUTHORIZATION_LIST_SIZE same as in EIP-6404
- for every successful authorization, there is a matching entry with its signer address here
- for every failed authorization, the list contains the zero address

JSON-RPC is extended with `from` and `authorities` entries, as well as with proof in same style as for transactions: https://github.com/ethereum/EIPs/pull/8884/files
