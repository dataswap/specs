# Data Deals

## 公共功能合约

### IDataTransaction合约（接口）

数据交易公共接口约束数据交易基本操作，供各交易类型继承使用，以下为其接口及属性相关定义。

#### IDataTransaction状态图

![](./img/DataTransactionMachine.png)

#### IDataTransaction方法

- 创建交易。
- 取消交易。
- 证明校验。
- 完成交易。

### Supervision合约（库）

数据交易币质押支付相关功能由Supervision合约实现，以下为其接口及属性相关定义。

#### Supervision方法

- 质押（pledge）
- 付款（successfulTransactionPaid）
- 退款（unsuccessfulTransactionRefunded）

#### Supervision属性

- 质押总量（totalPledgeAmount）

```solidity
uint256 public totalPledgeAmount;
```

## Storage Deal

数据集撮合拍卖功能由Auction合约和Bid合约实现，数据集管理功能由BigData合约和Replica合约实现，存储交易管理功能由StorageTransaction合约实现，datacap管理功能由datacap合约实现，以下为各合约接口及属性相关定义。

### BigData合约

#### BigData功能方法

- 创建
- 更新
- 查询
- 查询副本信息

#### BigData属性

- 说明（description）
- 版本号（versionNumber）
- ID（identifier）
- 大小（size）
- 位置（dataLocation）
- 状态（status）
- 副本数（replicasNumber）
- 副本信息（replicaInformation）

```solidity
struct BigDataInfo {
    string description;
    string versionNumber;
    uint256 identifier;
    uint256 size;
    string dataLocation;
    DataStatus status;
    uint256 replicasNumber;
    Replica[] replicas;
}
```

#### BigData状态

- 有效
- 无效

```solidity
enum DataStatus { Valid, Invalid }
```

### Replica合约

#### Replica方法

- 更新存储：更新SP地址，有效期，更新存储状态为有效（完成拍卖及存储交易后）。
- 续期：完成续期后，更新副本有效期。
- 设置检索价格：SP可以设置检索价格。
- 更新CID与datacap映射信息。datacap分配时更新。

#### Replica属性

```solidity
//副本定义
struct Replica {
    uint256 ID; // 副本ID
    uint256 expiration;    // 有效期
    uint256 datacapMinPriceUnit; // 单个Piece最低价格
    uint256 datacapMaxPriceUnit; // 单个Piece直接成交价格
    string[] spLocation;     // 副本存储区域要求
    uint256 spBandwidth;   // 副本存储带宽要求
    uint256 spReputation;   // 最小信誉值
}

//副本状态
enum ReplicaState { Storing, Active, Inactive }

//副本当前存储状态
struct ReplicaStorageStatus{
    address spAddress;     // 副本存储账号
    string spLocation;     // 副本存储区域要求
    ReplicaState  status;           // 存储进行中，存储有效，存储失效
}

//数据集切片
struct DatasetSlice {
    uint256 sliceId;
    uint256 startHashNumber;
    uint256 endHashNumber;
}

//数据集存储信息
struct DatasetStorageInfo {
    DatasetSlice[] slices
    uint256 counter;
    //数据集切片ID=>Replica[]
    maping(uint256 => Replica[])
    //replicaID=>ReplicaStatus
    maping(uint256 => ReplicaStatus)
}

//数据集ID => DatasetStorageInfo
maping(uint256 => DatasetStorageInfo)

uint256 public defautMinPrice; //默认每个piece最低价格
uint256 public defautMaxPrice; //默认每个piece直接成交价格
uint256 public defautLocationsPerReplica; // 默认数据集切片最少要在多少个区域进行存储
uint256 public defautBandwidth; // 默认带宽要求
uint256 public defautExpiration; // 默认存储时间
uint256 public defaultReputation;   // 默认最小信誉值

```

### Auction合约

#### Auction功能方法

- 创建拍卖（createAuction），根据数据属性创建智能合约拍卖单，拍卖单状态为投标中。
- 取消拍卖（cancelAuction），取消指定拍卖单，退款等清理操作完成后，状态置为取消。
- 重启拍卖（restartAuction），拍卖单重新开启拍卖，状态置为投标中。
- 一键拍卖（oneClickAuction），出价代币质押到监管合约，分配datacap与存储数据CID。状态置为完成状态。
- 结束拍卖（endAuction），拍卖完成，状态置为完成状态。
- 结束投标（endBidding），投标周期到期后，触发结束投标接口，将状态置为标的选择中；拍卖周期内未收到投标时，置为流标状态。
- 选择出价（selectBid），根据出价及SP实际情况，选择合适的SP，分配datacap与存储数据CID。
- 结束选择（endSelection），选择出价周期到期未选择时调用，使用默认选择策略（价高者），分配datacap与存储数据CID。
- 查询拍卖（showAuction），查询拍卖单信息。

#### Auction属性

- 拍卖说明（dataDescription）
- 副本ID（identifier）
- 拍卖价格（storagePrice）
- 拍卖最大时长（maxTransactionDuration）

```solidity
struct AuctionItem {
        string dataDescription;
        uint256 identifier;
        uint256 storagePrice;
        uint256 maxTransactionDuration;
    }
```

#### Auction状态

- 投标中（bidding）
- 流标（failure）, 拍卖周期内未收到投标
- 选择中（selection）
- 确认中（confirmation）
- 取消（cancellation）
- 完成（completion）

```solidity
 enum AuctionStatus { 
    Bidding, 
    Failure, 
    Selection, 
    Confirmation, 
    Cancellation, 
    Completion 
}
```

#### Bid功能方法

- 出价（placeBid），在拍卖单投标周期内，调用出价接口出价竞拍，状态置为投标中。
- 中标（selected），出价代币质押到监管合约，状态置为已选择。
- 取消（cancellation），取消出价竞拍，或未中标取消，出价代币从监管合约退款。状态置为已退款，完成所有取消工作后，状态置为取消。

#### Bid属性

- 投标金额（bidAmount）
- 投标时间（bidTime）
- 投标状态（bidStatus）

```solidity
struct Bid {
    uint256 bidAmount;
    uint256 bidTime;
    BidStatus bidStatus;
}
```

#### Bid状态

- 投标中（biddingInProgress）
- 中标（selected）
- 已退款（refunded）
- 取消（cancellation）

```solidity
enum BidStatus { 
    BiddingInProgress, 
    Selected, 
    Refunded, 
    Cancellation 
}
```

### datacap合约（库）

#### datacap方法  

- 查询：datacap相关信息查询。
- 签发授权：指定拍卖单datacap分配。
- 取消授权：取消指定拍卖单datacap分配。
- 冻结授权：冻结指定拍卖单datacap使用。预留接口，官方暂时不支持冻结操作。

#### datacap属性

- datacap签发量（datacapUsed）

```solidity
struct DataCapInfo { 
    uint256 datacapUsed; 
}
```

### StorageTransaction合约

#### StorageTransaction方法  

- 创建交易：创建存储交易。
- 取消交易：取消交易。
- 证明校验：SP 周期性将已完成存储扇区proof信息提交。证明校验完成datacap与扇区CID有效性校验，存储时长校验。
- 完成交易：存储交易数据被添加到SP可以证明数据存储的扇区中并完成校验。

#### StorageTransaction属性  

- 交易管理员（admin）
- 交易客户（client）
- 交易最大时长（maxTransactionDuration）
- 证明（proof）
- 状态（status）

```solidity
struct Transaction { 
    address admin, 
    address client, 
    uint256 maxTransactionDuration,
    string proof,
    TransactionStatus status; 
}
```

#### StorageTransaction状态

- 交易中（inProgress）
- 失败（failure）
- 完成（completion）

```solidity
enum TransactionStatus { InProgress, Failure, Completion }
```

## Retrieve Deal

检索交易中，数据集下载交易功能由RetrieveTransaction合约实现，数据集检索查找功能由前端业务层实现，以下为合约接口及属性相关定义。

### RetrieveTransaction合约

#### RetrieveTransaction事件

- 质押成功
- 完成进度

```solidity
event Pledged(address indexed payer, uint256 id);
event ProgressSubmitted(uint256 indexed id, string signatureProof);
```

#### RetrieveTransaction方法

- 创建交易：检索支付代币质押到监管合约，分配检索密钥。
- 取消交易：取消交易。
- 证明校验：RP 周期性将已传输完成扇区proof信息提交，证明校验完成签名校验与扇区CID有效性校验，触发自动分段支付。
- 完成交易：确认所有交易证明已经校验完成。

#### RetrieveTransaction属性

- 交易管理员（admin）
- 交易客户（client）
- 交易最大时长（maxTransactionDuration）
- 证明（proof）
- 状态（status）

```solidity
struct Transaction { 
    address admin, 
    address client, 
    uint256 maxTransactionDuration,
    string proof,
    TransactionStatus status; 
}
```

#### RetrieveTransaction状态

- 交易中（inProgress）
- 失败（failure）
- 完成（completion）

## Compute Deal
计算交易中，交易功能由ComputeTransaction合约实现，以下为合约接口及属性相关定义。

### 计算交易合约
每个计算交易以uint256类型作为ID,合约中设计一个计数器记录所有交易总数
计算交易类型分类：car计算、复制证明计算、时空证明计算
计算交易状态设计：计算交易发布公示、计算任务承接申请、计算任务确认、计算前处理、计算进行中、计算后处理、计算失败、计算完成
计算交易定义，包含信息：计算类型、计算交易ID、计算客户、计算承接人、计算交易状态、计算价格、交易最大时长、计算前处理证明、计算证明、计算后证明


#### 计算交易事件

- 质押成功
- 完成进度

```solidity
event Pledged(address indexed payer, uint256 id);
event ProgressSubmitted(uint256 indexed id, string signatureProof);
```

#### 计算交易方法

- 创建交易：计算支付代币质押到监管合约。
- 接受交易：Compute Provider接受计算任务。
- 证明校验：校验证明信息是否正确。不同计算交易校验证明算法不同，以car生成计算为例，使用数据一致性算法验证其正确性。
- 完成交易：完成所有证明校验。
- 取消交易：取消交易。

#### 计算交易属性

- ID（identifier）
- 说明（Description）
- 价格（price）
- 交易管理员（admin）
- 交易客户（client）
- 交易最大时长（maxTransactionDuration）
- 证明（proof）
- 状态（status）

```solidity
struct Transaction {
    uint256 id;
    string Description;
    uint256 price;
    address admin, 
    address client, 
    uint256 maxTransactionDuration,
    string proof,
    TransactionStatus status; 
}
```

#### 计算交易状态

- 交易中（inProgress）
- 失败（failure）
- 完成（completion）
