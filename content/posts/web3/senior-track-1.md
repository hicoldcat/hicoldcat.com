---
title: Web3系列教程之高级篇---1：默克尔树
description: null
author: 李留白
weight: 0
date: 2022-07-31T13:20:08.487Z
lastmod: 2022-07-31T13:35:52.302Z
tags: []
categories:
  - 区块链
  - WEB3.0
featuredImage: https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/R3RYera.png
---

默克尔树是区块链技术的一个基本概念。它们是一种特殊的二进制树，用于编码大块的信息。默克尔树的酷之处在于，它们是一种自下而上的 "建立"，并允许你验证某些值是否存在于树中，而不需要在树的每个元素上循环。正如我们会看到的，这可能非常有用。🧐

## 什么是默克尔树？

默克尔树是一种哈希树，其中每个叶子节点都标有数据块的加密哈希值，而每个非叶子节点都标有其子节点的加密哈希值的标签。大多数哈希树的实现是二进制的（每个节点有两个子节点），但它们也可以有更多的子节点。

一个典型的默克尔树看起来是这样的。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220731212643.png)

(引用自 [using-merkle-trees-for-nft-whitelists](https://medium.com/@ItsCuzzo/using-merkle-trees-for-nft-whitelists-523b58ada3f9))

让我解释一下发生了什么。树的所有叶节点，即没有任何其他子节点的节点，都包含您要编码的数据的哈希值。请注意，您要在树中编码的值始终只是叶节点的一部分。由于它是二叉树，每个非叶节点都有两个孩子。当您从叶节点向上移动时，父节点将拥有叶节点组合哈希的哈希，依此类推。

当您继续这样做时，最终您将到达单个顶级节点，称为 Merkle 树根，这将发挥非常重要的作用。

## 简单示例

假设我们有 4 个事务：“事务 A”、B、C 和 D。它们都在同一个块中执行。这些交易中的每一个都将被散列。让我们分别称这些哈希为“哈希 A”、B、C 和 D。

以下是这些交易的 Merkle Tree：

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220731212936.png)

## 使用 Merkle Root 验证有效性

当这些交易汇总到一个块中时，块头将包含 Merkle Root，Hash ABCD。到目前为止，所有矿工都拥有所有交易的副本，因此拥有所有交易哈希。任何矿工都可以按需重建 Merkle 树，这意味着每个矿工都可以针对同一组交易独立到达同一个 Merkle 根。

这允许任何矿工验证欺诈交易。假设有人试图引入虚假交易而不是交易 D。让我们将此称为交易 E。因为此交易与交易 D 不同，所以哈希也会不同。Transaction E的Hash就是Hash E。C和E的Hash加在一起就是Hash CE，与Hash CD不同。当哈希 AB 和 CE 一起哈希时，你得到哈希 ABCE。由于哈希 ABCE 与哈希 ABCD 不同，我们可以得出交易 E 是欺诈性的结论。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220731213049.png)

矿工可以在自己的区块中重新计算 Merkle Root 并尝试将该版本发布到区块链，但由于每个其他矿工都有不同的 Merkle Root，因此欺诈矿工很容易被拒绝。

## 哈希函数

在谈论 IPFS 时，我们之前已经介绍过哈希函数，但只是回顾一下：将事务 A 哈希到哈希 A 中，使用了单向加密哈希函数。一旦散列，Hash A 就不能变成 Transaction A；哈希是不可逆的。

每个区块链使用不同的哈希函数，但它们都具有相同的共同属性。

#### 确定性

当传递给散列函数时，相同的输入总是具有相同的输出。

#### 计算效率高

计算输入值的哈希值很快。

#### 无法逆向工程

给定一个结果哈希，几乎不可能确定输入。即散列函数是单向函数。例如：给定`y`，很难找到`x`这样的`h(x) = y`

#### 防撞

两个不同的输入永远不会产生相同的输出。

## 区块链中默克尔树的好处

Merkle Trees 允许快速验证数据完整性。

与整个事务集相比，已用完的磁盘空间非常少。出于这个原因，Merkle Root 包含在块头中。

如果你有两组不同的交易，用默克尔树验证它们是否相同比验证每一个单独的交易要快。只需知道 Merkle Root，就可以验证一个块没有被修改。

## 区块链之外的用例

Merkle 树不仅用于区块链应用程序。一些使用 Merkle 树的流行应用程序是：

- [IPFS](https://en.wikipedia.org/wiki/InterPlanetary_File_System)
- [Github](https://github.com/)
- [AWS DynamoDB](https://aws.amazon.com/dynamodb)和[Apache Cassandra](https://cassandra.apache.org/_/index.html)等分布式数据库使用 Merkle 树来控制差异

## 验证 Merkle 树中的存在

那么，我们如何真正验证某些数据是默克尔树的一部分呢？

您不希望验证器遍历 Merkle 树的每个叶节点，因为它可能非常大，那么我们如何以更有效的方式做到这一点？

假设`Verifier`唯一有`Merkle Root` `r`，即树的顶级父节点。你，作为一个`Prover`，想要证明默克尔树中存在`Verifier`一些价值。`K`

为此，您可以生成一个`Merkle Proof`. `Merkle Proof`让我们尝试通过示例 Merkle 树来了解 a 的实际含义。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220731213210.png)

（参考[默克尔证明解释](https://medium.com/crypto-0-nite/merkle-proofs-explained-6dd429623dc5)）

主要思想如下：如果您可以给出 的`Verifier`值`K`，以及树中的所有相关节点，这些节点被散列在一起以构建`r`散列，则`Verifier`可以将计算的根值与`r`它们已经拥有的值进行比较。如果它们是相同的散列，那一定意味着它`K`实际上存在于 Merkle 树中，因为您不可能使用不同的输入数据生成相同的 Merkle Root 散列。

在上图中，让我们考虑必须向验证者提供哪些信息，才能积极地向`K`作为默克尔树一部分的验证者证明。

- 自身的价值`K`（因此验证者可以`H(K)`自己计算）
- `H(L)`，因此验证者可以计算`H(KL)`
- `H(IJ)`所以验证者可以计算`H(IJKL)`
- `H(MNOP)`所以验证者可以计算`H(IJKLMNOP)`
- `H(ABCDEFGH)`所以验证者可以计算`H(ABCDEFGHIJKLMNOP)`

再次重要的是要记住，只有一个给定的节点组合可以生成这个唯一的根`r`，因为 Merkle 树是一个 `collision-resistant hash function`，这意味着它是一个哈希函数，给定两个输入几乎不可能产生相同的输出。

对于我们给定的示例，我们只需要提供以下节点即可证明 H[K] 确实存在于我们的节点中：

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220731213233.png)

此时，如果 的计算值与 Verifier`H(ABCDEFGHIJKLMNOP)`先前已知的值匹配`r`，则在 Merkle Tree 中存在一定为真，`K`否则哈希值将不一样。

这*比*遍历整个 Merkle 树要高效得多，因为对于具有`n`多个元素的树，您只需提供粗略`log(n)`的元素作为证明的一部分（树的每个“级别”一个）。这意味着如果您有大量数据，Merkle 树比存储数组或映射更有效。

> 当 ENS 启动他们的代币合约时，他们将 $ENS 代币空投到超过 100,000 个钱包地址。他们能够在汽油费极高的情况下以比将钱包地址存储在数组中的价格低得多的价格部署他们的合约（即使存储几百个地址也很容易超过汽油费）块的限制） - https://etherscan.io/tx/0xdfc76788b13ab1c033c7cd55fdb7a431b2bc8abe6b19ac9f7d22f4105bb43bff

## 智能合约中的用例

由于验证者不需要存储整个 Merkle Tree 来验证是否是其中的一部分，Merkle Trees 实际上对于某些事情非常方便。

在大二，我们创建了一个将用户地址存储在映射中的白名单 dApp。虽然这种方法有效，但在智能合约存储中存储数据是迄今为止你可以做的最昂贵的事情，就 gas 而言。那么如果你必须存储 1000 个地址呢？如果是一万呢？10万呢？🤯

到那时，直接使用智能合约存储是不可行的，仅仅将人们列入白名单就很容易花费数百万美元。另一方面，你可以建立一个默克尔树，然后将默克尔根值存储在合约中——一个微不足道的`bytes32`值。在这种情况下，合约现在是`Verifier`，并且希望使用他们的白名单来铸造 NFT 的用户，比方说，成为`Provers`他们确实是白名单的一部分的证明。让我们看看这是如何工作的。

## 建造

### 先决条件

- 如果您不了解 Mocha 和 Chai，请学习它们的基础知识，以了解它们是什么，请遵循本[教程](https://medium.com/spidernitt/testing-with-mocha-and-chai-b8da8d2e10f2)

让我们看看这一切在我们的白名单示例中是如何实际工作的。

- 要设置 Hardhat 项目，请打开终端并执行以下命令

```bash
  npm init --yes
  npm install --save-dev hardhat
```

- 如果您在 Windows 机器上，请执行此额外步骤并安装这些库：)

```bash
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers

```

- 在安装 Hardhat 的同一目录中运行：

```bash
npx hardhat
```

- 选择`Create a basic sample project`
- 按回车键已指定`Hardhat Project root`
- 如果您想添加一个问题，请按 Enter 键`.gitignore`
- 按回车键`Do you want to install this sample project's dependencies with npm (@nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers)?`

现在您的Hardhat文件夹已设置完毕。

我们还需要安装一些额外的依赖项来执行一切。因此，再次在指向根目录的终端中执行以下命令：

```bash
npm install @openzeppelin/contracts keccak256 merkletreejs
```

`contracts`现在首先在您的文件夹中创建一个名为的文件`Whitelist.sol`并将以下代码行添加到其中

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

contract Whitelist {

    bytes32 public merkleRoot;

    constructor(bytes32 _merkleRoot) {
        merkleRoot = _merkleRoot;
    }

    function checkInWhitelist(bytes32[] calldata proof, uint64 maxAllowanceToMint) view public returns (bool) {
        bytes32 leaf = keccak256(abi.encode(msg.sender, maxAllowanceToMint));
        bool verified = MerkleProof.verify(proof, merkleRoot, leaf);
        return verified;
    }
    
}
```

这里到底发生了什么？因此，正如我们提到的，我们不会在合约中存储每个用户的地址，而是只存储在构造函数中初始化的默克尔树的根。

我们还有另一个函数`checkInWhitelist`，它接受 a`proof`和`maxAllowanceToMint`。 `maxAllowanceToMint`是一个变量，用于跟踪给定地址可以铸造的 NFT 数量。

对于这个用例，我们实际存储在 Merkle 树中的值是存储用户的地址以及他们被允许铸造的 NFT 数量。您可以在 Merkle Trees 中存储您想要的任何数据，但这适用于我们的示例。该地址所在的叶节点的哈希值可以通过首先将发送者的地址和`maxAllowanceToMint`字节字符串编码来计算，该字符串进一步传递给`keccak256`哈希函数，哈希函数需要哈希字符串来生成哈希值。

现在我们使用 OpenZeppelin 的`MerkleProof`库来验证用户发送的证明确实有效。请注意，Openzeppelin 如何执行高级别的验证类似于我们在教程前面讨论的 Merkle 证明的验证。

接下来，让我们编写一个测试来帮助确定我们合约中的代码是否真的有效。

在您的`test`文件夹中创建一个新文件`merkle-root.js`并将以下代码行添加到其中

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");
const keccak256 = require("keccak256");
const { MerkleTree } = require("merkletreejs");

function encodeLeaf(address, spots) {
  // Same as `abi.encodePacked` in Solidity
  return ethers.utils.defaultAbiCoder.encode(
    ["address", "uint64"],
    [address, spots]
  );
}

describe("Check if merkle root is working", function () {
  it("Should be able to verify if a given address is in whitelist or not", async function () {
  
    // Get a bunch of test addresses
    const [owner, addr1, addr2, addr3, addr4, addr5] =
      await ethers.getSigners();
    
    // Create an array of elements you wish to encode in the Merkle Tree
    const list = [
      encodeLeaf(owner.address, 2),
      encodeLeaf(addr1.address, 2),
      encodeLeaf(addr2.address, 2),
      encodeLeaf(addr3.address, 2),
      encodeLeaf(addr4.address, 2),
      encodeLeaf(addr5.address, 2),
    ];

    // Create the Merkle Tree using the hashing algorithm `keccak256`
    // Make sure to sort the tree so that it can be produced deterministically regardless
    // of the order of the input list
    const merkleTree = new MerkleTree(list, keccak256, {
      hashLeaves: true,
      sortPairs: true,
    });
    // Compute the Merkle Root
    const root = merkleTree.getHexRoot();

    // Deploy the Whitelist contract
    const whitelist = await ethers.getContractFactory("Whitelist");
    const Whitelist = await whitelist.deploy(root);
    await Whitelist.deployed();

    // Compute the Merkle Proof of the owner address (0'th item in list)
    // off-chain. The leaf node is the hash of that value.
    const leaf = keccak256(list[0]);
    const proof = merkleTree.getHexProof(leaf);

    // Provide the Merkle Proof to the contract, and ensure that it can verify
    // that this leaf node was indeed part of the Merkle Tree
    let verified = await Whitelist.checkInWhitelist(proof, 2);
    expect(verified).to.equal(true);
    
    // Provide an invalid Merkle Proof to the contract, and ensure that
    // it can verify that this leaf node was NOT part of the Merkle Tree
    verified = await Whitelist.checkInWhitelist([], 2);
    expect(verified).to.equal(false);
  });
});
```

在这里，我们首先让一些签名者使用 hardhat 的扩展 ethers 包进行测试。

然后我们创建一个节点列表，这些节点都使用`ethers.utils.defaultAbiCoder.encode`

使用我们输入列表中的`MerkleTree`类，指定我们的散列函数，并将节点的排序设置为`merkletreejs``keccak256``true`

创建 后`Merkle Tree`，我们通过调用`getHexRoot`函数来获取它的根。我们使用这个根来部署我们的`Whitelist`合约。

在我们的合约被验证后，我们可以`checkInWhitelist`通过提供证明来调用我们的合约。所以现在在这里我们将检查`(owner.address, 2)`我们的数据集中是否存在。为了生成证明，我们对 的编码值进行哈希处理，并使用库中的函数`(owner.address, 2)`生成证明 。`getHexProof``merkletreejs`

然后将该证明`checkInWhitelist`作为参数发送，该参数进一步返回 true 值以表示`(owner.address, 2)`存在。

要运行测试，请从目录的根目录执行以下命令：

```bash
npx hardhat test
```

如果你所有的测试都通过了，你就成功地了解了 Merkle 树是什么以及它如何用于白名单 🥳 🥳 🥳

希望你玩得开心！！

干杯🥂

> 原文：[https://www.learnweb3.io/tracks/senior/merkle-trees](https://www.learnweb3.io/tracks/senior/merkle-trees)

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/my.png)