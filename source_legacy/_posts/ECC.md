---
title: ECC
date: 2025-03-05 16:06:27
tags:
---
## 原理
* 椭圆曲线离散对数问题:给定基点P,k为私钥,Q为公钥点,等式$$ Q=kP$$根据加法法则,由kP算Q很容易,由P,Q算k无法实现(质数P非常大时).
* 椭圆曲线:满足$$y^2=x^3+ax+b$$且$4a^3+27b^2\neq0$.与计算椭圆周长的公式类似.
![ECCCurve](/images/ECC.png)
* 在密码学中,该公式被改写为$$y^2 \equiv x^3+ax+b mod p$$形成有限域点集(点坐标结果超过p时取modp余数).其所有加运算公式即是在上述运算公式的基础上模p.
![limitedECC](/images/limitedECC.png)
上图中G为生成元.
* 椭圆曲线加法:A+B即为连接AB两点找到该线与椭圆曲线的交点关于X轴对称点.A+A即为找到A切线与曲线交点关于X轴对称的点.nA即为重复该过程n次.
![ECCPlus](/images/ECCPluse.png)
## ECC公钥加密
如果ECC公钥加密,需要将明文转化为一个点P
* 生成元G
* Alice公钥Q=dG,私钥d
#### **过程**
* Bob向Alice发送加密后的P,选取一个随机数k,计算kQ,kG向Alice发送(P+kQ)和(kG)
* Alice计算(P+kQ)-dkG得到p
#### **加解密范式**
  $$
  P_M = P_M+k(dG) - d(kG)
  $$
整体类似DH交换

## ECDH
**DH推荐2048bit参数,而ECDH只要256bit**
#### 步骤
* 生成元G,Alice和Bob两者的私钥为a,b
* Alice向Bob发送aG
* Bob向Alice发送bG
两者可以得到共享密钥abG.
