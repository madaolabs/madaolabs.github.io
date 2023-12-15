---
title: 解读 BTC 的地址类型
date: 2023-12-14 16:42:09
tags:
---

BTC 的交易是基于 UTXO 系统，本文不介绍 UTXO，不知道的同学自行查阅学习。

**比特币主网的地址类型分为 4 中:**

1. 1...（Pay-to-Public-Key-Hash（P2PKH）类型的脚本）
2. 3...（Pay-to-Script-Hash (P2SH) 类型的脚本）
3. bc1q...（以 bc1q 开头）
4. bc1p...（以 bc1p 开头）

每一类地址背后对应一种 BTC 的技术升级，下面逐个分析每一类地址用到的技术。

#### 以 1 开头的地址（P2PKH）

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

#### 以 3 开头的地址（P2SH）

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