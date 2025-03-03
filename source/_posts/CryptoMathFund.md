---
title: CryptoMathFund
date: 2025-02-27 09:40:41
tags:
---
## 群映射:
* 单射:一对一,但可能存在没有对应的项
* 满射:所有项都有对应,但可能出现多对一和一对多
* 双射:一一对应,既单射又满射.

## Kerckhoff原则:
只有密钥保密,加密算法本身应当向所有人公开.

## 齐夫定律Zipf's law
最简单的齐夫定律的例子是“1/f function”。给出一组齐夫分布的频率，按照从最常见到非常见排列，第二常见的频率是最常见频率的出现次数的½，第三常见的频率是最常见的频率的1/3，第n常见的频率是最常见频率出现次数的1/n。
在布朗语料库中，“the”、“of”、“and”是出现频率最前的三个单词，其出现的频数分别为69971次、36411次、28852次，大约占整个语料库100万个单词中的7%、3.6%、2.9%，其比例约为6：3：2。大约占整个语料库的7%（100万单词中出现69971次）。满足齐夫定律中的描述。

## 生日界限
* $ 2^n$大小随机空间,当不同的尝试过$2^{n/2}$种可能时,发生碰撞的概率就达到50% 
* 密码算法的安全级别不能低于$2^{128}$,即至少要经过约$2^{128}$次尝试后才能破解.

## 循环群
可由一个生成元生成群内所有元素.素数p可以构成p-1阶循环群(原根定理).
* 如一个非素数8,他生成的群的元素只有{1,3,5,7}(根据gcd=1),其子群的阶数最多为2(如以3为生成元{1,3}),无法生成所有群内元素,不是循环群.

* **注意不要把循环群和群,子群搞混.**

## 原根定理:
素数p形成的模p乘法群中,至少存在一个生成元a能形成p-1阶循环子群.注意,不是所有小于p的生成元都能生成p-1阶.

## 拉格朗日定理
有限群G,子群H,G的阶数一定能被H的阶数整除,即都为G的因子(普通因子,而非质因子).

## 循环群子群定理
在拉格朗日定理基础上,有限循环群P,阶数为p-1,生成元为a.以**p-1**的因子**d**作为阶数形成的子群**H**的生成元为:$$a^{(p-1)/d} mod{p}$$

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
且 $ m_1, m_2, ..., m_n $ 互质，则可以唯一确定 $ x $。
* 其结果x的公式比较罗嗦,求解时直接设倍数为参数使用同余消去律带入即可.

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
