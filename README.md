# Dataswap specs

This is the Dataswap Specification, a repository that contains documents, code, models, and diagrams that constitute the specification

Dataswap is a blockchain-based Layer 2 project built on [IPFS](https://ipfs.tech/) and [Filecoin](https://filecoin.io/), functioning as a decentralized open data exchange platform. Its goal is to aggregate open datasets from various regions and industries globally, enabling the permanent storage of valuable human data. Additionally, Dataswap offers comprehensive and reliable services for data retrieval, downloading, and analysis. Through these efforts, it aims to facilitate data sharing and collaborative progress for humanity.

## Spec Status

### I. Spec Status Legend

|Spec state|Label|
|:---:|:---:|
|Unlikely to change in the foreseeable future.|<font color="blue">Stable</font>|
|All content is correct. Important details are covered.|<font color="green">Reliable</font>|
|All content is correct. Details are being worked on.|<font color="yellow">Draft/WIP</font>|
|Do not follow. Important things have changed.|<font color="orange">Incorrect</font>|
|No work has been done yet.|<font color="red">Missing</font>|

### II.Spec Status Overview

|Section|State|Theory Audit|
|:---|:---:|:---:|
|[1 Introduction](./introduction/)|<font color="yellow">Draft/WIP</font>|
|[2 Systems](./systems/)|<font color="yellow">Draft/WIP</font>|
|[3 Libraries](./libraries/)|<font color="yellow">Draft/WIP</font>|
|[4 Algorithms](./algorithms/)|<font color="yellow">Draft/WIP</font>|
|[5 Glossary](./glossary/)|<font color="yellow">Draft/WIP</font>|

### III. Implementations Status

|Repo|Language|CI|Test Coverage|Security Audit|
|:---:|:---:|:---:|:---:|:---:|
|[core](https://github.com/dataswap/core)|solidity||||
|[go-metadata](https://github.com/dataswap/go-metadata)|golang||||
|[go-merkletree](https://github.com/dataswap/go-merkletree)|golang||||
|[generate-car](https://github.com/dataswap/generate-car)|golang||||
|[go-unixfs](https://github.com/dataswap/go-unixfs)|golang||||
|[dataswap.js](https://github.com/dataswap/dataswap.js)|ts||||
|[explorer](https://github.com/dataswap/explorer)|ts||||
|[web](https://github.com/dataswap/web)|ts||||

## Installation

To build the spec website you need

```shell
To be added
```

## Contribute

Dataswap is a universally open project and welcomes contributions of all kinds: code, docs, and more. However, before making a contribution, we ask you to heed these recommendations:

* If the change is complex and requires prior discussion, [open an issue](https://github.com/dataswap/specs/issues). This is to avoid disappointment and sunk costs, in case the change is not actually needed or accepted.

* Please refrain from submitting [PRs](https://github.com/dataswap/specs/pulls) to adapt existing code to subjective preferences. The changeset should contain functional or technical improvements/enhancements, bug fixes, new features, or some other clear material contribution. Simple stylistic changes are likely to be rejected in order to reduce code churn.

When implementing a change:

* The spec is written in markdown. [writing norms](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
* Stick to the idioms and patterns used in the codebase. Familiar-looking code has a higher chance of being accepted than eerie code. Pay attention to commonly used variable and parameter names, avoidance of naked returns, error handling patterns, etc.

* Minimize code churn. Modify only what is strictly necessary. Well-encapsulated changesets will get a quicker response from maintainers.

* Title the PR in a meaningful way and describe the rationale and the thought process in the PR description.

* Write clean, thoughtful, and detailed [commit messages](https://chris.beams.io/posts/git-commit/). This is even more important than the PR description, because commit messages are stored _inside_ the Git history. One good rule is: if you are happy posting the commit message as the PR description, then it's a good commit message.

## License

This project is licensed under [GPL-3.0-or-later](https://www.gnu.org/licenses/gpl-3.0.en.html).
