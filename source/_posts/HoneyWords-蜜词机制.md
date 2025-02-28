---
title: HoneyWords-蜜词与其生成机制浅析
date: 2025-02-26 09:49:38
tags:
---
## introduce：“Honeywords:Making Password-Cracking Detectable”by  Ari JuelsRonald and L. Rivest
**蜜词机制的首次提出**
###  背景
* 基本的口令保护机制有加盐、蜜罐（honeypot accounts）等，都是为了抵御彩虹表类攻击。
* 加盐：常见的加盐方式为，在对密钥进行一次sha-256哈希后，拼接随机串，再进行一次md5哈希（其次序和哈希算法无关紧要）。配合使用慢哈希函数如PBKDF2，Scrypt，Argon2，能大幅增加彩虹表的构建成本和长度。但是有足够算力的攻击者拿到数据库中的随机串后，仍有可能进行暴力破解。并且并不能提供入侵检测。
* 蜜罐帐号：服务端生成一些非真实用户的蜜罐帐号，一旦检测到该蜜罐帐号的密码被尝试用于登入，就说明哈希表已经被窃取并且暴力破解。这种方法提供了比较保守的入侵检测功能，但无法保证攻击者无法区分真实帐号和蜜罐帐号，且处于被动地位，无法增加攻击的成本和风险。
### 原理
* 系统为每个用户生成一个包含 **1 个真实密码（Sugarword）+ k-1 个蜜词（Honeywords）** 的列表。
* 其中，**仅有一个密码是用户的真实密码**，其索引值存储在一个**独立的安全服务器**（Honeychecker）上，系统存储的是**所有蜜词及真实密码的哈希值**。
* 由于哈希值存储在系统中，而正确密码的索引仅存储在 Honeychecker 服务器中，即使攻击者窃取了系统中的哈希文件，也无法区分真实密码。当攻击者花费比以往高的多的攻击成本生成彩虹表后进行尝试时，只要选取的是蜜词就会触发入侵检测。

1. **蜜词检测（Honeychecker 交互）**  
系统向**独立的 Honeychecker 服务器** 发送查询：
**如果输入的是正确密码**
正常登录，允许访问系统。
**如果输入的是蜜词**
**触发安全告警**（但是否允许登录可以由系统策略决定）。
可以选择：
* **直接拒绝登录**
* **发送静默告警，观察攻击者行为**
* **重定向到蜜罐（Honeypot）环境进行监控**

### 技术细节
* 假设攻击者无法直接获取密码明文（如使用了RSA交换）但可以获取整张在/etc/shadow或数据库中存储的哈希表，以及盐值。并且有暴力破解获取彩虹表的能力。
* 假设存在一个冗余服务器"honeyChecker"，不对外界开放接口，且之在用户登录或者修改密码时对主服务器的"set"和"check"指令进行响应。（分布式安全：即使主服务器暴露，也不会使用户密码完全暴露。）
* 即使heneychecker暴露，也只是将安全性降低到引入蜜词机制之前的水平，也就是攻击者仍需要手动破解真实哈希值。

### 蜜词生成
Ari JuelsRonald 和 L. Rivest提出了最基本的蜜词生成方式，由于要求攻击者即使得到list也无法将真实密钥从蜜词中分离出来，所以密钥的设计原则为"**similar in style to the password p**"。
####  法1：特定部分进行小幅调整Chaffing by tweaking：
如，选取密码后t位，用随机数字替换数字，字母替换字母，特殊符号替换特殊符号。
如：**给定BG+7y45，t = 3 ，k = 4，生成list为BG+7q03, BG+7m55, BG+7y45, BG+7o92。**
* 可以选择的调整方法有，尾部调整，数字调整，特殊字符调整。
* 易于实现，但易被预测且难以用于弱密码。
#### 法2：基于密码模型生成Chaffing-with-a-Password-Model：
利用从大规模密码列表中学习到的模式来生成蜜词，提高蜜词的多样性和不可预测性.
* **基于密码列表（Password List-Based Honeyword Generation）**
直接用一个大密码列表进行混淆
* **PCFG**:
mice3blind 可能具有 W4 | D1 | W5 这种结构：
W4（4个字母的单词，如 mice）
D1（1个数字，如 3）
W5（5个字母的单词，如 blind）
PCFG先学习数据集中的密码P(W4 D1 W5),P(W6 D2 S1)..........然后再根据输入密码mice3blind(结构为 W4-D1-W5)构建句法树,生成list.
```
gold5rings
#生成:
frog7happy
```
* **随机蜜词（Random Honeyword Generation）**
该方法不考虑用户的密码，只是随机生成一组蜜词。
```
sjd8hskq
aH73ks93
93mskdlf
```
* **强密码掺杂法（Tough Nuts Honeywords）**
在list中加入“Tough Nut” 即一个极难破解的蜜词，它的哈希值无法在合理时间内被破解（例如一个 40+ 位的随机密码）。
* **组合方法（Hybrid Honeyword Generation）**
即以一定比例组合以上方式生成的蜜词为list


## problemDiscover:"A Security Analysis of Honeywords"by Ding Wang, Haibo Cheng, Ping Wang, Jeff Yan, Xinyi Huang
**汪定教授通过10个真实世界的密码数据集（共 1.04 亿个密码）来评估蜜词系统的安全性,发现其安全性远不及预期,并提出相应改进方法.**
### 背景:
* 蜜词解决方案的提出
* 即使加盐后的哈希密码表仍易被机器学习技术, GPU还原(**this poses no real obstacle for an attacker to recover them by an overwhelming percentage by using modern machine-learning based cracking algorithms**)而由于哈希操作可以离线进行,慢哈希函数无法造成实质性的阻碍.
* 多数抗离线破解方案都基于对整个系统的改动,实施困难
### 广泛猜测攻击TRAWLING GUESSING ATTACKS
* 针对尾部修改等简单的蜜词生成方案
* 使用Dodonew 数据集，将其拆分为 训练集（8.1M 条密码） 和 测试集（8.1M 条密码）
##### Top-PW（Top Password）攻击:
成功破解率29.29%～32.62%（比预期的 5% 高出 6 倍以上）
* 从已泄漏的密码数据库中选取最常见的密码,哈希后比对并根据其常见程度进行排名
* 若能针对每个用户分别构建排名,并归一化,则成功率还能上升:
```
NormPr(swi,1) = Pr(swi,1) / ΣPr(swi,j) (j=1~20)
```
### 高级广泛猜测攻击Advanced Trawling Guessing Attacks
building.............