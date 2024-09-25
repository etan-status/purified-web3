```ts
export const BlockHeader = new StableContainerType(
  {
    parentHash: new OptionalType(Bytes32),
    coinbase: new OptionalType(Bytes20),
    stateRoot: new OptionalType(Bytes32),
    transactionsTrie: new OptionalType(Bytes32),
    receiptsTrie: new OptionalType(Bytes32),
    number: new OptionalType(Uint64),
    gasLimits: new OptionalType(FeesPerGas),
    gasUsed: new OptionalType(FeesPerGas),
    timestamp: new OptionalType(Uint64),
    extraData: new OptionalType(new ByteListType(MAX_EXTRA_DATA_BYTES)),
    mixHash: new OptionalType(Bytes32),
    baseFeePerGas: new OptionalType(FeesPerGas),
    withdrawalsRoot: new OptionalType(Bytes32),
    excessGas: new OptionalType(FeesPerGas),
    parentBeaconBlockRoot: new OptionalType(Bytes32),
    requestsRoot: new OptionalType(Bytes32),
  },
  MAX_BLOCKHEADER_FIELDS,
  { typeName: 'BlockHeader', jsonCase: 'eth2' },
)
```
