+++
title = "Crystal-Kyber中的多项式运算"
author = ["lkcp"]
date = 2023-09-20T14:32:00+08:00
draft = true
+++

**Kyber** (全称为 CRYSTALS-Kyber 前缀 CRYSTALS 是指 Kyber 和 Dilithium 两个算法构成的 Cryptographic Suite for Algebraic Lattices) 算法是一种基于 MLWE 的密钥封装机制。2022年7月， NIST 宣布 Kyber 作为 PQC(Post-Quantum Cryptography) 竞赛第三轮的胜选算法将成为首批被标准化的后量子密码算法。因此，在可预见的未来，学习 Kyber 会像过去这些年学习 ECC 和 RSA 一样重要。Kyber 中的核心运算是有限域上的多项式算术，关键是基于 NTT ([Number-Theory Transofrm](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjN5fOoyriBAxV5h1YBHXiYAs0QFnoECB4QAQ&url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FDiscrete_Fourier_transform_over_a_ring&usg=AOvVaw1y5H8Od5Og_Kj-Qt5Pfpw6&opi=89978449)) 的多项式乘法。NTT 脱胎于经典的算法 FFT(快速傅里叶变换)，普遍应用于诸多基于结构格（尤其是基于 RLWE 困难问题）的密码算法中。但是，Kyber 中所使用的 NTT 相关操作是一种高度定制化的版本，并且其原始文档中对该部分的介绍对非专家读者极其不友好。因此，在刚开始接触 Kyber 算法时，会非常容易迷惑：为什么这里的 NTT 和我在别处看到的 NTT 非常不同？为什么参考实现中给出的原根序列不是17的幂？本文旨在解答在已经对 Kyber 和 NTT 算法有了初步的了解后，对 Kyber 算法中多项式运算仍然可能存在的疑惑。基础知识建议参考 Amber Sprenkels 的博客文章 [The Number Theoretic Transform in Kyber and Dilithium](https://electricdusk.com/ntt.html)


## 256 阶原根而非 512 阶原根意味着什么 {#256-阶原根而非-512-阶原根意味着什么}

1.  7 层的 NTT 结构而非 8 层，可以理解为两个 127 阶的多项式分别执行普通的 NTT
2.  NTT 的结果是 128 个1阶多项式，因此 NTT 域的多项式运算是 pair-wise 而非 point-wise
3.  pai-wise 乘法是 \\(\mod (X^2 - \epsilon^{2br\\(i\\)+1}\\) 的，也就是 \\(X^{2} = \epsilon^{2br\\(i\\)+1}\\)
4.

在 Kyber 的初始论文 <&bosCRYSTALSKyberCCASecure2018> 中，模数 \\(q=7681\\) ，这个模数存在一个 512(\\(2n\\)) 阶的原根 (root) 62，能够支持 Kyber 进行常规的 NTT 运算。在 NIST 的 PQC 竞赛的第二轮提交的文档中， Kyber 团队指出，为了增加带宽，降低公钥的尺寸，他们需要使用一个更小的模数。而最新的研究表明 (<https://eprint.iacr.org/2019/040>)，只需要 \\(q\\) 存在 \\(n\\) 阶原根同样可以实现 NTT，并且能够达到相同甚至更好的性能。因此 Kyber 改为使用一个更小的模数 \\(q=3329\\) ，这一模数存在一个256阶的原根17。

在这样的条件下，NTT 只能将多项式分解为 128 个二阶的多项式，而不再是原本的情况下将多项式分解为256个点值（一阶多项式）。那也就是说不再像一般的 NTT 域上的多项式乘法一样直接进行 point-wise-multiplication，而是需要对这些一阶多项式两两相乘，由于是一对系数乘以一对系数，所以一般称为 pair-wise-multiplication。相应地，


## 参考实现中进行了哪些优化 {#参考实现中进行了哪些优化}

在参考实现中，
使用的是 \\(\mod^{+-\frac{q}{2}}\\) 的模运算


### 使用蒙哥马利约减加速乘法 {#使用蒙哥马利约减加速乘法}

选用蒙哥马利域上的 zeta 值，选取的参数为 \\(R = 2^{16}\\)，首先通过 \\(zeta\\\_mont = \text{REDC}((zeta \mod q)(R^2 \mod q))\\) 将 zeta 转换到蒙哥马利域上。
