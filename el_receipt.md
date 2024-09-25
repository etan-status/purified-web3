```py
class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]
    data: ByteList[MAX_LOG_DATA_SIZE]

class Receipt(StableContainer[MAX_RECEIPT_FIELDS]):
    gas_used: Optional[uint64]
    contract_address: Optional[ExecutionAddress]
    logs: Optional[List[Log, MAX_LOGS_PER_RECEIPT]]

    # EIP-658
    status: Optional[boolean]
```
