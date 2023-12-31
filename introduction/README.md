# 1. Introduction
## 1.1 Features

- Proposing [Dataset Consistency Algorithms](../algorithms/README.md#2-dataset-consistency-algorithm), including dataset proof submission and validation, to implement a fine-grained management process for datacap issuance while simultaneously ensuring the accurate loading of datasets.

- Leveraging blockchain technology for permanent storage (using Filecoin) and distribution of datasets, establishing a transparent and publicly accessible distributed data index.

- Offering open retrieval and download services, enabling users to effortlessly search for and obtain the datasets they need. This encompasses diverse access methods, such as web interfaces, API integrations, and file downloads.

- Providing decentralized data analysis and matching services, empowering data-driven decision-making and intelligent solutions.

- Aggregating open big data from various global regions and industries, encompassing economic, financial, medical, and health data types. This creates efficient and valuable gateways to datasets.

- Implementing a decentralized matching mechanism to attract more suppliers of open datasets, storage providers, and users, fostering worldwide data sharing and innovation.

## 1.2 Architecture Diagrams

### 1.2.1 Layered Architecture Diagram
#### 1.2.1.1 High-Level Architecture 
![img](./img/architecture.png)

Layer2(dataswap storage) meets the requirements of [Ideation: Trustless Notary Design Space + Guidelines](https://medium.com/filecoin-plus/ideation-trustless-notary-design-space-guidelines-bc21f6d9d5f2)

#### 1.2.1.2 Dataswap contractArchitecture 
![](../systems/img/contractArchitecture.png)

### 1.2.2 Data Lifecycle Diagram
![](./img/dataLifecycle.png)

### 1.2.3 DataSwap Operational Simulation on the Filecoin Network

[DataSwap Operational Simulation on the Filecoin Network](https://observablehq.com/d/eda624edd4751495)

## 1.3 Key Concepts