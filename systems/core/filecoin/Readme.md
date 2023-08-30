## Filecoin

### 1 Concepts
- please refer to [filecoin](https://filecoin.io/)
- This module primarily communicates with built-in contracts within Filecoin.

### 2 Types 
```js
    /// @notice Enum representing the possible states of a Filecoin storage deal.
    enum DealState {
        Stored, // The filecoin deal's verification was successful.
        StorageFailed, // The filecoin deal's verification failed.
        Slashed, // The filecoin deal has been slashed.
        Expired // The filecoin deal has expired.
    }

    enum Network {
        Mainnet,
        CalibrationTestnet
    }
```

### 3 State Variables

```js
FilecoinType.Network public network;
```

### 4 interface
```js
/// @title IFilplus
interface IFilecoin {
    /// @notice Internal function to get the state of a Filecoin storage deal for a replica.
    function getReplicaDealState(
        bytes32 _cid,
        uint64 _filecoinDealId
    ) external returns (FilecoinType.DealState);

    /// @dev do nothing,just for mock
    function setMockDealState(FilecoinType.DealState _state) external;
}

```
[return to the system page](../../README.md#231-core-layeryou-can-consider-it-as-the-infrastructure-layer)