---
title: 解读 BTC 的地址类型
date: 2023-12-14 16:42:09
tags:
---

BTC 的交易是基于 UTXO 系统，本文不介绍 UTXO，不知道的同学自行查阅学习。

**比特币主网的地址类型分为 4 中:**

1. 1...（以 1 开头）
2. 3...（以 3 开头）
3. bc1q...（以 bc1q 开头）
4. bc1p...（以 bc1p 开头）

每一类地址背后对应一种 BTC 的技术升级，下面逐个分析每一类地址用到的技术。

#### 以 1 开头的地址（1...）

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

#### 以 3 开头的地址（3...）
