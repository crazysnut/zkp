本文作为上篇的进阶内容，讲详细介绍下KZG Commitment（kate commitment，[Kate-Zaverucha-Goldberg](https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf)）。

给定一个阶为$p$的循环群 $\mathbb{G}$ 如下，$G$为生成元：
$$
\mathbb{G} = \{0, G, 2G, 3G, \ldots, (p-1)G\}
$$

*信任设置*
- 选择随机值$\tau \in \mathbb{F}_p$
- 计算$\forall i \in [d] : H_i = \tau^iG$
$$
gp = \left(H_0=G, H_1=\tau G, H_2=\tau^2 G, \ldots, H_d=\tau^d G \right) \in \mathbb{G}^{d+1}
$$
- 删除$\tau$

 *承诺阶段*
$$com_f=f(\tau)G$$
假设$f(X) =f_0+f_1X+f_2X^2+...+f_dX^d$
$$
\begin{align*}
f(\tau)G &=f_0G+f_1\tau G+f_2\tau^2G+...+f_d\tau^dG \\
         &=f_0H+f_1H_1+f_2H_2+...+f_dH_d
\end{align*}
$$

**单点证明**：
此时证明者有$(gp, f, u, v)$，验证者有$(gp, com_f, u, v)$，目标是要验证$f(u)=v$
$$
\begin{align*}
f(u)=v &\Leftrightarrow \hat{f} := f - v \\
       &\Leftrightarrow  (X-u)\  divides\  \hat{f} \\
       &\Leftrightarrow \exists q \in \mathbb{F}_p[X], q(X)(X-u) = \hat{f}(X)=f-v
\end{align*}
$$
将$x=\tau$代入，
$$
\begin{align*}
& q(\tau)(\tau-u)=f(\tau)-v  \\
\Rightarrow &q(\tau)G(\tau-u)=f(\tau)G-vG \\
\Rightarrow &com_q(\tau-u)=com_f-vG
\end{align*}
$$

*打开阶段*
- 证明者发送Proof $\pi=q(\tau)G=com_q$
- 验证者验证： $com_q(\tau-u)=com_f-vG$

注意：这个等式仍然应用到了已经删除的参数$\tau$，为了消除这个参数，可以利用在part1中介绍的双线性配对。
为简便起见，记
$$[x]_1=xG \in \mathbb{G}_1 \ and\ [x]_2=xH \in \mathbb{G}_2$$
对 $q(\tau)(\tau-u)=f(\tau)-v$进行双线性配对操作
$$e(\pi, [\tau-u]_2)=e(com_f-[v]_1, H)$$

为了减少验证者在$\mathbb{G}_2$上的昂贵操作，也可以进行如下变化[plonk-polycom](https://github.com/sec-bit/learning-zkp/blob/develop/plonk-intro-cn/5-plonk-polycom.md)：
$$
\begin{align*}
& q(\tau)\tau=f(\tau)+q(\tau).u-v \\
\Rightarrow &e(\pi,[\tau]_2)=e(com_f+\pi.u-[v]_1,H)
\end{align*}
$$
这个式子计算所用到的参数都在公共参数$gp$中已提供。

**多重证明**（Batch Open）

KZG最大的优势就是可以利用单个群元素验证任意次数的多项式。
目标：有一个包含k个点的列表$(z0,y0),(z1,y1),…,(z_{k−1},y_{k−1})$，证明$f(X)$通过这些点。

利用拉格朗日插值，可以找到一个次数小于$k$的多项式来经过这些点：
$$
I(X)=\sum_{i=0}^{k-1}y_i\prod_{j=0,j\neq i}^{k-1}\frac{X-z_j}{z_i-z_j}
$$
令:$$Z(X)=(X−z_0)⋅(X−z_1)⋯(X−z_{k−1})$$
$$q(X)=\frac{f(X)-I(X)}{Z(X)}$$证明者发送Proof： $\pi=q(\tau)G=com_q$
验证者验证：$e(\pi, [Z(\tau)]_2)=e(com_f-[I(\tau)]_1, H)$

总结下Kate承诺的**特点**：
单点和多点打开都证明大小固定，对于 BLS12_381，均为48bytes

**Kate作为向量承诺**

回复下向量承诺定义：针对矢量$a_0,…,a_{n−1}$的承诺，并允许证明任意位置$i$对应$a_i$。

利用Kate承诺，使$p(X)$为对所有的$i$计算 $p(i)=a_i$的一个多项式，则$p(X)$可以通过拉格朗日插值来计算：
$$
p(X)=\sum_{i=0}^{k-1}a_i\prod_{j=0,j\neq i}^{k-1}\frac{X-z_j}{z_i-z_j}
$$
从而转换成上面的多重证明。

 Part3介绍一种不需要信任设置的commitment schema：IPA
 
参考文献：
1. Dankrad Feist: [KZG polynomial commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html)
2. Alin Tomescu: [Kate-Zaverucha-Goldberg (KZG) Constant-Sized Polynomial Commitments](https://alinush.github.io/2020/05/06/kzg-polynomial-commitments.html)
3. [KZG in Practice: Polynomial Commitment Schemes and Their Usage in Scaling Ethereum](https://scroll.io/blog/kzg)
4. ZKPs over non-aligned fields: [How to use KZG commitments in proofs](https://notes.ethereum.org/@dankrad/kzg_commitments_in_proofs#ZKPs-over-non-aligned-fields)


