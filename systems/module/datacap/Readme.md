## Datacap

### 1 Concepts

Any car file that successfully matches is eligible for datacap allocation. Datacap distribution in Dataswap occurs in batches. For instance, a maximum of 10TB of datacap can be allocated in each batch. Only when a user is nearing the completion of storing 10TB will they be eligible to apply for the next 10TB datacap allocation.

For example, if a user has a matching order for 100TB, after successful matching, the user can apply for a first 10TB datacap for storage. Once the user has genuinely stored 8TB, they can apply for the next 10TB datacap allocation.

### 2 Types

### 3 State Variabless
```js
//(matchingID => allocated datacap size)
mapping(uint64 => uint64) private allocatedDatacaps;
```

### 4 Interface
```js
interface IDatacaps {
    /// @dev Requests the allocation of matched datacap for a matching process.
    /// @param _matchingId The ID of the matching process.
    function requestAllocateDatacap(uint64 _matchingId) external;

    /// @dev Gets the allocated matched datacap for a storage.
    /// @param _matchingId The ID of the matching process.
    /// @return The allocated datacap size.
    function getAvailableDatacap(uint64 _matchingId) external returns (uint64);

    /// @dev Gets the allocated matched datacap for a matching process.
    /// @param _matchingId The ID of the matching process.
    /// @return The allocated datacap size.
    function getAllocatedDatacap(
        uint64 _matchingId
    ) external view returns (uint64);

    /// @dev Gets the total datacap size needed to be allocated for a matching process.
    /// @param _matchingId The ID of the matching process.
    /// @return The total datacap size needed.
    function getTotalDatacapAllocationRequirement(
        uint64 _matchingId
    ) external view returns (uint64);

    /// @dev Gets the remaining datacap size needed to be allocated for a matching process.
    /// @param _matchingId The ID of the matching process.
    /// @return The remaining datacap size needed.
    function getRemainingUnallocatedDatacap(
        uint64 _matchingId
    ) external view returns (uint64);

    /// @dev Checks if the next datacap allocation is allowed for a matching process.
    /// @param _matchingId The ID of the matching process.
    /// @return True if next allocation is allowed, otherwise false.
    function isNextDatacapAllocationValid(
        uint64 _matchingId
    ) external view returns (bool);
}
```

