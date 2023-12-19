---
title: Ordinals 协议 和 BRC-20
date: 2023-12-14 15:25:59
tags: BTC, Ordinal
---

下文需要对 BTC 的交易模型有所了解，不知道的同学请先看: [《解读 BTC 的地址类型以及交易模型》](https://madaolabs.github.io/2023/12/14/blockchain/btc/address/)

#### Ordinals 协议是什么？

Ordinals 做了两个事情：

1. 给每一个 satoshi 一个序号
2. 将一段固定格式的文本结合**序号**。该文本叫做成为铭文（Inscriptions）并且数据存储在比特币 taproot script-path 花费脚本中。

Ordinals Theory：Ordinals + Inscriptions = Digital Artefacts。

#### 铸造过程

铸造的流程是一个固定的步骤分为两步(本质上是**两笔交易**):

1. 提交(commit)
2. 揭露(reveal)

**提交**的交易必须由 bc1p 的地址发起，输出脚本(锁定脚本)是 Taproot 的脚本: 见证版本号 1 + MASK root 的公钥，得益于 Taproot 升级此时只能在脸上看到这笔交易，但无法看到交易的内容

**揭露**的交易是消费上一笔交易的输出，在此交易中的输入包含了铭文数据，达到揭露的目的。

#### Ordinal 的脚本格式

OP_FALSE OP_IF … OP_ENDIF。 从 OP_IF 与 OP_ENDIF 之间的内容可以是任意内容

```shell
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "text/plain;charset=utf-8"
  OP_PUSH 0
  OP_PUSH "Hello, world!"
OP_ENDIF
```

#### Ordinals 的价值

Ordinals 的价值在于 satoshi 的稀有度，分为 6 种稀有：

1. common: 不是区块第一个 sat 的任何 sat
2. uncommon: 每个区块的第一个 sat
3. rare: 每个难度调整期（每 2016 个区块调整 1 次）的第一个 sat
4. epic: 每个减半周期（大约每四年）的第一个 sat
5. legendary：每个周期（减半和难度调整同时发生）的第一个 sat
6. mythic: 创世块的第一个 sat

#### BRC-20

是一段 json 格式的固定文本, 目前仅有部署、铸造和转帐三种功能

#### BRC-20 脚本案例

```shell
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "text/plain;charset=utf-8"
  OP_PUSH 0
  OP_PUSH "{p: "brc-20", op: "deploy", tick: "ordi", max: "21000000", lim: "1000"}"
OP_ENDIF
```

#### 真实案例

1. 提交：[43eed...20e5e](https://www.blockchain.com/explorer/transactions/btc/43eed3dac6f11b4b8f1d5d85b15dbbbe76ed8240f3053d9c44123289ac720e5e)
2. 接着提交揭露: [29110...3d511](https://www.blockchain.com/explorer/transactions/btc/2911040743b16b71c4c00dc2561b91dac87650e0957d8acd016da0ffd8d3d511)
