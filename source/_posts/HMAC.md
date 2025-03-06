---
title: HMAC,Pring,HKDF
date: 2025-03-02 17:09:56
tags:
---
## HMAC
* 能同时实现收发双方的完整性验证和身份验证.
* 主要基于SHA-2
$$
\text{HMAC}(K, M) = H\Big( (K \oplus opad) \| H( (K \oplus ipad) \| M ) \Big)
$$

其中：
- $ K $：密钥（Key）
- $ M $：消息（Message）
- $ H $：哈希函数（如 SHA-256）
- $ opad $：外部填充（Outer Padding），固定值 `0x5c5c...5c`
- $ ipad $：内部填充（Inner Padding），固定值 `0x3636...36`
- $ \oplus $：按位异或（XOR）
- $ \| $：字符串连接（Concatenation）

**这样设计主要是为了防止SHA2的长度扩展攻击**
**除此之外还有基于SHA3的KMAC**

## 伪随机数生成器Pring
伪随机数生成器与真随机数生成器Tring相对应,需要种子来形成均匀的随机数序列,相同的种子产生相同的序列.
* 线性同余生成器
* BBS生成器
* 分组加密生成器

## HKDF密钥派生函数
基于HMAC,可以指定输出长度.
### 步骤:
### **（1）HKDF-Extract（提取阶段）**
**作用**：从输入密钥材料（IKM）中提取一个固定长度的伪随机密钥（PRK），用于后续扩展。

#### **输入：**
- **IKM（输入密钥材料，Input Keying Material）**：
  - 例如，TLS 1.3 中的 **ECDHE 共享密钥**，或其他原始密钥输入。
- **Salt（可选的盐值）**：
  - 如果可用，推荐使用一个随机或固定的非零值（如 TLS 1.3 使用 `HelloRetryRequest` 产生的值）。
  - 如果 `Salt` 为空，则默认用全零填充。

#### **输出：**
- **PRK（伪随机密钥，Pseudo-Random Key）**
  - 用于 HKDF-Expand 过程。

#### **计算公式：**
$$
\text{PRK} = \text{HMAC}(\text{Salt}, \text{IKM})
$$

---

### **（2）HKDF-Expand（扩展阶段）**
**作用**：从 PRK 生成目标密钥（OKM），确保输出的密钥长度符合需求。

#### **输入：**
- **PRK（伪随机密钥）**：
  - 由 HKDF-Extract 计算得出。
- **Info（上下文信息，Context Information，可选）**：
  - 附加数据，用于区分不同用途的密钥派生（如 TLS 1.3 中的 `server finished`）。
  - 例如，在 TLS 1.3 中，它可能包括 `"tls13 server finished"` 这样的标识符。
- **L（目标输出密钥长度）**：
  - 目标密钥的长度，通常依赖于应用需求（例如 32 字节或 48 字节）。

#### **输出：**
- **OKM（输出密钥，Output Keying Material）**
  - 长度为 L 的最终密钥，可用于加密、身份验证等用途。

#### **计算公式：**
$$
T_1 = \text{HMAC}(\text{PRK}, \text{Info} || 0x01)
$$
$$
T_2 = \text{HMAC}(\text{PRK}, T_1 || \text{Info} || 0x02)
$$
$$
T_n = \text{HMAC}(\text{PRK}, T_{n-1} || \text{Info} || n)
$$
$$
\text{OKM} = T_1 || T_2 || ... || T_n
$$
其中，`n = ceil(L / hash_output_size)`，即输出密钥长度 L 由多少个哈希块组成。
