## Access control

### 1 Concepts
| Role | Acronym | Description |
|:--:|:--:|:--|
| Storage Client | SC | (1) Provide globally useful dataset metadata, submit according to DataSwap standards to earn ecosystem builder rewards. <br> (2) Submits Datacap collateral to the Supervisory Contract based on the dataset's storage capacity to obtain access rights to the Datacap |
| Data Preparer | DP | (1) Download and successfully submit verified metadata datasets, form data packages, and engage in transactions on Dataswap to earn funding rewards |
| Storage Provider | SP | (1) Pre-pledge auction funds to bid for datacap storage qualification<br>(2) Store verified data to earn Filecoin network block rewards <br>(3) Store orders from Dataswap to receive ecosystem rewards|
| Data Auditor | DA | (1) Review dataset source data validity to earn ecosystem builder rewards |
| Retrieve Provider | RP | (1) Provide retrieval services to earn rewards from retrieval clients<br>(2) Offer Dataswap retrieval nodes and services to receive ecosystem rewards |
| Compute Provider | CP | (1) Provide big data analysis and mining services to obtain funding rewards<br>(2) Offer ecosystem-related computations, such as DP, Seal, and Proof calculations, to receive funding rewards |
| Retrieve Client | RC | (1) Spending funds to access retrieval services |
| Compute Client | CC | (1) Spending funds to acquire computational services |

### 2 Type and State Variabless
you can refer to @openzeppelin/contracts/access

### 3 Constant 
```
    bytes32 public constant STORAGE_CLIENT = keccak256("SC");
    bytes32 public constant DATASET_PROVIDER = keccak256("DP");
    bytes32 public constant STORAGE_PROVIDER = keccak256("SP");
    bytes32 public constant DATASET_AUDITOR = keccak256("DA");
    bytes32 public constant RETRIEVE_PROVIDER = keccak256("RP");
    bytes32 public constant COMPUTE_PROVIDER = keccak256("CP");
    bytes32 public constant REVIEWER_CLIENT = keccak256("RC");
    bytes32 public constant COMPUTE_CLIENT = keccak256("CC");
```

### 3 Interface
```
/// @title IRoles Interface
/// @notice This interface defines the role-based access control for various roles within the system.
/// Note:IAccessControlEnumerable is from @openzeppelin/contracts/access/IAccessControlEnumerable.sol

interface IRoles is IAccessControlEnumerable {
    ///@dev The new owner accepts the ownership transfer.
    function acceptOwnership() external;

    /**
     * @dev Revert with a standard message if `_msgSender()` is missing `role`.
     * Overriding this function changes the behavior of the {onlyRole} modifier.
     *
     * Format of the revert message is described in {_checkRole}.
     *
     * _Available since v4.6._
     */
    function checkRole(bytes32 _role) external view;

    ///@dev Returns the address of the current owner.
    function owner() external view returns (address);

    ///@dev Returns the address of the pending owner.
    function pendingOwner() external view returns (address);

    /**
     * @dev Leaves the contract without owner. It will not be possible to call
     * `onlyOwner` functions. Can only be called by the current owner.
     *
     * NOTE: Renouncing ownership will leave the contract without an owner,
     * thereby disabling any functionality that is only available to the owner.
     */
    function renounceOwnership() external;

    /**
     * @dev Starts the ownership transfer of the contract to a new account. Replaces the pending transfer if there is one.
     * Can only be called by the current owner.
     */
    function transferOwnership(address _newOwner) external;
}
```

[return to the system page](../../README.md#231-core-layeryou-can-consider-it-as-the-infrastructure-layer)