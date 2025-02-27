---
title: DH密钥交换-算法浅析
date: 2025-02-20 08:44:24
tags:
---
## 数论基础：
* 离散对数问题难解性
* 循环群
* 模运算的幂运算性质
## 步骤:
* 双方交换a,P;Alice生成X1 < p,Bob生成X2 < p.
* Y1=a的X1次幂mod p
* Alice向Bob发送Y1
* Bob将Y1的X2次幂mod p后得到key2
* (**同时**)Y2=a的X2次幂mod p
* Bob想Alice发送Y2
* Alice将Y2的X1次幂modp后得到key1

由模幂运算性质:**结果都等于a的X1*X2次幂,即Key1=key2.**