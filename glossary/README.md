## glossary
### 1 system

**SP**

>Storage Provider

**RP**

>Retrieve Provider

**CP**

>Compute Provider

**MDP**

>Dataset Metadata Provider

**DP**

>Dataset Provider  

**MDA**

>Dataset Metadata Auditor

**DA**

>Dataset Auditor  

### 2 algorithms

**CarLeafHash**

>The dataset consistency algorithm splits each car file into multiple data fragments (such as 1M\2M). The hash value of each data fragment is referred to as CarLeafHash.

**CarRootHash** and **CarMerkleTree**

>The CarRootHash of a car file can be obtained by constructing a CarMerkleTree using all the CarLeafHashes in the car file. The CarMerkleTree is stored on the Filecoin network (off-chain).

**DatasetLeafHash**:

>The CarRootHash (pieceCid) of a car file is equivalent to the DatasetLeafHash.

**DatasetRootHash** and **DatasetMerkleTree**:

>The DatasetRootHash of dataset can be obtained by constructing a DatasetMerkleTree using all the DatasetLeafHash(CarRootHash). The DatasetMerkleTree is stored on-chain. 

**MappingFiles**

>The file that contains the mapping relationship between each Car and the source files that constitute it, including the Car-to-source file mapping and the source file-to-Car mapping, is called MappingFiles The MappingFiles is stored on the Filecoin network (off-chain).

**CarProof**

>CarProofs are composed of CarMerkleTree and MappingFiles.TheCarProof is stored on the Filecoin network (off-chain) .

**DatasetProof**

>DatasetProof consists of a DatasetMerkleTree and CarProofs corresponding to all DatasetLeafHashes(CarRootHash). The DatasetMerkleTree is stored on-chain, while the CarProofs are stored on the Filecoin network (off-chain) .