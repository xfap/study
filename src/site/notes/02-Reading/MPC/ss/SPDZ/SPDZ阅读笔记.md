---
{"title":"阅读笔记","creation date":"2022-03-02 13:51:59","parent":null,"rating":"⭐","dg-home":true,"dg-publish":true,"permalink":"/02-reading/mpc/ss/spdz/spdz/","tags":"gardenEntry","dgHomeLink":true,"dgPassFrontmatter":true}
---

tags: 

# Notes

[Teambition](https://www.teambition.com/project/5f977eb99585343c1f1e4d89/app/5eba5fba6a92214d420a3219/workspaces/5f977eb957fbb20016744947/docs/6103e25b4e168300014d697f)

# 秘密共享 MPC支柱之一

# What is SPDZ(speedz)? Part 1

https://bristolcrypto.blogspot.com/2016/10/what-is-spdz-part-1-mpc-circuit.html

两种MPC的主要构造之一，SS！和 Garble Circuit

SS不同于GC（Yao首次提出的MPC实现之一），SPDZ是SS-based的MPC，但还有许多其他的实现（例如TinyOT等可以视为OT版本的SPDZ）。

不同：

-   GC里面是通过特定的顺序去执行电路计算，其中会引入加解密秘钥。（电路刚开始是Boolean电路，现在也有Arithmetic(算数)电路）
    
-   不同于此，SSMP通过在所有参与方进行“秘密分享”输入，并且用这些分享share去计算电路。
    

什么是秘密分享 SS？

假设有元素a，我们“随机”的分享a为pieces，各方P_i拥有对应的部分a_i.(e.g., a=a_1+a_2).没有任何一方知道a是多少，但是他们一起可以恢复。

我们用<a>去表示，a是被分享。

上面是Additive SS，当然还有其它的共享形式，例如乘法Multiplication，但是加法SS对于MPC极其有用。

SSMPCoAC SSMPC 基于Arithmetic Circuit，一个流程，看截图。

## Circuit Garbling vs Secret Sharing

There are **two main constructions of MPC protocols** for circuit evaluation: circuit garbling and secret sharing.

**The answer to the so-called millionaire’s problem** was first found in the 1980s with Yao’s **garbled circuits** [10]. As circuit garbling is somewhat parallel to the MPC model we work with in SPDZ, we will not discuss it here.

Contrasting this, the SPDZ protocol is a **secret-sharing-based MPC protocol**.

## Secret-Sharing-Based MPC

Whereas **circuit garbling involves encrypting and decrypting keys** in a specific order to emulate a circuit evaluation (originally a Boolean circuit, but now arithmetic circuits too [1]), SPDZ instead ‘secret shares’ inputs amongst all parties and uses these shares to evaluate a circuit.

SPDZ is neither the first nor the only secret-sharing-based MPC protocol. Other well known constructions include BDOZ [3], TinyOT [8] and MiniMAC [6]. MASCOT [7] can be seen as an oblivious-transfer-based version of SPDZ. This will be discussed in a little more detail later on.

## What is secret sharing?

Suppose I have some field element a∈F, split it up ‘at random’ (uniformly) into two pieces, **a=a1+a2, and give party P1 the value a1 and P2 the value a2.** Neither party knows the value a, but together they can recover it. We will write **⟨a⟩** to mean that the value a is **secret-shared** between **all parties (i.e. for each i, party P_i has a_i, where ∑_i a_i=a)**.

Of course, there **are different ways of secret sharing data** (e.g. the analogous multiplicative sharing a=a1⋅a2, and also more complicated schemes like Shamir’s [9]), but it turns out that the additive scheme is particularly useful for MPC applications, as we shall see.

The basic overview of secret-sharing MPC of arithmetic circuits (SSMPCoAC?) is the following:

![](https://tcs.teambition.net/storage/3128dfbdb2f97127578510146a1d697e7241?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjhkZmJkYjJmOTcxMjc1Nzg1MTAxNDZhMWQ2OTdlNzI0MSJ9.aSzRN4PP5NaZxmjSAtOxJ3ctMavjAXtOrvLt5v7g9jc)

These are the steps we follow in a few different MPC circuit evaluation protocols, as we have discussed. The way we compute the circuit differs (slightly) with the protocol.

# What is SPDZ? Part 2: Circuit Evaluation

https://bristolcrypto.blogspot.com/2016/10/what-is-spdz-part-2-circuit-evaluation.html

介绍在SPDZ中是如何对分享的秘密值进行加法和乘法。

**首先是介绍预处理模型**：假设一个**信任dealer的存在，其在电路计算前，分发raw material给到不同的party**。这是因为如果这些raw material已经存在，电路计算可以十分高效。

我们通过将协议分为预处理（离线）和在线阶段：计算电路。

**此后是电路计算**，首先假设parties已经协商共同计算某些算术电路。

第一步是，提供输入。计算电路从parties读取输入。在SPDZ，是让各个party输入，P_i输入x^i，并且将输入x^i SS给其它parties。

数据SS

**（加性）****秘密分享**：我们首先需要用一个**random value的分享（）**，

①**r的合成**。每个**Pi拥有r_i**，各方的随机数加和，value r=\sum{i}r_i 是均匀随机的，**其不被任何一方知晓**。然后每一个Party将他们r_j发送给P_i（这称作 部分打开），**此时P_i可以发现r的值**，通过加上这些接收到的shares和自己的r_i。

②此时Pi可以广播自己的值，**x_i-r，****r对于其他人都是未知的****。这是为了实现SS的关键。**

③然后，我们让P_1(也不一定总是P1需要计算不同的),设置它的数据x^i的SS为，x^i_1=r_1+**x^i-r**(**x^i-r就是上面公布的**)，其它j>1,设置为 **x^i_j=r_j（就是设计为自己的随机数，也就是除了第一步的发送随机数，不需要再做其它交互）**

**可以发现 x^i_1+\sum{j>1}{x^i_j}=x^i**

借此，我们实现了x^i的共享。

另外一种加法共享实现

https://zhuanlan.zhihu.com/p/44999983

![](https://tcs.teambition.net/storage/31283e43fecb5e551cca0932a37e5048fcef?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgzZTQzZmVjYjVlNTUxY2NhMDkzMmEzN2U1MDQ4ZmNlZiJ9.cuXY-ojZp_S-B7I6U3ucj15GnJzWsfjkpo5FVHCMuYo)

![](https://tcs.teambition.net/storage/31283d02e3c8d185d833120d8af9c8e74fcd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgzZDAyZTNjOGQxODVkODMzMTIwZDhhZjljOGU3NGZjZCJ9.dw2zCeFRP7Ez6pw7WbYEMFqWr2T0_ybRBWnPhdqW6rk)

**借助SS实现安全运算**

**加法**

举例，如果想要计算a+b

那么各方加性秘密共享后，<a>和<b>实现了共享，也就是各方拥有了自己的a_i,b_i，

此时如果Parties共识计算a+b，那么只需要自己加上a_i+b_i(对于不是1方的人，就是把自己两个随机数加起来，或者乘以一个数)，因为\sum a_i +\sum_i b_i =\sum{a_i+b_i}, 也就获得了<a+b>的分享。

**乘法**

想计算xy，<x>,<y>

**需额外的共享，a，b，c（称为triples），得到<a>,<b>,<c>, c=ab.（offline，其他时候，**三元组的可以由可信第三方生成或者是两方执行OT。**）**

Pi各自广播x_i-a_i，y_i-b_i

此时各方可以计算，x-a，y-b，这些值也就是公开

此时，计算z_i=c_i+(x-a)b_i+(y-b)a_i，这个z_i也就是xy的分享？

然后其中一个Party(随机选择),把他的share加上公开数值(x-a)(y-b)

此时，加上各方的分享，\sum{z_i}=c+(x-a)b+(y-b)a+(x-a)(y-b)=xy

如此，如果我们有更多的triples，就可以在SS值上计算多次乘法， 也就可以计算任何算术电路。

**My concern：SS的明显达不到的，无法保护算法（电路）。**

abc是不能复用的，因为x-a,y-b是公开的，如果复用，就可以建立方程组，去恢复

**更重要的，abc是**和xy是独立的，也就是它的share可以在offline阶段进行计算。

**Secure Vectorization**

![](https://tcs.teambition.net/storage/312829e871d34cabc6a67f777d63bcfab3b5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgyOWU4NzFkMzRjYWJjNmE2N2Y3NzdkNjNiY2ZhYjNiNSJ9.Ti7wZHgrAfSNTumQbyVPy97_11u16_F3SH-haIgcRZQ)

想要恢复数据或进行计算的时候，一方将自己的数据发给另一方，或者将数据一起发给第三方（具体根据隐私需求来定）。

---

show how we **perform the additions and multiplications** of **shared secret values** as in the SPDZ protocol.

## The Preprocessing Model

**The preprocessing model** is a framework in which we assume the existence of a 'trusted dealer' who distributes 'raw material' to the parties before any circuit evaluation takes place. The reason for doing this is that if we have the 'raw material' already in place, the circuit can be evaluated very efficiently.

We realise this by splitting the protocol into two phases: a 'preprocessing' (or 'offline') phase where we generate the preprocessed data, and an 'online' phase where we evaluate the circuit. (The term 'offline' is a little misleading, since the parties still communicate.)

![](https://tcs.teambition.net/storage/3128b615f517b10e1c9a82c08cb785cde4ec?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjhiNjE1ZjUxN2IxMGUxYzlhODJjMDhjYjc4NWNkZTRlYyJ9.PWRuKKZ9EkBFbRk4BGEB8Vd1lBk1Ky_K1OhUTKHN5UQ)

![](https://tcs.teambition.net/storage/31286698ce1c6710d83ff86f9427c802882c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjg2Njk4Y2UxYzY3MTBkODNmZjg2Zjk0MjdjODAyODgyYyJ9.a3eEUdXD23-zAwv85QJoTZSFdgOgllI5SUE6P16odPI)

![](https://tcs.teambition.net/storage/312876672858bd6cecbaaa806119ef721838?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjg3NjY3Mjg1OGJkNmNlY2JhYWE4MDYxMTllZjcyMTgzOCJ9.pcTUeURTH16o7p2xstJD0fKzg_9qZ3F0t3FMfH-J8qk)

![](https://tcs.teambition.net/storage/31284f7c46a6a9e1cc282913ac494599b594?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjg0ZjdjNDZhNmE5ZTFjYzI4MjkxM2FjNDk0NTk5YjU5NCJ9.muX7U46-8na8a00c_71Qtdwy9oYC3HbUrCjz6NngRdE)

# What is SPDZ? Part 3: SPDZ specifics

https://bristolcrypto.blogspot.com/2016/11/what-is-spdz-part-3-spdz-specifics.html

最初的SPDZ，是使用SHE去做预处理，分享很多的triples（这个triple是由某个信任第三方产生，或者执行OT，或者FHE？），这些加密后的秘密值，拥有加法和有限次乘法。

SPDZ是FHE在MPC中的理想化应用。

![](https://tcs.teambition.net/storage/31281e40d2ad09dc029c34032e4c514f348c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgxZTQwZDJhZDA5ZGMwMjljMzQwMzJlNGM1MTRmMzQ4YyJ9.k5lUaEf2ujA7cHNRCzFa2aPUaryORiANFFpFfl4di_o)

牺牲一个triple去做验证，保证triple是没有被引入error的

![](https://tcs.teambition.net/storage/3128402a4fa14606a4328ccf088e4ff2a14c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjg0MDJhNGZhMTQ2MDZhNDMyOGNjZjA4OGU0ZmYyYTE0YyJ9.51NYw9rZ1zuTd5pF2U7UuvCRfkWnvh01sB5ossVHKIA)

第二代SPDZ，使用前置MAC认证，和生成shared SHE解密秘钥去构成一个安全的协议。

![](https://tcs.teambition.net/storage/3128b078c675c2fee06fbbbd5fa7b8cf3740?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjhiMDc4YzY3NWMyZmVlMDZmYmJiZDVmYTdiOGNmMzc0MCJ9.Of1CwrI9pQvP1nsKMiK5lDZEJdb_YVJLl_buF74DqvE)

第三代 使用OT去做预处理，达到triples的共享。

![](https://tcs.teambition.net/storage/312834f977d24a38637582946c5e76f62e10?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY0NjgwNTA5MCwiaWF0IjoxNjQ2MjAwMjkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgzNGY5NzdkMjRhMzg2Mzc1ODI5NDZjNWU3NmY2MmUxMCJ9.cZwxUi6O_bEL7JDDoiimApTivCEBabzYI4kNiXZbf6o)