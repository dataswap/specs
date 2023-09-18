# 2. Systems

## 2.1 Trustless notary
### 2.1.1 Dataswap storage overview

![](img/trustlessnotory-roles.png)

### 2.1.2 Handling Sybil Attacks

In the above-mentioned system, how to prevent Sybil attacks by DAs (collusion between DP and DA) is a critical issue. In the absence of malicious behavior by DAs, dataset consistency algorithms can prevent collusion and other malicious activities among SC, DP, and SP.

Here is the operational process through which DAs can prevent Sybil attacks:

- DA's admission requires staking, and anyone can become a DA by submitting a stake. DA's admission stake can only be released after the dataset audit is completed.
- Only after becoming a DA can one earn DA fees and DA challenge fees.
  - DA fees are the fees earned by DAs for correctly submitting proofs for datasets.
  - DA challenge fees refer to fees earned by DAs when they successfully challenge other DAs' proofs, which are found to be fraudulent.
- If a DA discovers that another DA has submitted a false proof for a dataset, they can submit a dispute to the dispute resolution robot.
- The dispute resolution robot is a voting cluster built by a trusted community organization to handle DA disputes. It is only activated when a dispute arises, and it completes the dispute resolution.
- When the dispute resolution results in a successful challenge, the staked amount of the challenged DA is forfeited, and it is used to pay the fees for both the challenging DA and the dispute resolution robot.
- A DA that has been successfully challenged has their staked amount forfeited and loses their DA role authorization.

![](img/sybil-attacks-handling.png)

### 2.1.3 Dataswap storage overall process

```mermaid
sequenceDiagram
Storage Client(SC)->>Dataswap: submit dataset info(metadata)
Storage Client(SC)->>Dataswap: datacap collateral
Storage Client(SC)->>Dataswap: DP fee(SC) 
Data Preparer(DP)->>Dataswap: submit dataset proof
Data Auditor(DA)->>Dataswap: submit challenge dataset proof
Data Preparer(DP)->>Dataswap: publish matching
Storage Provider(SP)->>Dataswap: bid and win
Storage Provider(SP)->>Dataswap: datacap(small chunk) collateral
Storage Provider(SP)->>Dataswap: DP fee(SP)
Data Preparer(DP)->>Storage Provider(SP): provide car data
loop Storage in progress 
Storage Provider(SP)->>Dataswap: submit claimIds
Dataswap->>Dataswap: verify claimIds
Dataswap->>Data Preparer(DP): DP fee
end
Dataswap->>Storage Provider(SP): datacap(small chunk) collateral
Dataswap->>Storage Client(SC): Storage end:datacap collateral
Governance Team(GT)->>Dataswap: governance

%%{init:{'themeCSS':'g:nth-of-type(3) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(10) rect.actor { stroke:lightgreen;fill: lightgreen; }; '}}%%
```
### 2.1.4 Dataswap storage diagram

![](img/TrustlessNotary.png)

### 2.1.5 Dataswap storage runtime sequence diagram

```mermaid
sequenceDiagram

%%{init:{'themeCSS':'g:nth-of-type(3) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(4) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(5) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(7) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(9) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(11) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(16) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(17) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(18) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(20) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(22) rect.actor { stroke:lightgreen;fill: lightgreen; }; g:nth-of-type(24) rect.actor { stroke:lightgreen;fill: lightgreen; }; '}}%%
 
SC->>DatasetContract: Submit metadata and storage rules for the dataset
DatasetContract->>FilplusContract: Verify storage rules
FilplusContract-->>DatasetContract: response:Storage rules are compliant
DatasetContract-->>SC: response:The metadata of dataset submission was successful
SC->>SupervisoryContract: Submit DP fee(SC) 
SC->>SupervisoryContract: Submit datacap collateral
SupervisoryContract->>DatasetContract:Check the DataCap collateral base on the size of the dataset and replicas
DatasetContract-->>SupervisoryContract: Datacap collateral is valid
SupervisoryContract-->>SC: response
DP->>DatasetContract: Submit dataset proof 
DatasetContract->>CarstoreContract: Submit cars 
DatasetContract-->>DP: response:The dataset proof has been submitted
loop Dataset challenge verification
DA->>DatasetContract: Submit dataset challenge proof 
DatasetContract->>DatasetContract: Proof verification
DatasetContract-->>DA: response: Challenge verification passed
end
DP->>MatchingContract: Publishing replicas triggers a matching transaction
MatchingContract->>DatasetContract: Check the dataset proof status
DatasetContract-->>MatchingContract: Dataset proof has been approved
MatchingContract->>CarstoreContract: Check the validity of replicas
CarstoreContract-->>MatchingContract: response: Replica is valid,can matching
MatchingContract->>FilplusContract: Check Filplus rules
FilplusContract-->>MatchingContract: Filplus rules is valid 
MatchingContract-->>DP: response: Start the auction
SP->>SupervisoryContract: Submit Dataset chunk collateral
loop In the midst of the auction
SP->>MatchingContract: bidding
MatchingContract->>SupervisoryContract: Check the collateral
SupervisoryContract-->>MatchingContract: response:collateral is valid
MatchingContract->>FilplusContract: Check the compliance of SP storage
FilplusContract-->>MatchingContract: SP storage is compliant
end
MatchingContract-->>CarstoreContract: Update the state of cars
MatchingContract-->>SP: won
SP->>SupervisoryContract: Submit DP fee(SP)
DP-->>SP: off-chain:Data transmission
loop Storage in progress
SP->>StorageContract: Start storage/claimids submission
StorageContract-->>StorageContract: Verify deals
StorageContract->>CarstoreContract:  Update state of replica of chunk
StorageContract->>SupervisoryContract: Expense Settlement
SupervisoryContract->>DP: Commission Settlement for Data Preparation
StorageContract->SupervisoryContract: Check the datacap pledge for chunk
SupervisoryContract-->>StorageContract: response:Approved datacap amount
StorageContract->>SC: Approve datacap
SC->>SP: perform storage deal
end
SupervisoryContract->>SP: Release dataset chunk collateral 
SupervisoryContract->>SC: Storage end:Release datacap collateral
```

### 2.1.6 Concepts 
**Dataswap storage (Service) fully satisfies the specific design requirements of "Trustless Notary Design Space" outlined in the [Filecoin ideation article](https://medium.com/filecoin-plus/ideation-trustless-notary-design-space-guidelines-bc21f6d9d5f2).**

Dataswap storage provides a complete solution through dataset auditing, matching, automatic datacap allocation, and  storage.

The Dataset module, specifically DatasetMetadata, DatasetProof, and DatasetVerification, plays a crucial role in implementing the Trustless Notary.

- **DatasetMetadata:**

  - Storage Client (SC) submits dataset information such as title, industry category, name, description, data source, owner, creation time, creator, modification history, etc., to the dataset contract.
  - Storage Client (SC) submits Datacap collateral to the Supervisory Contract based on the dataset's storage capacity to obtain access rights to the Datacap.

- **DatasetProof:**

  - The Data Preparer (DP) utilizes data proof tools (designed based on [Dataset Consistency Algorithm](../algorithms/README.md#2-dataset-consistency-algorithm)s) to generate dataset proofs, specifically the dataset proof Merkle tree.
  - DP submits the dataset proof to the dataset contract.

- **DatasetVerification:**

  - Data Auditor (DA) uses data proof verification tools (designed based on [Dataset Consistency Algorithm](../algorithms/README.md#2-dataset-consistency-algorithm)s) to generate dataset challenge proof verification information.
  - DA submits the dataset challenge proof verification information to the dataset contract.
  - After all DAs vote to the governance contract.the governance contract determines the final approval of the Dataset based on the audit results from all DAs.

In summary, Dataswap storage, through its dataset module components (DatasetMetadata, DatasetProof and DatasetVerification), addresses the specific design requirements of a Trustless Notary as described in the Filecoin ideation article.

The Matching module implements decentralized matchmaking functionality.
  - Data Preparer (DP) participate in auctions within the matching contract to store their prepared datasets.
  - Data Preparer (DP) need to ensure that their data is transferred (if the transaction is successful, they will send the data) to the winning SP. DP also needs to assist SP in completing the data onboarding process to receive their DP fee.
  - SPs pre-stake funds in the supervisory contract to participate in dataset auctions and obtain validated dataset storage transactions.
  - Storage Provider (SP) who bid during the matching period commit to completing data onboarding within a specific window after the auction ends. If they fail to do so, they forfeit their collateral.

The storage module implements dataset onboarding management.
  - When a Storage Provider (SP) wins the auction, the storage contract automatically approves Datacap for the SC without requiring any additional authorization.
  - After the Storage Provider (SP) completes the storage, they submit the Claim ID to the storage contract to ensure the correct onboarding of data. If there are non-compliant data onboarding issues, the storage contract will halt Datacap approval.

### 2.1.7 Dataset Consistency Verification Algorithm
**Note:Algorithm prototype validation has been successfully passed.**

See [Dataset Consistency Algorithm](../algorithms/README.md#2-dataset-consistency-algorithm) for details.

### 2.1.8 Dataset Consistency Proof and verification Toolset
**Note:Algorithm prototype validation has been successfully passed.**

The Dataset Consistency Proof Toolset is used to implement the consistency proof algorithm, including the generation of dataset proofs, source data sampling, and verification tools etc.
See [Consistency Proof and verify Toolset Design](../algorithms/README.md#3-dataset-consistency-proof-and-verification-toolset) for details.

## 2.2 Architecture
### 2.2.1 Contract Architecture
![](./img/contractArchitecture.png)
### 2.2.2 Test Architecture
![](./img/testArchitecture.png)


## 2.3 detail architecture design
### 2.3.1 core layer(You can consider it as the infrastructure layer.)
|module|status|
|:---:|:---:|
|[access](./core/access/Readme.md)|Draft/WIP|
|[filecoin](./core/filecoin/Readme.md)|Draft/WIP|
|[filplus](./core/filplus/Readme.md)|Draft/WIP|
|[carstore](./core/carstore/Readme.md)|Draft/WIP|
|[token](./core/token/Readme.md)|Draft/WIP|
|[governance](./core/governance/Readme.md)|Missing|

### 2.3.2 module layer(You can consider it as the domain layer.)
|module|status|
|:---:|:---:|
|[dataset](./module/dataset/Readme.md)|Draft/WIP|
|[matching](./module/matching/Readme.md)|Draft/WIP|
|[storage](./module/storage/Readme.md)|Draft/WIP|
|[datacap](./module/datacap/Readme.md)|Draft/WIP|
|[retrieve](./module/retrieve/Readme.md)|Missing|
|[compute](./module/compute/Readme.md)|Missing|
|[reputation](./module/reputation/Readme.md)|Missing|
|[reward](./module/reward/Readme.md)|Missing|

### 2.3.3 service layer(You can consider it as the domain service layer.)
|module|status|
|:---:|:---:|
|[dataswapstorage](./service/dataswapstorage/Readme.md)|Draft/WIP|
|[dataswapretrieve](./service/dataswapretrieve/Readme.md)|Missing|
|[dataswapcompute](./service/dataswapcompute/Readme.md)|Missing|
