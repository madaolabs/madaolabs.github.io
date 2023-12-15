---
title: 解读 BTC 的地址类型
date: 2023-12-14 16:42:09
tags:
---

BTC 的交易是基于 UTXO 系统，本文不介绍 UTXO，不知道的同学自行查阅学习。

**比特币主网的地址类型分为 4 中:**

1. 1...（Pay-to-Public-Key-Hash（P2PKH）类型的脚本）
2. 3...（Pay-to-Script-Hash (P2SH) 类型的脚本）
3. bc1q...（Segwit 隔离见证地址）
4. bc1p...（Taproot）

每一类地址背后对应一种 BTC 的技术升级，下面逐个分析每一类地址用到的技术。

#### **以 1 开头的地址（P2PKH）**

这是 BTC 最原始的地址，此类地址功能非常纯粹，只能做转账。如交易[062705...c2f2](https://www.blockchain.com/explorer/transactions/btc/0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2)

下面是该交易的详细数据，通过这个交易我们需要学习到 2 个概念: **解锁脚本和锁定脚本（scriptSig 和 scriptPubKey）**

```json
{
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
      "vout": 0,
      "scriptSig": "483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf",
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.015,
      "scriptPubKey": "OP_DUP OP_HASH160 ab68025513c3dbd2f7b92a94e0581f5d50f654e7 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": 0.0845,
      "scriptPubKey": "OP_DUP OP_HASH160 7f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a8 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
```

**vin** 交易输入标识哪个 UTXO（通过引用）将被消费，并通过解锁脚本（scriptSig）提供所有权证明。其中 txid 和 vout 表示用户上 1 笔收款交易，可以索引到一条 UTXO 列表中的数据。scriptSig 用于解锁 UTXO 中的数据。

**vout** 表示这个交易的输出。

#### **以 3 开头的地址（P2SH）**

上面的信息中在 P2PKH 的交易的解锁脚本（scriptSig）是用来解锁上一笔交易的输出的锁定脚本（scriptPubKey）。也就是说上一笔交易的锁定脚本（scriptPubKey）中的逻辑影响了本次交易的解锁脚本（scriptSig）。比如本次交易是多签转账，那么要求上一次交易的输出就要是多签的逻辑。

假设迪拜的电子产品进口商 Mohammed 对所有客户付款（会计术语称为“应收账款”）都使用多重签名脚本。基于多重签名方案，客户支付的任何款项都会被锁定，必须至少 2 个签名才能解锁，一个来自 Mohammed，另一个来自其合伙人或拥有备份密钥的律师。这样的多重签名机制能提升公司治理管控，同时也能有效防范盗窃、挪用和丢失。

那么 **Mohammed 必须在客户付款前将上面的脚本发送给每一位客户，而每一位客户也必须使用专用的能创建自定义交易脚本的比特币钱包软件，每位客户还得学会如何利用自定义脚本来创建交易。** 此外，由于脚本可能包含特别长的公钥，最终的交易脚本可能是最初交易脚本长度的 5 倍之多。超大的交易还将给客户造成费用负担。

P2SH 正是为了解决这一实际难题而被引入的，它使复杂脚本的使用能与直接向比特币地址支付一样简单。

具体实现如下：

1. 把复杂锁定脚本(这里给它一个新的名字叫: redeem script)通过两次哈希（sha256 和 ripemd160）

```shell
echo \
2 \
[04C16B8698A9ABF84250A7C3EA7EEDEF9897D1C8C6ADF47F06CF73370D74DCCA01CDCA79DCC5C395D7EEC6984D83F1F50C900A24DD47F569FD4193AF5DE762C587] \
[04A2192968D8655D6A935BEAF2CA23E3FB87A3495E7AF308EDF08DAC3C1FCBFC2C75B4B0F4D0B1B70CD2423657738C0C2B1D5CE65C97D78D0E34224858008E8B49] \
[047E63248B75DB7379BE9CDA8CE5751D16485F431E46117B9D0C1837C9D5737812F393DA7D4420D7E1A9162F0279CFC10F1E8E8F3020DECDBC3C0DD389D9977965] \
[0421D65CBD7149B255382ED7F78E946580657EE6FDA162A187543A9D85BAAA93A4AB3A8F044DADA618D087227440645ABE8A35DA8C5B73997AD343BE5C2AFD94A5] \
[043752580AFA1ECED3C68D446BCAB69AC0BA7DF50D56231BE0AABF1FDEEC78A6A45E394BA29A1EDF518C022DD618DA774D207D137AAB59E0B000EB7ED238F4D800] \
5 CHECKMULTISIG \
| bx script-encode | bx sha256 | bx ripemd160
54c557e07dde5bb6cb791c7a540e0a4796f5e97e
```

2. 把上一步的计算结果进行 Base58Check 编码，由于 P2SH 地址采用 5 作为版本前缀，所以得到以 “3” 开头的地址, 告诉用户往这个地址转账。

```shell
echo \
'54c557e07dde5bb6cb791c7a540e0a4796f5e97e'\
 | bx address-encode -v 5
39RF6JqABiHdYHkfChV6USGMe6Nsr66Gzw
```

3. 最终锁定脚本（scriptPubKey）变成

```shell
OP_HASH160 54c557e07dde5bb6cb791c7a540e0a4796f5e97e OP_EQUAL
```

4. 对应的解锁脚本也是需要修改的，改为

```shell
<Sig1> <Sig2> < redeem script >
```

我们再来看看是否可以解决 P2PKH 的问题，上一笔交易的输出只需要知道用户的地址就好了，并不需要知道其他，同时上一笔交易的锁定脚本的大小也得到了减少，费用转移到本次交易。P2SH 很好的解决 P2PKH 带来的问题。

#### **bc1q（Segwit 隔离见证地址）**

正式交易如： [ec9f...d959](https://www.blockchain.com/explorer/transactions/btc/ec9f03d79de1b408a2880e77b7be67c149ddb5e89c5b8c5a648fe29f4524d959)
无论是 P2PKH，还是 P2SH，都会把脚本写在交易中，**Segwit** 把 **解锁脚本** 从交易中剔除，那么区块中可以容纳更多的交易。“见证”是密码学中的名词，含义是解锁方案，在我们这里叫做解锁脚本。这部分数据占据原交易的约 75%。矿工在验证输入后可以删除 witness

在 Segwit 升级后，P2PKH 和 P2SH 变为了 P2WPKH 和 P2WSH。在收款方的地址是 bc1q 的情况下。

P2WPKH 的具体的技术实现：

1. 上一笔交易的输出变为：见证版本号为 0 + 公钥哈希值(或者 reedom script 的 hash)

```shell
// P2PKH 的样子
OP_DUP OP_HASH160 ab68025513c3dbd2f7b92a94e0581f5d50f654e7 OP_EQUALVERIFY OP_CHECKSIG
// P2WPKH
0 ab68025513c3dbd2f7b92a94e0581f5d50f654e7
```

2. 本次交易的输入不再用 scriptSig

```shell
// P2PKH
"vin": [
    {
      "txid": "7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
      "vout": 0,
      "scriptSig": "483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf",
      "sequence": 4294967295
    }
  ],
// P2WPKH
"vin": [
    {
      "txid": "7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
      "vout": 0,
      "scriptSig": "",
      "sequence": 4294967295,
      "witness": "<Bob' Signature> <Bob's PublicKey>" 或者 "<SigA> <SigB> <2 PubA PubB PubC PubD PubE 5 CHECKMULTISIG>"
    }
  ],
```

#### **bc1p...（Taproot 地址）**

真实交易: [3ef65...2e29](https://www.blockchain.com/explorer/transactions/btc/3ef659aff282acc1f677b7c003a6edd07b810d85eaf3cb81210bd11ea3142e29)
Taproot 在 Segwit 的基础上进一步减小了存储空间，提高了交易效率，并提供了更好的隐私性, Taproot 升级包含 3 个升级:

1. 默克尔抽象语法树(MASK)
2. Schnorr 签名
3. Taproot（MASK 和 Schnorr 签名落地应用）

##### 默克尔根和哈希树是如何生成的？

首先分别对所有脚本（条件）做哈希计算；然后将计算得到的哈希值与相邻哈希值组合起来进行哈希计算，生成一组新的哈希值。不断重复这个两两哈希计算的过程，直到计算出最后一个哈希值为止。这个哈希值就是默克尔根。

假设共有四组条件。首先，分别计算出这四组条件的哈希值；再将这四个哈希值两两配对，计算出两个哈希值；最后，把这两个哈希值组合起来做哈希计算，生成最终的哈希值。最后这个哈希值就是默克尔根。

![](mask.webp)

**这个默克尔根可以翻译成一个能够接收付款的有效比特币地址**，即，默克尔比特币地址（Merklized Bitcoin address）。默克尔比特币地址有很多优点，最主要的优点是无需知晓所有脚本单元就能验证某个脚本是否位于这棵默克尔树上。这个技术叫作默克尔证明（Merkle Proof），可以用来轻松验证一个比特币 UTXO 是否包含某些解锁条件。

在 MAST 中，BTC 与一棵默克尔树绑定。这棵默克尔树指定了可以解锁未花费 BTC 的所有复杂条件。每个叶节点都代表着一个条件。为了解锁 BTC，你必须生成一个满足默克尔树上某个分支所代表的条件的脚本。仅使用默克尔根即可验证这个条件是否属于原始条件集合（即验证某个叶节点是否在这棵默克尔树上）。一旦比特币区块链网络发现某个脚本（及其所代表的条件）属于这个默克尔根，网络就会知道这个脚本是这些比特币的锁定条件并开始验证解锁脚本。因此，我们无需生成完整的脚本并将其包含到交易内，即可花费以 MAST 锁定的 BTC。这有助于减少 BTC 交易的体积。

##### Schnorr 签名

在密码学中，Schnorr 签名是由 Claus Schnorr 提出的 Schnorr 签名算法生成的数字签名。 Schnorr 签名算法是一种以简单闻名的数字签名方案，通过将多个签名聚合成单个签名以优化验证和认证过程。该方案适用于多签交易。

若想执行交易，你需要使用私钥签名该交易，以证明你是某个公钥背后的 BTC 的所有者。但是，若想执行多签交易，你必须提供多个签名。这些签名会占据额外的空间。

以 12/20 多签交易为例。12/20 指的是执行该交易至少需要提供 20 个签名中的任意 12 个。签署交易时，签名也会存储在区块内。假设 1 个签名的大小是 5 字节，12 个签名需要占用区块 60 字节（5\*12 = 60）的内存，100 个签名需要占用 500 字节的内存。这会增加内存用量。Schnorr 签名恰好可以解决这一问题。

为了理解 Schnorr 签名，我们来看两个例子：

（……）
另一种情况是多签交易。假设你需要 100 个签名且每个签名的大小是 5 字节，Schnorr 签名方案可以将这 100 个签名合并成一个大小为 64 字节的 Schnorr 签名。省下 436 字节（5\*100-64=436）的内存可以用来存储更多交易。（注：现在的椭圆曲线签名可不止 5 字节）

##### Taproot

沿用 Maxwell 原文中的例子：假设两个用户各有公钥 A、B，两人聚合公钥 A + B = C，再生成最终公钥 P = C + H(C||S)\*G，其中 S 为自定义的脚本。就以这个最终公钥 P 来定义资金的解锁条件。假设两个用户都在线，他们很容易可以共同使用这笔资金，只要其中一方在签名时在自己的私钥里加上 H(C||S) 即可；如果只有其中一方在线，比如 S 定义了 B 可以花费资金的条件，Taproot 的规则使得公钥 B 用户可通过揭示聚合公钥 P 以及 H(C||S) 并提供可以满足 S 的条件来使用资金。

总而言之，在 Taproot 之后，他人将无法从地址形式上分辨一个 P2TR 地址到底是个人用户还是合约用户；由于 Schnorr 签名的效果，当这个地址里的资金使用单签名来解锁时，他人将无法分辨这到底是一个人在使用，还是 n 个人一起使用，也无法知道这个地址是否还有自定义的脚本；由于 MAST 的效果，当用户使用自定义的脚本来花费资金时，只需暴露需要用到的部分脚本；他人虽然知道了这个地址有自定义的脚本，但整个脚本到底包括哪些条件，仍然是不可知的。

具体技术实现与 segwit 相似，见证版本号改为 1
