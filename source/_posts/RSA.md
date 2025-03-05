---
title: RSA
date: 2025-02-27 15:15:33
tags:
---
## 数论基础
* 离散对数问题的难解性和模运算的不可逆性
* 模反元素:a,m互质时,有x使a*xmodm=1(x可由扩展欧几里德算法或 费马小定理得出)
* 大数质分解困难性
* 欧拉定理和欧拉函数,以及欧拉函数积性定理
* 中国剩余定理
## 步骤:
* ϕ(pq)=ϕ(p)*ϕ(q),用大质数p,q得ϕ(n)=ϕ(pq)=(p-1)*(q-1)
* 选取小于n且与n互质的e
* 计算e关于ϕ(n)的模反d
* 公钥:(e,n);私钥(d,n)
* 加解密式:(m^e mod n)^d mod n =m^ed mod n = m
## 结果证明:
* 由模反,ed mod ϕ(n)=1,即ed=k*ϕ(n)+1
* m^ed mod n = m^(k*ϕ(n)+1) mod n=(m modn )*(m^k*ϕ(n) modn) modn  (乘法模运算)
* m与n互质时,根据欧拉定理有m^k*ϕ(n) mod n=1(还要用到同余指数化)
* **m与n不互质时,也成立**(此时m一定为p或者q的倍数.详细证明见附1)
* 因此原式=m mod n=m
## 破解难点:
已知n是两个质数pq之积,pq足够大时无法由n推得pq,也就无法算出ϕ(n),也就无法由e和ϕ(n)算出d
## 应用:
* RSA PKCS#1v1.5签名和RSA-PSS签名
* RSA密钥交换(公钥加密对称密钥后发送,私钥解密)
## 附:
### **RSA 解密时 $ M $ 是 $ p $ 的倍数的推导总结**
---

#### **问题**
已知：
- RSA 加密：$ C = M^e \mod n $
- RSA 解密：$ M' = C^d \mod n $
- $ M $ 是 $ p $ 的倍数，即 $ M = k p $
- 目标：证明解密后 $ M' = M $

---

#### **第一步：计算 $ M' \mod p $**
1. 由于 $ M = k p $，加密后：
   $$
   C = (k p)^e = k^e p^e \mod n
   $$
2. 取模 $ p $：
   $$
   C \equiv 0 \pmod{p}
   $$
3. 解密：
   $$
   M' = C^d \equiv 0^d \equiv 0 \pmod{p}
   $$
4. 结论：$ M' $ 仍然是 $ p $ 的倍数。

---

#### **第二步：计算 $ M' \mod q $**
1. 取模 $ q $：
   $$
   C \equiv k^e p^e \pmod{q}
   $$
2. 由于 $ p^e $ 指数较大，利用 **费马小定理** 化简：
   $$
   p^{q-1} \equiv 1 \pmod{q}
   $$
   使得：
   $$
   p^e \equiv p^r \pmod{q}, \quad 其中 \quad r = e \mod (q-1)
   $$
3. 进一步解密：
   $$
   M' = C^d \equiv (k^e p^r)^d \equiv k^{e d} p^{r d} \pmod{q}
   $$
4. 由于 $ e d \equiv 1 \pmod{\varphi(n)} $，所以：
   $$
   p^{e d} \equiv p \pmod{q}
   $$
   以及：
   $$
   k^{e d} \equiv k \pmod{q}
   $$
5. 因此：
   $$
   M' \equiv k p \pmod{q}
   $$

---

#### **第三步：使用中国剩余定理（CRT）合并**
已知：
$$
\begin{cases}
M' \equiv 0 \pmod{p} \\
M' \equiv k p \pmod{q}
\end{cases}
$$
1. 设 $ M' = p x $，带入第二个方程：
   $$
   p x \equiv k p \pmod{q}
   $$
   两边除以 $ p $（因 $ p $ 和 $ q $ 互质）：
   $$
   x \equiv k \pmod{q}
   $$
2. 设通解：
   $$
   x = k + m q
   $$
3. 代回 $ M' = p x $：
   $$
   M' = p (k + m q) = k p + m p q
   $$
   由于 $ m p q $ 是 $ pq $ 的倍数：
   $$
   M' \equiv k p \pmod{pq}
   $$
4. **最终得到：**
   $$
   M' = k p = M
   $$
---
证毕.