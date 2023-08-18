## Storage

### 1 Concepts
Only dataset car files that have undergone successful matching can proceed to the storage phase, where they are stored on the Filecoin network.

### 2 Types
```js
struct Storage {
    bytes32[] doneCars;
}
```

### 3 State Variabless
```js
mapping(uint64 => Storage) private storages; //matchingId=>Storage
```

### 4 Interface
```js
interface IStorages {
    /// @dev Submits a Filecoin deal Id for a matchedstore after successful matching.
    /// @param _matchingId The ID of the matching.
    /// @param _cid The content identifier of the matched data.
    /// @param _filecoinDealId The ID of the successful Filecoin storage deal.
    function submitStorageDealId(
        uint64 _matchingId,
        bytes32 _cid,
        uint64 _filecoinDealId
    ) external;

    /// @dev Submits multiple Filecoin deal Ids for a matchedstore after successful matching.
    /// @param _matchingId The ID of the matching.
    /// @param _cids An array of content identifiers of the matched data.
    /// @param _filecoinDealIds An array of IDs of successful Filecoin storage deals.
    function submitStorageDealIds(
        uint64 _matchingId,
        bytes32[] memory _cids,
        uint64[] memory _filecoinDealIds
    ) external;

    /// @dev Gets the list of done cars in the matchedstore.
    /// @param _matchingId The ID of the matching.
    /// @return An array of content identifiers of the done cars.
    function getStoredCars(
        uint64 _matchingId
    ) external view returns (bytes32[] memory);

    /// @dev Gets the count of done cars in the matchedstore.
    /// @param _matchingId The ID of the matching.
    /// @return The count of done cars in the matchedstore.
    function getStoredCarCount(
        uint64 _matchingId
    ) external view returns (uint64);

    function getTotalStoredSize(
        uint64 _matchingId
    ) external view returns (uint64);

    /// @dev Checks if all cars are done in the matchedstore.
    /// @param _matchingId The ID of the matching.
    /// @return True if all cars are done in the matchedstore, otherwise false.
    function isAllStoredDone(uint64 _matchingId) external view returns (bool);
}

```

[return to the system page](../../README.md#232-module-layeryou-can-consider-it-as-the-domain-layer)
