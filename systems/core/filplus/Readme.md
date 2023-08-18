## Filplus

### 1 Concepts
please refer to [filplus](https://filplus.info/)

### 2 Types 
```js
enum MatchingRuleCommissionType {
    BuyerPays,
    SellerPays,
    SplitPayment
}
```

### 3 State Variables
State variables in FilPlus can only be modified by governance contracts.

```js
///@notice car rules
uint16 public carRuleMaxCarReplicas; // Represents the maximum number of car replicas in the entire network

///@notice dataset region rules
uint16 public datasetRuleMinRegionsPerDataset; // Minimum required number of regions (e.g., 3).

uint16 public datasetRuleDefaultMaxReplicasPerCountry; // Default maximum replicas allowed per country.

mapping(bytes32 => uint16) private datasetRuleMaxReplicasInCountries; // Maximum replicas allowed per country.

uint16 public datasetRuleMaxReplicasPerCity; // Maximum replicas allowed per city (e.g., 1).

///@notice dataset sp rules
uint16 public datasetRuleMinSPsPerDataset; // Minimum required number of storage providers (e.g., 5).

uint16 public datasetRuleMaxReplicasPerSP; // Maximum replicas allowed per storage provider (e.g., 1).

uint16 public datasetRuleMinTotalReplicasPerDataset; // Minimum required total replicas (e.g., 5).

uint16 public datasetRuleMaxTotalReplicasPerDataset; // Maximum allowed total replicas (e.g., 10).

///@notice datacap rules
uint64 public datacapRulesMaxAllocatedSizePerTime; // Maximum allocate datacap size per time.

uint8 public datacapRulesMaxRemainingPercentageForNext; // Minimum completion percentage for the next allocation.

///@notice matching rules
uint8 public matchingRulesDataswapCommissionPercentage; // Percentage of commission.

MatchingRuleCommissionType private matchingRulesCommissionType; // Type of commission for matching.
```

### 4 interface
```js
/// @title IFilplus
interface IFilplus {
    // Public getter function to access datasetRuleMaxReplicasInCountries
    function getDatasetRuleMaxReplicasInCountry(
        bytes32 _countryCode
    ) external view returns (uint16);

    // Set functions for public variables
    function setCarRuleMaxCarReplicas(uint16 _newValue) external;

    function setDatasetRuleMinRegionsPerDataset(uint16 _newValue) external;

    function setDatasetRuleDefaultMaxReplicasPerCountry(
        uint16 _newValue
    ) external;

    function setDatasetRuleMaxReplicasInCountry(
        bytes32 _countryCode,
        uint16 _newValue
    ) external;

    function setDatasetRuleMaxReplicasPerCity(uint16 _newValue) external;

    function setDatasetRuleMinSPsPerDataset(uint16 _newValue) external;

    function setDatasetRuleMaxReplicasPerSP(uint16 _newValue) external;

    function setDatasetRuleMinTotalReplicasPerDataset(
        uint16 _newValue
    ) external;

    function setDatasetRuleMaxTotalReplicasPerDataset(
        uint16 _newValue
    ) external;

    function setDatacapRulesMaxAllocatedSizePerTime(uint64 _newValue) external;

    function setDatacapRulesMaxRemainingPercentageForNext(
        uint8 _newValue
    ) external;

    function setMatchingRulesDataswapCommissionPercentage(
        uint8 _newValue
    ) external;

    function setMatchingRulesCommissionType(uint8 _newType) external;

    // Default getter functions for public variables
    function governanceAddress() external view returns (address);

    function carRuleMaxCarReplicas() external view returns (uint16);

    function datasetRuleMinRegionsPerDataset() external view returns (uint16);

    function datasetRuleDefaultMaxReplicasPerCountry()
        external
        view
        returns (uint16);

    function datasetRuleMaxReplicasPerCity() external view returns (uint16);

    function datasetRuleMinSPsPerDataset() external view returns (uint16);

    function datasetRuleMaxReplicasPerSP() external view returns (uint16);

    function datasetRuleMinTotalReplicasPerDataset()
        external
        view
        returns (uint16);

    function datasetRuleMaxTotalReplicasPerDataset()
        external
        view
        returns (uint16);

    function datacapRulesMaxAllocatedSizePerTime()
        external
        view
        returns (uint64);

    function datacapRulesMaxRemainingPercentageForNext()
        external
        view
        returns (uint8);

    function matchingRulesDataswapCommissionPercentage()
        external
        view
        returns (uint8);

    function getMatchingRulesCommissionType() external view returns (uint8);
}
```
[return to the system page](../../README.md)