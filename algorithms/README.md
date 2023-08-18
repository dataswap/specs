
# 算法设计

## 1 目标
提出数据一致性校验算法，赋能有效数据存储，支撑无须信任公证人实现:
- 实现对有效数据的存储的证明和校验；
- 以更细的粒度监控客户有效数据存储和校验；
- 通过代码降低公证人工作的复杂度；
- 将datacap和公证人签名解耦，从而避免作恶行为；

## 2 Dataset Consistency Algorithm
### 2.1 介绍
基于一致性算法可以证明数据集原始数据与filecoin存储的数据集的一致性
我们知道filecoin存储交易的存储数据是car文件形式，每个数据集在filecoin上的存储最终体现为一系列car文件的存储完成
- 数据集一致性算法基于merkle证明，car文件的pieceCid是car文件的merkle树根hash,称为leafHash
- 由全部car文件的根hash构成数据集的一颗merkle树，merkle树的根为rootHash 这棵树被称为数据集证明，作为数据集的指纹，这一数据指纹存储到链上
- 一致性算法将每个car文件切分为数据碎片（1M\2M）car文件的数据碎片到car的根hash，即leafHash构成的merkle树称为，数据集的校验树，这些校验树存储到filecoin网络中
- 生成数据集校验树的过程中同时生成原始数据到car数据碎片的映射文件称为数据集校验树源文件；
- 通过校验树源文件和校验树，配合引入随机性可以通过从源文件获取数兆级别的原始文件生成随机采样点到leafHash的merkle证明，这可以证明原始文件是否在脸上数据集指纹锁代表的原始数据是否一致；
- 通过上述证明我们可以知道只要存储数据指纹中leafHash（即PieceCID）所对应的car文件，即存储了与原始数据一致的数据集文件；
- 上述所描述数据证明有和数据集校验信息的提交是DataSwap成员基于数据集一致性算法形成的共识；
- 因为leafHash（即PieceCID）被提前校验，在DataCap分配时可以将发放粒度控制到一个car文件级别的自动发放和管理；

### 2.2 原理

数据一致性校验算法基本原理如下图所示:
![数据一致性校验算法原理图](./img/datasetConsistencyAlgorithm.png)

## 3 Dataset Consistency Proof and Verification Toolset

一致性证明工具集用于实现一致性证明算法，包括数据集证明的生成、源数据采样、校验工具


### 3.1 car生成工具
增加如下功能
    - 基于singularity改造
    - 数据集证明信息（上链）
    - 源文件与car的映射关系
    - 缓存树（car文件的merkle树,filecoin/ipfs存储）
	- 数据集原始文件扫描策略(含car文件大小配置)

参考工具：singularity(generate-car)、boost（验证参考项目）、依赖go-unixfs、go-car、go-unixfsnode

car生成工具通过对go-unixfs、go-ipfs-chunker、以及对Writer的改造，通过对generate-car扩展实现功能
MerkleTree 通过[go-merkletree](https://github.com/txaty/go-merkletree)实现commP以及一致性证明的生成

原理如下:

![metaservice](./img/metaservice.png)

#### 3.1.1 源文件扫描策略
// TODO: 构建源文件扫描策略使同一份源文件能够输出相同的数据集证明
- 从数据源到本地的文件规范化下载
- 从源数据到car文件映射信息，控制到单个car的生成(包含其源数据下载)


#### 3.1.2 映射文件生成
映射文件生成基于对[generate-car](https://github.com/tech-greedy/generate-car)的改造完成，不影响generate-car原有功能，增加映射文件的保存

##### 3.1.2.1 go-unixfs二次开发设计
定义Helper接口

```
type Helper interface {
	Done() bool
	Next() ([]byte, error)
	GetDagServ() ipld.DAGService
	GetCidBuilder() cid.Builder
	NewLeafNode(data []byte, fsNodeType pb.Data_DataType) (ipld.Node, error)
	FillNodeLayer(node *FSNodeOverDag) error
	NewLeafDataNode(fsNodeType pb.Data_DataType) (node ipld.Node, dataSize uint64, err error)
	ProcessFileStore(node ipld.Node, dataSize uint64) ipld.Node
	Add(node ipld.Node) error
	Maxlinks() int
	NewFSNodeOverDag(fsNodeType pb.Data_DataType) *FSNodeOverDag
	NewFSNFromDag(nd *dag.ProtoNode) (*FSNodeOverDag, error)
}

```

实现Helper接口并对NewLeafDataNode重新封装，记录源数据构造node的源文件信息
定义Helper接口实例

```
// Helper回调函数,构造car文件node时将node cid及node类型回传
type HelperAction func(node ipld.Node, nodeType pb.Data_DataType)

// Helper接口实例
type WrapDagBuilder struct {
	db  *ihelper.DagBuilderHelper  //原Helper
	hcb HelperAction //helper回调函数
}

```

改造Layout和fillNodeRec采用Helper接口实现car文件dag布局,car文件的生成布局方法不改动，只在node加入dag时调用Helper回调将cid和节点类型传回

[go-unixfs](https://github.com/ipfs/go-unixfs)中关于Layout策略如下:
```
//	       +-------------+
//	       |   Root 1    |
//	       +-------------+
//	              |
//	 ( fillNodeRec fills in the )
//	 ( chunks on the root.      )
//	              |
//	       +------+------+
//	       |             |
//	  + - - - - +   + - - - - +
//	  | Chunk 1 |   | Chunk 2 |
//	  + - - - - +   + - - - - +
//
//	                     ↓
//	When the root is full but there's more data...
//	                     ↓
//
//	       +-------------+
//	       |   Root 1    |
//	       +-------------+
//	              |
//	       +------+------+
//	       |             |
//	  +=========+   +=========+   + - - - - +
//	  | Chunk 1 |   | Chunk 2 |   | Chunk 3 |
//	  +=========+   +=========+   + - - - - +
//
//	                     ↓
//	...Layout's job is to create a new root.
//	                     ↓
//
//	                      +-------------+
//	                      |   Root 2    |
//	                      +-------------+
//	                            |
//	              +-------------+ - - - - - - - - +
//	              |                               |
//	       +-------------+            ( fillNodeRec creates the )
//	       |   Node 1    |            ( branch that connects    )
//	       +-------------+            ( "Root 2" to "Chunk 3."  )
//	              |                               |
//	       +------+------+             + - - - - -+
//	       |             |             |
//	  +=========+   +=========+   + - - - - +
//	  | Chunk 1 |   | Chunk 2 |   | Chunk 3 |
//	  +=========+   +=========+   + - - - - +

```

##### 3.1.2.2 go-ipfs-chunker二次开发设计

该包用于将原数据切分以构建dag，默认采用SizeSpliter将源数据按大小相等创建数据块，基于SizeSplider进行改造，在切分数据块时将源文件信息传回
定义splitter
```
type sliceSplitter struct {
	r    io.Reader     // 数据源
	size uint32        // 数据块切分的大小
	err  error

	srcPath string     // 记录原文件路径
	cb SplitterAction  // 允许外部传入回调函数获取原始文件读取信息
	offset uint64      // 记录当前文件读取offset
}
```

定义splitter回调函数,用于metaservice接受源文件数据
```
type SplitterAction func(srcPath string, offset uint64, size uint32, eof bool)
```
实现Splitter接口并通过SplitterAction实现源文件拆分信息的输出
```
type Splitter interface {
	Reader() io.Reader
	NextBytes() ([]byte, error) //封装对SplitterAction的调用实现源数据信息的采集
	Bytes(start, offset int) ([]byte, error)
}
```

##### 3.1.2.3 metasevice设计
metaservice以node cid追踪node源数据和car文件node之间的关联

```
//chunk meta数据定义
type ChunkMeta struct {
	SrcPath   string           `json:"srcpath"`    //该chunk 采集源文件的路径
	SrcOffset uint64           `json:"srcoffset"`  //该chunk data所在的源文件偏移
	Size      uint32           `json:"size"`       //该chunk data大小
	DstPath   string           `json:"dstpath"`    //该chunk所在的car文件路径
	DstOffset uint64           `json:"dstoffset"`  //该chunk在目标car中偏移
	NodeType  pb.Data_DataType `json:"nodetype"`   //node类型
	Cid       cid.Cid          `json:"cid"`        //node cid
	Links     []*ipld.Link           `json:links`  // node的chunk,即子node
}

//获取一个chunk在目标car中的start和结束位置
func (cm *ChunkMeta) GetDstRange(c cid.Cid) (uint64, uint64)

//定义源数据信息
type SrcData struct {
	Path   string
	Offset uint64
	Size   uint32
}
```

```
// MetaService定义
type MetaService struct {
	spl    chunker.Splitter //Splitter
	writer io.Writer        //目标car文件writer
	helper ihelper.Helper   //Helper

	metas map[cid.Cid]*types.ChunkMeta  //源数据列表
	lk    sync.Mutex

	splCh chan *types.SrcData           //用于回传源数据信息

	calc          *commp.Calc           //commp计算器
	hashs         map[uint]map[int][]byte   //层数->节点序号->hash
	hlk           sync.Mutex
}


//目标writer封装
//写入后Action
type WriteAfterAction func(path string, cid cid.Cid, count int, offset uint64)

//写入前Action
type WriteBeforeAction func([]byte, io.Writer) ([]byte, error)

//writer封装
type WrapWriter struct {
	io.Writer                   //目标car writer
	path   string               //目标car path
	offset uint64               //当前的写入offset
	after  WriteAfterAction     //car写入后Action 句柄
	before WriteBeforeAction    //car写入前Action 句柄
}

重写func (bc *WrapWriter) Write(p []byte) (int, error) 记录写入信息，重新实现 Writer
```


##### 3.1.2.4 generate-car二次开发设计



### 3.2 证明挑战算法

#### 3.2.1 源文件证明挑战
	- 源数据碎片(+映射文件+缓存树)到merkle证明的生成

#### 3.2.2 car文件证明挑战
	- car文件碎片(+映射文件+缓存树)到merkle证明的生成

### 3.3 校验

#### 3.3.1 一致性证明的链上校验算法
	- merkle证明的正确性校验 

#### 3.3.2 一致性证明的通用校验算法
	- merkle证明的正确性校验 

### 3.4 随机抽样算法
	- car文件的随机挑战
	- 源文件随机挑战点

参考filecoin的随机挑战算法

### 3.5 源数据采样工具
	- 通用源（aws\s3\http）数据碎片采样、下载%