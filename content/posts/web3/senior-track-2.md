---
title: Web3系列教程之高级篇---2：以太坊存储和执行
description: null
author: 李留白
weight: 0
date: 2022-08-01T13:53:02.116Z
lastmod: 2022-08-01T14:32:52.603Z
tags: []
categories:
  - 区块链
  - WEB3.0
featuredImage: https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/R3RYera.png
---

在过去的几个教程上，我们一直在编写智能合约，并简要地提到，以太坊智能合约在这个叫做以太坊虚拟机（EVM）的东西中运行。

我们还顺便简单地提到，EVM能够运行某些OPCODES，并处理存在于堆栈或堆中的数据。如果你有正式的计算机科学背景，这对你来说可能是有意义的，但对其他人来说，这实际上意味着什么？🤔

在这一层次，我们将更深入地挖掘EVM执行引擎，以及在整个交易过程中如何存储、操作和运行数据。

## 回顾

在继续学习之前，让我们回顾一下我们在先前的课程中所教授的一些东西。

回顾一下，以太坊作为基于事务的状态机工作。从某个状态开始`s1`，事务处理某些数据以将世界状态转换为某个状态`s2`。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801220258.png)

为了将事情分组，事务被打包成块。一般来说，每个区块将世界状态从状态s1改变为s2，而转换是根据区块内每个事务所做的状态改变来计算的。

当我们想到这些状态变化时，以太坊可以被认为是一个状态链。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221115.png)

但是，这个世界的状态是什么？🤨

## 世界状态

以太坊的世界状态是地址和账户状态之间的映射。以太坊上的每个地址都有它自己的状态，这可能是一个用户账户（EOA）或一个智能合约。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221246.png)

每个区块本质上都操纵着多个账户状态，从而操纵着以太坊的整体世界状态。

## 账户状态

好吧，那么世界状态是由各种账户状态组成的。什么是账户状态？

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221424.png)

账户状态包含一些常见的东西，如nonce和余额（以ETH为单位）。此外，智能合约还包含一个存储哈希和一个代码哈希。这两个哈希值作为对独立状态树的引用，分别存储状态变量和智能合约的字节码。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221448.png)

回顾一下，在以太坊有两种类型的账户。外部拥有的账户（如Coinbase钱包，Metamask钱包等）和智能合约账户。

EOA是由私钥控制的，没有任何EVM代码。另一方面，合同账户包含EVM代码，由代码本身控制，没有与之相关的私钥。

## 交易的类型

在以太坊上主要有两种类型的交易。一种是创建新合约，另一种是发送消息。

在这里发送消息意味着进行交易，要么转移ETH，要么调用智能合约的功能。它们只是EOA可以发送的不同类型的消息。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221636.png)

当一个合同创建交易被进行时，一个新的账户被添加到世界状态中。该事务带有要创建的合同的字节码和初始化代码（即构造函数调用）。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221700.png)

另一方面，对于所有其他交易，即消息调用，现有账户的账户状态在交易后被修改。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221716.png)

## 消息

以太坊的消息在两个账户之间传递。它们主要由两样东西组成 - `data`和`value`。

`data`是一组字节，表示需要进行的交易类型（转移ETH，铸造NFT，在DAO中投票，等等），`value`是与交易一起转移的以太币价值。

由EOA进行的交易会向收件人账户发送一个信息。合同账户也可以通过EVM代码向账户发送信息。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801221826.png)

## 以太坊虚拟机

现在我们来谈谈EVM的问题。

就像Java的JVM，Javascript和Python也有自己的运行时环境一样，以太坊智能合约的运行时环境就是EVM。

EVM有一个基于堆栈的架构。对现代CPU架构进行了大规模的简化。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222030.png)

智能合约代码，或EVM代码，存在于EVM内的一个不可变的存储位置。

对于运行时的计算，即局部变量之类的，EVM可以访问两个存储位置--堆栈和内存（即堆）。

EVM也可以访问持久的世界状态，即账户状态的读写，例如改变合同内的状态变量。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222144.png)

堆栈是一个简单的堆栈，支持PUSH/POP操作，每个堆栈元素为256位（32字节），最大深度为1024个元素。

内存（或堆）是一个线性内存结构，在运行期间可以存储动态大小的数据，即字符串和动态数组。

账户存储是世界状态的一部分，是持久性的存储，即使在事务执行完毕后，所做的任何改变都将继续留在这里。

## 栈

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222232.png)

堆栈是一种用于保存临时值的后进先出数据结构。可以把它想象成一摞盘子。你堆在上面的盘子，将是第一个被移除的。堆栈在整个计算机科学中被用来对固定大小的数据进行快速操作，EVM也不例外。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222343.png)

所有来自EVM的操作都在堆栈中运行。EVM堆栈支持对堆栈的前16个元素进行操作，但不支持更深的操作。其他1008个堆栈元素可用于存储操作数据，如要运行的OPCODES等。

> 有趣的事实：在Solidity中，如果你编写的函数中声明的局部变量超过16个，你会得到一个编译错误。因为堆栈不能处理超过前16个元素的数据，拥有超过16个变量意味着在EVM中不能对其中一些变量进行操作。

## 内存

EVM存储器是一个线性寻址的存储器，可以在字节级寻址。你可以在内存中一次存储8比特（1字节）或256比特（32字节），但只能以256比特（32字节）为单位从内存中读取。内存用于在实体中存储动态值，如可变长度数组、字符串等。

最初，所有内存位置的值都是零。然而，在事务执行过程中，这些值可以被更新和修改。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222459.png)

## 账户存储

持久账户存储是一个从256位键到256位值的映射。持久性存储中的所有位置最初也被定义为零（因此Solidity中整数的初始值为0，布尔运算为假，字符串为空，等等）。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222553.png)

这些映射中的键通常被称为槽。智能合约中的每个状态变量都被分配到账户存储中的一个槽，按照它们被定义的顺序。

因此，对于一个看起来像这样的合同：

```solidity
contract Sample {
    uint256 first;
    uint256 second;
    address third;
}
```

`first`将有0号插槽，`second`将有1号插槽，`third`将有2号插槽。

当我们在本专题后面开始学习Solidity中的`DELEGATECALL(.delegatecall())`时，槽的这个概念将变得非常重要。

## 执行模式

让我们来看看EVM中的高水平执行模型。这张图一开始可能有点混乱，但读完这一节，你就会明白发生了什么。

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222805.png)

EVM包含一个程序计数器（PC）。PC，有时也称为指令指针，是一个指向计算机在代码执行中的位置的数值。

如果你把EVM代码看成是一个要运行的指令列表，PC将指向需要运行的指令。最初，PC指向零，也就是第一条指令。当该指令被运行时，PC会被更新为指向下一条指令，以此类推。

被PC指向的指令用给定的数据执行某些操作。这些操作发生在堆栈中，堆栈可以从内存和账户存储中读/写值。

我以前用过这个比喻，我将再次使用它--把内存想象成你的RAM，把账户存储想象成你的硬盘。堆栈（指令处理器）可以从RAM和硬盘中读/写数据，但只有对硬盘数据所做的修改在代码运行结束后才会继续存在，而内存则会被清除掉。

到目前为止，这与实际的CPU架构相当相似。对于那些有正式计算机科学背景的人来说，如果你在大学里上过硬件或计算机处理器的课，你一定被教导过关于实际处理器如何工作的类似内容。EVM的行为也非常类似。

但是，这里有一件特别的事情。EVM还存储了一个计数器，用于计算有多少气体可用。EVM执行的每个操作都要花费一定量的气体，只要有足够的气体来运行操作，EVM就会继续执行操作。如果可用的气体低于继续运行所需的数量，整个执行将停止并导致交易失败。正如我们之前所讲，这样做是为了避免在EVM内出现无限循环，从而使以太坊网络陷入停顿。因此，对于复杂的交易，你需要支付更高的气体来覆盖执行成本。

## 执行过程中的Gas

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/20220801222921.png)

突出以上几点，你可以看到EOA在发送消息时，将一定量的气体传递给合约账户。EVM代码运行并耗尽部分气体。如果有剩余的气体，就会退还给EOA。

然而，如果EVM的代码耗尽了气体，即没有提供足够的气体，执行会失败，交易也会失败。在这种情况下，没有气体被退还，因为EVM仍然不得不执行所有这些操作，以弄清所供应的气体太少，所以气体被收取了所做的工作。

## 总结

以太坊是一个复杂的软件。如果你能走到这一步，为你点赞。我希望这一层能帮助你解开一些围绕以太坊存储如何工作的疑惑，以及EVM如何在运行期间处理数据和执行交易。

我们可以更深入地研究在EVM中运行的Assembly和OPCODES，但这值得单独写一篇（或更多）文章，因为这是一个巨大的话题。

## 参考

- [EVM Illustrated](https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf)


> 原文：[https://www.learnweb3.io/tracks/senior/eth-data-storage-execution](https://www.learnweb3.io/tracks/senior/eth-data-storage-execution)

![](https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/my.png)