---
title: Ordinal 协议
date: 2023-12-14 15:25:59
tags: BTC
---

下文需要对 BTC 的交易模型有所了解，不知道的同学请先看: [《解读 BTC 的地址类型以及交易模型》](https://madaolabs.github.io/2023/12/14/blockchain/btc/address/)

#### Ordinal 协议是什么？

Ordinal 协议是一段固定格式的文本。该文本叫做成为铭文（Inscriptions）并且数据存储在比特币 taproot script-path 花费脚本中。

#### 铸造过程

铸造的流程是一个固定的步骤分为两步(本质上是**两笔交易**):

1. 提交(commit)
2. 揭露(reveal)

**提交**的交易必须由 bc1p 的地址发起，输出脚本(锁定脚本)是 Taproot 的脚本: 见证版本号 1 + MASK root 的公钥，得益于 Taproot 升级此时只能在脸上看到这笔交易，但无法看到交易的内容

**揭露**的交易是消费上一笔交易的输出，在此交易中的输入包含了铭文数据，达到揭露的目的。

#### 真实案例

1. 提交：[43eed...20e5e](https://www.blockchain.com/explorer/transactions/btc/43eed3dac6f11b4b8f1d5d85b15dbbbe76ed8240f3053d9c44123289ac720e5e)
2. 接着提交揭露: [29110...3d511](https://www.blockchain.com/explorer/transactions/btc/2911040743b16b71c4c00dc2561b91dac87650e0957d8acd016da0ffd8d3d511)
