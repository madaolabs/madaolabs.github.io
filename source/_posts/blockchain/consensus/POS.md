---
title: POS Consensus
date: 2023-10-24 13:54:33
tags:
---

### 简介

Proof of Stake，股权证明
类似于财产储存在银行，这种模式会根据你持有数字货币的量和时间，分配给你相应的利息。
简单来说，就是一个根据你持有货币的量和时间，给你发利息的一个制度，在股权证明 POS 模式下，有一个名词叫币龄，每个币每天产生 1 币龄，比如你持有 100 个币，总共持有了 30 天，那么，此时你的币龄就为 3000，这个时候，如果你发现了一个 POS 区块，你的币龄就会被清空为 0。你每被清空 365 币龄，你将会从区块中获得 0.05 个币的利息(假定利息可理解为年利率 5%)，那么在这个案例中，利息 = 3000 \* 5% / 365 = 0.41 个币，这下就很有意思了，持币有利息。以太坊就是采用 POS 共识算法。

```golang{.line-numbers}
package main

import (
	"time"
	"strconv"
	"crypto/sha256"
	"math/rand"
	"fmt"
	"encoding/hex"
)

//实现pos挖矿的原理

type Block struct {
	Index int
	Data string //
	PreHash string
	Hash string
	Timestamp string
	//记录挖矿节点
	Validator *Node

}

func genesisBlock() Block  {

	var genesBlock  = Block{0, "Genesis block","","",time.Now().String(),&Node{0, 0, "dd"}}
	genesBlock.Hash = hex.EncodeToString(BlockHash(&genesBlock))
	return genesBlock
}

func BlockHash(block *Block) []byte  {
	record := strconv.Itoa(block.Index) + block.Data + block.PreHash + block.Timestamp + block.Validator.Address
	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)

	return hashed
}

//创建全节点类型
type Node struct {
	Tokens int //持币数量
	Days int //持币时间
	Address string //地址
}


//创建5个节点
//算法的实现要满足 持币越多的节点越容易出块
var nodes = make([]Node, 5)
//存放节点的地址
var addr = make([]*Node, 15)


func InitNodes()  {

	nodes[0] = Node{1, 1, "0x12341"}
	nodes[1] = Node{2, 1, "0x12342"}
	nodes[2] = Node{3, 1, "0x12343"}
	nodes[3] = Node{4, 1, "0x12344"}
	nodes[4] = Node{5, 1, "0x12345"}

	cnt :=0
	for i:=0;i<5;i++ {
		for j:=0;j<nodes[i].Tokens * nodes[i].Days;j++{
			addr[cnt] = &nodes[i]
			cnt++
		}
	}

}

//采用Pos共识算法进行挖矿
func CreateNewBlock(lastBlock *Block, data string) Block{

	var newBlock Block
	newBlock.Index = lastBlock.Index + 1
	newBlock.Timestamp = time.Now().String()
	newBlock.PreHash = lastBlock.Hash
	newBlock.Data = data


	//通过pos计算由那个村民挖矿
	//设置随机种子
	rand.Seed(time.Now().Unix())
	//[0,15)产生0-15的随机值
	var rd =rand.Intn(15)

	//选出挖矿的旷工
	node := addr[rd]
	//设置当前区块挖矿地址为旷工
	newBlock.Validator = node
	//简单模拟 挖矿所得奖励
	node.Tokens += 1
	newBlock.Hash = hex.EncodeToString(BlockHash(&newBlock))
	return newBlock
}

func main()  {

	InitNodes()

	//创建创世区块
	var genesisBlock = genesisBlock()

	//创建新区快
	var newBlock = CreateNewBlock(&genesisBlock, "new block")

	//打印新区快信息
	fmt.Println(newBlock)
	fmt.Println(newBlock.Validator.Address)
	fmt.Println(newBlock.Validator.Tokens)

}

```
