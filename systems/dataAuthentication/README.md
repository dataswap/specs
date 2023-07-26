# 无须信任公证人合约设计

## DataSet合约设计
### 对外依赖
依赖Role合约，获取数据集审批人列表
```
interface Role {
    //获取数据集内容审核人
    function getContentApprovers(address datasetAddress, address[] memory excludedApprovers, uint256 count) external view returns (address[] memory);
    //获取证明提交人，提交人申请和分配逻辑在Role中实现
    function getProofSubmitters(address datasetAddress) external view returns (address);
}
```

### 状态变量设

#### 自定义数据结构

```
// 定义数据集状态
enum DatasetState {Submitted,ContentReview,ContentApproved,ProofSubmitted,ProofSubmissionFailed,ProofVerifying,ProofVerificationDispute,Approved,Rejected}

// 定义数据集
struct Dataset {
    string title;         // 标题
    string industry;      // 行业分类
    string name;          // 名称
    string description;   // 描述
    string source;        // 数据来源
    string accessInfo;    // 描述源数据如何访问的字段
    address owner;        // 所有者
    uint256 createTime;   // 创建时间
    uint256 size;  // 数据集大小
    //uint256 数据属性，是否公共数据集
    uint64 version;
    bool noProofRequired;
}

// 数据集证明结构
struct DatasetProof {
    bytes32 rootHash;           // 根哈希
    bytes32[] leafHashes;       // 所有叶子节点哈希
    string leafAccessInfo;      // 叶子节点信息访问方式
    string metadataAccessInfo;  // 数据集证明元数据信息访问方式
}

// 定义发布信息
struct PublishInfo {
    uint256 numOfCopies;  // 副本数
    string version;       // 版本号
    uint256 releaseDate;  // 发布日期
}
```

### 状态变量

```
mapping(address => Dataset) public datasets;
mapping(address => DatasetState) private datasetStatus;
mapping(address => DatasetProof) public datasetProofs;
mapping(address => PublishInfo) public datasetPublishInfos;
```

### 函数接口设计
#### 数据集注册
```
// 提交数据集注册请求，记录数据集状态为Submitted，生成 datasetAddress 并返回
function registry(
    string memory title,
    string memory industry,
    string memory name,
    string memory description,
    string memory source,
    string memory accessInfo,
    bool noProofRequired,
    address owner,
    uint256 size) internal returns (address)
```

#### 数据集状态更新
//数据集提交随着无须信任公证人合约处理状态不断变更

```
// 数据集状态变更事件
event DatasetStatusChanged(address indexed datasetAddress, uint256 newStatus);

// 改变数据集状态，只允许内部调用
//TODO：只允许TrustlessNotary合约调用
function updateState(address datasetAddress, uint256 newStatus) internal
```

#### 数据集证明提交

```
// 数据集提交接口，用于提交数据集证明
//TODO：只允许TrustlessNotary合约调用
function submitDatasetProof(
    address datasetAddress,
    bytes32 rootHash,
    bytes32[] memory leafHashes,
    string memory leafAccessInfo,
    string memory metadataAccessInfo
) internal

```
#### 数据集证明验证
```
// 数据集证明校验接口，用于验证数据集证明中的 rootHash 和 leafHashes 是否构成一颗 Merkle 树
function verifyDatasetProof(DatasetProof memory datasetProof) internal view returns (bool) {
    // 在此实现验证数据集证明的逻辑
    // TODO:
    // 返回 true 表示验证通过，返回 false 表示验证失败
}
```
#### 获取数据集发布信息
```
// 获取数据集的发布信息
function getPublishInfo(address datasetAddress) external view returns (PublishInfo memory) 
```
#### 判断piece hash是否存在于数据集中
```
// 判断数据集的数据证明的 leafHashes 是否包含指定的哈希值
function isLeafHashInProof(address datasetAddress, bytes32 leafHash) external view returns (bool)
```


## TrustlessNotary合约设计
### 对外依赖
依赖数据集合约

```
interface IDataSet {
    enum DatasetState { Submitted, ContentReview, ContentApproved, ProofSubmitted, ProofVerifing, Approved, Failed }
    function registry(
        string memory title,
        string memory industry,
        string memory name,
        string memory description,
        string memory source,
        string memory accessInfo,
        bool noProofRequired,
        address owner,
        uint256 size) external returns (address);
    function updateState(address datasetAddress, uint256 newStatus) external;
    function submitProof(address datasetAddress,bytes32 rootHash,bytes32[] memory leafHashes,string memory leafAccessInfo,string memory metadataAccessInfo) external;
}
```

### 状态变量设计
#### 自定义数据结构
```
// 内容审核信息
struct ContentReviewInfo {
    bool approved;
    uint128 approvalCount;
    mapping(address => bool) approvals;
}

// 定义证明提交人信息结构
struct ProofInfo {
    address submitter;  // 证明提交人地址
    // 可添加其他相关信息
}

// 定义 Merkle 证明类型
enum VerifyType { Correctness, DataConsistencyDispute, MetadataInaccessibilityDispute }

// 定义数据集校验信息结构
struct DatasetVerificationInfo {
    bytes32[] merkleProof;      // Merkle 证明
    bytes32 proofRandomValue;   // 证明随机值
    mapping(bytes32 => bytes) sampleMapping;   // 采样点与源的映射关系信息
    VerifyType   verifyType;    // 校验类型
    address submitter; //提交人
}
```

#### 状态变量
```
// DataSet合约地址
address public dataSet;

// 内容审核状态映射
mapping(address => ContentReviewInfo) public contentReview;

// 为数据集增加证明提交人信息映射
mapping(address => ProofInfo) public proofInfos;

// 为数据集增加 校验信息
mapping(address => DatasetVerificationInfo ) public datasetVerificationInfo;

// 审批门限值
uint256 public contentThreshold;

// 数据集审批人数，默认为5人，可变更
uint256 public contentApproversCount = 5;

// Role 合约地址
address public roleContract;

```
### 函数接口设计

#### 数据注册请求
向数据集合约注册数据集，获取数据集地址，本地初始化该数据集的内容审批信息，从Role合约获取审批人信息
> TODO：审核人超时未处理更新审核人逻辑

```
// 数据集注册事件
event DatasetRegistered(address indexed datasetAddress, address indexed owner);

// 调用DataSet合约的注册函数
function registryDataset(
    string memory title,
    string memory industry,
    string memory name,
    string memory description,
    string memory source,
    string memory accessInfo, // 新增的字段：描述源数据如何访问
    bool noProofRequired
) external returns (address)
```

#### 内容审批提交
//内容开始审批变更数据集状态为ContentReview，审批人数达到审批门限则数据集状态变更为ContentApproved
```
// 修饰器：验证是否是数据集的审批人
modifier onlyContentApprovers(address datasetAddress) {
    ContentReviewInfo storage reviewInfo = contentReview[datasetAddress];
    require(reviewInfo.approvals[msg.sender], "Only approvers can perform this action");
    _;
}

// 定义数据集审批事件
    event DatasetApproved(address indexed datasetAddress, address indexed approver);

// 提交内容审批通过
function approveContent(address datasetAddress) external onlyContentApprovers(datasetAddress)
```
#### 数据集证明提交
```
// 修饰器：验证是否是数据集证明的提交人
modifier onlyProofSubmitter(address datasetAddress) {
    ProofInfo storage proofInfo = proofInfos[datasetAddress];
    require(proofInfo.submitter == msg.sender, "Only proof submitter can perform this action");
    _;
}

// 增加证明提交接口，只能由证明提交人调用
function submitDatasetProof(
    address datasetAddress,
    bytes32 rootHash,
    bytes32[] memory leafHashes,
    string memory leafAccessInfo,
    string memory metadataAccessInfo
) external
```

#### 校验信息提交
数据集证明提交后允许提交三种类型的校验信息：
正确性证明:Correctness
一致性争议:DataConsistencyDispute
元数据访问性争议:MetadataInaccessibilityDispute
```
// 新增数据集校验信息提交接口，只能由证明提交人调用
function submitDatasetVerificationInfo(
    address datasetAddress,
    bytes32[] memory merkleProof,
    bytes32 proofRandomValue,
    bytes32[] memory samplePoints,
    bytes[] memory sampleData,
    VerifyType verifyType
) external
```
//TODO：
校验通过策略：正确性通过制定门限，争议低于指定门限则判定为成功，可发布数据集，否则判定为失败

#### merkle证明校验 

```
// 私有函数来进行Merkle树的验证
function verifyMerkleProof(
    bytes32[] memory merkleProof,
    bytes32 rootHash,
    bytes32 leafHash,
    bytes32 proofRandomValue
) private pure returns (bool)
```

#### 基本参数配置函数
```
// 设置数据集审批人数
function setContentApproversCount(uint256 _count) external onlyOwner {
    contentApproversCount = _count;
}

// 设置审批门限值
function setContentThreshold(uint256 _threshold) external onlyOwner {
    contentThreshold = _threshold;
}

```