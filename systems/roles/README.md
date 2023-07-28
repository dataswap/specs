# 成员管理合约设计
## 1 对外依赖
成员管理基于openzeppelin的AccessControl合约设计
```
// 在此导入OpenZeppelin的Access Control库
import "@openzeppelin/contracts/access/AccessControl.sol";

// 合约定义
contract BigDataDaoRole is AccessControl 

```
## 2 状态变量设计
### 2.1 自定义数据结构
```
    // 定义角色
    bytes32 public constant STORAGE_PROVIDER_ROLE = keccak256("SP");
    bytes32 public constant RETRIEVE_PROVIDER_ROLE = keccak256("RP");
    bytes32 public constant COMPUTE_PROVIDER_ROLE = keccak256("CP");
    bytes32 public constant METADATA_PROVIDER_ROLE = keccak256("MDP");
    bytes32 public constant FULLDATA_PROVIDER_ROLE = keccak256("FDP");
    bytes32 public constant METADATA_REVIEWER_ROLE = keccak256("MDR");
    bytes32 public constant FULLDATA_REVIEWER_ROLE = keccak256("FDR");
    bytes32 public constant REVIEWER_CLIENT_ROLE = keccak256("RC");
    bytes32 public constant COMPUTE_CLIENT_ROLE = keccak256("CC");
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");

    // 自定义的权限等级，用于按贡献度划分等级，最低等级NO_PERMISSION只具备角色不能使用角色权限
    enum PermissionLevel { NO_PERMISSION, CONTRIBUTOR, AUDITOR, ADMIN }

    // 记录成员的信息
    struct Member {
        address addr;
        PermissionLevel permissionLevel; //权限等级
        string nickname; // 昵称
        string country; // 国家
        string city; // 城市
        string organization; // 单位
        string email; //EMAIL
        bool kycCompleted; // 是否已经完成KYC
        bool kybCompleted; // 是否已经完成KYB
    }

```
### 2.2 状态变量
```
    // 存储成员信息
    mapping(address => Member) public members;

    // 数据集任务信息表
    mapping(address => mapping(address=> bool)) taskrecords;
```

## 3 函数接口设计
### 3.1 事件设计
```
    // 定义事件，用于通知成员加入和退出等状态变化
    event MemberRegisted(address indexed member, PermissionLevel permissionLevel);
    event MemberJoined(address indexed member, bytes32 indexed role, PermissionLevel permissionLevel);
    event MemberExited(address indexed member);
    event MemberTaskNotify(address indexed member);
    event RoleApplied(address indexed member, bytes32 indexed role);
    event KYCCompleted(address indexed member);
    event KYBCompleted(address indexed member);
```

### 3.2 成员注册
```
    // 成员注册
    function registerMember(
        string memory _nickname,
        string memory _country,
        string memory _city,
        string memory _organization,
        string memory _email,
        string[] memory _roles
    ) external
```
### 3.3 成员权限申请

```
    // 成员权限申请
    function applyPermission(bytes32 _role) external {
        // 申请的成员必须是已注册成员
        require(members[msg.sender].addr != address(0), "You must be a registered member.");
        // 简化示例，申请过程可以由合约管理员或特定角色进行审核
        // 此处省略审核流程，直接将申请的权限级别更新到成员信息
    }
```

### 3.4 成员权限审核
```
    // 成员权限审核（此处简化为由管理员直接设定权限）
    function approvePermission(address _member,bytes32 _role) external onlyRole(ADMIN_ROLE) {
        require(members[_member].addr != address(0), "Member does not exist.");
        //TODO: 成员权限审核,增加权限
    }
```

### 3.5 成员权限删除

```
    // 成员权限删除
    function deletePermission(address _member, bytes32 _role) external onlyRole(ADMIN_ROLE) {
        require(members[_member].addr != address(0), "Member does not exist.");
        //TODO: 成员权限删除
    }
```

### 3.6 成员退出

```
    // 成员退出
    function exit() external {
        require(members[msg.sender].addr != address(0), "You must be a registered member.");
        // TODO:sender是否等于角色地址
        // TODO:是否有在处理任务
        // TODO:删除自己角色
        emit MemberExited(msg.sender);
    }
```

### 3.7 获取任务成员

```
    // 成员任务分配
    function applyDatasetTask(address datasetAddress, address[] memory excludedApprovers, uint256 count,bytes32 _role) external view returns (address[] memory){
        // TODO:查询数据集当前执行任务的成员
        // TODO:更新任务记录表 
    }

    // 添加任务
    function addTask(address _datasetAddress, address _memberAddress, bytes32 _role) internal {

    }

    // 设置任务完成状态
    function setTaskCompleted(address _datasetAddress, address _memberAddress, bool _completed) internal {

    }

    // 获取任务完成状态
    function getTaskCompleted(address _datasetAddress, address _memberAddress) external view returns (bool) {
    }

    // 获取成员信息
    function getMemberInfo(address _memberAddress) external view returns (Member memory) {
        return members[_memberAddress];
    }

    // TODO:成员任务分配策略
    // 成员在分配任务后制定时间内需要完成任务
    // 如果成员在制定时间未完成分配任务，由外部触发更换处理成员，并记录原有成员不良记录
    // 成员不良记录超过阈值自动删除成员角色或者停止成员角色的权限

```

### 3.8 成员权限级别管理

```
    // 获取成员权限级别
    function getMemberPermissionLevel(address _member) external view returns (PermissionLevel) {
        return members[_member].permissionLevel;
    }

    // 以下函数可以由合约管理员使用来更改角色的权限级别
    function setMemberPermissionLeval(address _member,PermissionLevel _permissionlevel) external onlyRole(ADMIN_ROLE) {

    }
```

### 3.9 成员KYC管理

```
     // 设置KYC完成状态
    function setKYCCompleted() external onlyRole(ADMIN_ROLE) {
        address _member = msg.sender;
        require(members[_member].addr != address(0), "You must be a registered member.");
        members[_member].kycCompleted = true;
        emit KYCCompleted(_member);
    }

    // 设置KYC完成状态
    function setKYBCompleted() external onlyRole(ADMIN_ROLE) {
        address _member = msg.sender;
        require(members[_member].addr != address(0), "You must be a registered member.");
        members[_member].kybCompleted = true;
        emit KYBCompleted(_member);
    }
```