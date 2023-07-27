---
title: 字节与字符
date: 2023-07-27 08:14:06
tags: 计算机基础知识
---
字节即 Byte，1 Byte = 8 bits;
字符即字符集中的一员，字符集有很多，比如: ASCII, UTF8，base64, base58, base32, base16(hex) 等等。

字符与字节的转换公式：
字符 <---> Bit <----> 字节

下面我对常用字符ASCII, UTF8, base64, base58, base16 与 字节 转换:
1. ASCII. ASCII字符集共127个字符，理论上用 7 个 bits就可以，但是ASCII依旧用 8 个 bits 表示一个字符，所以 ASCII 字符与字节是 1 对 1 转换

