---
title: MathFund
date: 2025-02-27 09:40:41
tags:
---
## 生日界限
* $ 2^n$大小随机空间,当不同的尝试过$2^{n/2}$种可能时,发生碰撞的概率就达到50% 
* 密码算法的安全级别不能低于$2^{128}$,即至少要经过约$2^{128}$次尝试后才能破解.
## 循环群
素数a,p,满足a < p. X,Y为小于p的整数全集.a^X mod p =Y能构成X到Y无序且一一对应（双射）关系.a为生成元,该群的阶数为p-1.
## 循环群子群定理
## 拉格朗日定理
## Sylow 定理
## 离散对数问题难解性
在1的条件下,等式$ a^{Xi} mod {p}={Yi} $, p极大时,已知a,p,y极难求出x,已知a,Xi,p很好验证Yi(快速幂等算法)
## 模运算性质

**(1) 自反性（Reflexivity）**
$$
a \equiv a \pmod{m}
$$
任何整数 $ a $ 对模 $ m $ 都与自己同余。

**(2) 对称性（Symmetry）**
$$
a \equiv b \pmod{m} \quad \Rightarrow \quad b \equiv a \pmod{m}
$$
如果 $ a $ 和 $ b $ 同余，则 $ b $ 和 $ a $ 也同余。

**(3) 传递性（Transitivity）**
$$
a \equiv b \pmod{m}, \quad b \equiv c \pmod{m} \quad \Rightarrow \quad a \equiv c \pmod{m}
$$
如果 $ a \equiv b $ 且 $ b \equiv c $，则 $ a \equiv c $。

---

### **加法、减法和乘法的模运算**
#### **(1) 加法**
$$
(a + b) \mod m = [(a \mod m) + (b \mod m)] \mod m
$$
$$
a \equiv b \pmod{m}, \quad c \equiv d \pmod{m} \quad \Rightarrow \quad (a + c) \equiv (b + d) \pmod{m}
$$

#### **(2) 减法**
$$
(a - b) \mod m = [(a \mod m) - (b \mod m)] \mod m
$$
$$
a \equiv b \pmod{m}, \quad c \equiv d \pmod{m} \quad \Rightarrow \quad (a - c) \equiv (b - d) \pmod{m}
$$

#### **(3) 乘法**
$$
(a \times b) \mod m = [(a \mod m) \times (b \mod m)] \mod m
$$
$$
a \equiv b \pmod{m}, \quad c \equiv d \pmod{m} \quad \Rightarrow \quad (a \times c) \equiv (b \times d) \pmod{m}
$$

---

### **幂运算（指数运算）**
(1)
$$
(a^b) \mod m = [(a \mod m)^b] \mod m
$$
**快速幂算法（Exponentiation by Squaring）**可用于高效计算 $ a^b \mod m $。

如果 $ a, b $ 互质，则有：
$$
a^{\phi(m)} \equiv 1 \pmod{m}
$$
其中 **$ \phi(m) $** 是欧拉函数。


(2)**同余的指数化**
如果 $ a \equiv b \pmod{n} $，则对于任意正整数 $ k $，都有：

$$
a^k \equiv b^k \pmod{n}
$$
---

### **逆元运算（模逆）**
如果 $ a $ 在模 $ m $ 下**可逆**(即a与m互质)，则存在整数 $ x $ 使得：
$$
a \times x \equiv 1 \pmod{m}
$$
**求模逆的方法：**
- **扩展欧几里得算法（Extended Euclidean Algorithm）**
- **费马小定理（仅对素数模有效）**：
  $$
  a^{-1} \equiv a^{p-2} \pmod{p}, \quad (p 为素数)
  $$

---

### **其他性质**
#### **(1) 约简性质**
$$
(a + km) \mod m = a \mod m
$$
对任意整数 $ k $，增加 $ m $ 的整数倍不会影响同余关系。

#### **(2) 乘法消去律**
如果：
$$
ac \equiv bc \pmod{m}
$$
且 $ c $ 与 $ m $ 互质（$ \gcd(c, m) = 1 $），则可以消去 $ c $：
$$
a \equiv b \pmod{m}
$$

#### **(3) 乘法分配律**
$$
(a \times (b + c)) \mod m = [(a \times b) \mod m + (a \times c) \mod m] \mod m
$$

#### **(4) 负数模运算**
$$
(-a) \mod m = (m - (a \mod m)) \mod m
$$

---

## **费马小定理**
若 $ p $ 为素数，且 $ a $ 不是 $ p $ 的倍数，则：
$$
a^{p-1} \equiv 1 \pmod{p}
$$

## **欧拉定理**
若 $ \gcd(a, m) = 1 $，则：
$$
a^{\phi(m)} \equiv 1 \pmod{m}
$$
其中 $ \phi(m) $ 是**欧拉函数**，表示小于 $ m $ 且与 $ m $ 互质的整数个数。

## **中国剩余定理（CRT）**
如果：
$$
x \equiv a_1 \pmod{m_1}, \quad x \equiv a_2 \pmod{m_2}, \quad ... \quad x \equiv a_n \pmod{m_n}
$$
且 $ m_1, m_2, ..., m_n $ 互质，则可以唯一确定 $ x $（模 $ M = m_1 m_2 ... m_n $）。

## **Wilson 定理**
若 $ p $ 为素数，则：
$$
(p - 1)! \equiv -1 \pmod{p}
$$

## **欧拉函数积性定理**
如果 $ p $ 和 $ q $ 是互质的整数（$\gcd(p, q) = 1$$），则欧拉函数满足：
$$
\phi(pq) = \phi(p) \cdot \phi(q)
$$

building............
