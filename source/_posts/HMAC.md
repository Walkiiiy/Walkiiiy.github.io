---
title: HMAC
date: 2025-03-02 17:09:56
tags:
---
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