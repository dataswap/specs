## Access control

### 1 Concepts
| Role | Acronym | Description |
|:--:|:--:|:--|
| Storage Provider | SP | (1) Store useful data to earn FIL rewards<br>(2) Store orders from Dataswap to earn ecosystem rewards |
| Retrieve Provider | RP | (1) Provide retrieval services to earn FIL rewards<br>(2) Offer Dataswap retrieval nodes and services to earn ecosystem rewards |
| Compute Provider | CP | (1) Offer big data analysis and mining services to earn FIL or ecosystem rewards<br>(2) Provide ecosystem-related computations, such as DP, Seal, and Proof calculations, to earn FIL or ecosystem rewards |
| Metadata Provider | MDP | (1) Provide globally useful dataset metadata, submit according to DataSwap standards to earn ecosystem builder rewards |
| Data Provider | DP | (1) Download and prove successfully submitted metadata datasets, form data packages for trading on Dataswap to earn ecosystem builder rewards |
| Metadata Auditor | MDA | (1) Review dataset metadata validity to earn ecosystem builder rewards |
| Data Auditor | DA | (1) Review dataset source data validity to earn ecosystem builder rewards |
| Retrieve Client | / | (1) Spend FIL to access retrieval services |
| Compute Client | / | (1) Spend FIL to access computation services |

### 2 Type and State Variabless
you can refer to @openzeppelin/contracts/access

### 3 Constant 
```
    bytes32 public constant STORAGE_PROVIDER = keccak256("SP");
    bytes32 public constant RETRIEVE_PROVIDER = keccak256("RP");
    bytes32 public constant COMPUTE_PROVIDER = keccak256("CP");
    bytes32 public constant METADATA_DATASET_PROVIDER = keccak256("MDP");
    bytes32 public constant DATASET_PROVIDER = keccak256("DP");
    bytes32 public constant METADATA_DATASET_AUDITOR = keccak256("MDA");
    bytes32 public constant DATASET_AUDITOR = keccak256("DA");
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

[return to the system page](../../README.md)