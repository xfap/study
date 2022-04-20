---
{"title":"What is SPDZ？","creation date":"2022-03-02 13:42:02","parent":null,"rating":"⭐","dg-home":false,"dg-publish":true,"permalink":"/02-reading/mpc/ss/spdz/what-is-spdz/","dgHomeLink":true,"dgPassFrontmatter":true}
---

tags: 

# Notes

---
created: 2022-03-02T13:42:22 (UTC +08:00)
tags: []
source: https://bristolcrypto.blogspot.com/2016/10/what-is-spdz-part-1-mpc-circuit.html
author: 
---

# Bristol Cryptography Blog: What is SPDZ? Part 1: MPC Circuit Evaluation Overview

> ## Excerpt
> This blog post is the first in a  series of three in which we look at  what MPC circuit evaluation is, an outline of how MPC protocols in t...

---
This blog post is the first in a series of three in which we look at what MPC circuit evaluation is, an outline of how MPC protocols in the so-called 'preprocessing model' work, and finally the specifics of SPDZ. They will come in weekly instalments.

In this part, we will introduce the idea of MPC circuit evaluation.

### Introduction

If you do research in the field of cryptography, at some point you’ve quite possibly come across the curiously named SPDZ ('speedz'). The aim of this blog post is to explain what it is and why it’s used. In order to keep this post as short and accessible as possible, lots of the details are omitted, and where new concepts are introduced, they are kept superficial.

We start by defining secure multi-party computation (MPC): MPC is a way by which multiple parties can compute some function of their combined secret input without any party revealing anything more to the other parties about their input other than what can be learnt from the output.

Let’s make this more concrete: suppose there are two millionaires who want to know which of them has more money without revealing exactly how much money they have. How can they do this? Clearly we can do it with MPC, providing it exists.

Thankfully, MPC does exist. It is used in many different contexts and has various applications, ranging from the 'simple' and specific such as oblivious transfer (more on this later), to the relatively general-purpose functionality of joint circuit computation.  SDPZ is an MPC protocol allowing joint computation of arithmetic circuits.

#### Circuit Garbling vs Secret Sharing

There are two main constructions of MPC protocols for circuit evaluation: circuit garbling and secret sharing.

The answer to the so-called millionaire’s problem was first found in the 1980s with Yao’s garbled circuits \[10\]. As circuit garbling is somewhat parallel to the MPC model we work with in SPDZ, we will not discuss it here.

Contrasting this, the SPDZ protocol is a secret-sharing-based MPC protocol.

### Secret-Sharing-Based MPC

Whereas circuit garbling involves encrypting and decrypting keys in a specific order to emulate a circuit evaluation (originally a Boolean circuit, but now arithmetic circuits too \[1\]), SPDZ instead ‘secret shares’ inputs amongst all parties and uses these shares to evaluate a circuit.

SPDZ is neither the first nor the only secret-sharing-based MPC protocol. Other well known constructions include BDOZ \[3\], TinyOT \[8\] and MiniMAC \[6\]. MASCOT \[7\] can be seen as an oblivious-transfer-based version of SPDZ. This will be discussed in a little more detail later on.

#### What is secret sharing?

Suppose I have some field element $a \in \mathbb{F}$, split it up ‘at random’ (uniformly) into two pieces, $a = a_1 + a_2$, and give party $P_1$ the value $a_1$ and $P_2$ the value $a_2$. Neither party knows the value $a$, but together they can recover it. We will write $\langle a \rangle$ to mean that the value $a$ is secret-shared between all parties (i.e. for each i, party $P_i$ has $a_i$, where $\sum_i a_i = a$).

Of course, there are different ways of secret sharing data (e.g. the analogous multiplicative sharing $a = a_1 \cdot a_2$, and also more complicated schemes like Shamir’s \[9\]), but it turns out that the additive scheme is particularly useful for MPC applications, as we shall see.

The basic overview of secret-sharing MPC of arithmetic circuits (SSMPCoAC?) is the following: 

1.  The parties first secret-share their inputs; i.e. input $x^i$ is shared so that $\sum_j x_j^i = x^i$ and party $P_j$ holds $x_j^i$ (and $P_i$ which provides input is included in this sharing, even though it knows the sum).
2.  The parties perform additions and multiplications on these secret values by local computations and communication of certain values (in methods specified below). By construction, the result of performing an operation is automatically shared amongst the parties (i.e. with no further communication or computation).
3.  Finally, the parties 'open' the result of the circuit evaluation. This last step involves each party sending their 'final' share to every other party (and also performing a check that no errors were introduced by the adversary along the way).

These are the steps we follow in a few different MPC circuit evaluation protocols, as we have discussed. The way we compute the circuit differs (slightly) with the protocol.

_**Next time**_: In the [next part](https://bristolcrypto.blogspot.co.uk/2016/10/what-is-spdz-part-2-circuit-evaluation.html) in this series, we will see how to use these secret-shared values to evaluate an arithmetic circuit as in the SDPZ protocol.


---
created: 2022-03-02T13:42:53 (UTC +08:00)
tags: []
source: https://bristolcrypto.blogspot.com/2016/10/what-is-spdz-part-2-circuit-evaluation.html
author: 
---

# Bristol Cryptography Blog: What is SPDZ? Part 2: Circuit Evaluation

> ## Excerpt
> This blog post is the second in a  series of three in which we describe what SPDZ is. The first part can be found here  and the third here ...

---
This blog post is the second in a series of three in which we describe what SPDZ is. The first part can be found [here](https://bristolcrypto.blogspot.co.uk/2016/10/what-is-spdz-part-1-mpc-circuit.html) and the third [here](https://bristolcrypto.blogspot.co.uk/2016/11/what-is-spdz-part-3-spdz-specifics.html).

In this part we discuss how we perform the additions and multiplications of shared secret values as in the SPDZ protocol.

### The Preprocessing Model

The preprocessing model is a framework in which we assume the existence of a 'trusted dealer' who distributes 'raw material' to the parties before any circuit evaluation takes place. The reason for doing this is that if we have the 'raw material' already in place, the circuit can be evaluated very efficiently.

We realise this by splitting the protocol into two phases: a 'preprocessing' (or 'offline') phase where we generate the preprocessed data, and an 'online' phase where we evaluate the circuit. (The term 'offline' is a little misleading, since the parties still communicate.)

### Evaluating the Circuit

We now suppose that the parties have agreed on some arithmetic circuit they want to evaluate jointly.

#### Providing input

The first step of the circuit evaluation is to read in values from (some of) the parties. In SPDZ, this means getting each party with input, $P_i$ with $x^i$ (the superscript $i$ is an index to avoid confusion with $x_i$ which is a share of $x$), to secret-share input $x^i$ to the other parties.

To do this, we need to use a share of a random value: each party $P_i$ holds some $r_i$ and the value $r = \sum_i r_i$ is uniformly random and not known by any party. First, every party sends their share $r_j$ of $r$ to party $P_i$. This is called a 'partial opening' because $P_i$ now discovers the value of $r$ (by summing the shares).

Party $P_i$ then broadcasts the value $x^i - r$. Party $P_1$ sets its share of $x^i$ as $x_1^i=r_1 + x^i - r$, and $P_j$ for $j > 1$ sets its share of $x^i$ as $x^i_j=r_j$. (In practice, it is not always $P_1$ who does a different computation.)

Thus we have turned $x^i$ into $\langle x^i \rangle$.

#### Addition

Suppose we want to compute some arithmetic circuit on our data; that is, some series of additions and multiplications. We have the following very useful properties of an additive sharing scheme:

-   If we have some secret-shared field elements $\langle a \rangle$ and $\langle b \rangle$, so party $P_i$ has $a_i$ and $b_i$, each party can locally compute $a_i + b_i$, and hence, since $\sum_i a_i + \sum_i b_i =
     \sum_i (a_i+b_i)$, they obtain a secret sharing $\langle a + b \rangle$ of the value $a+b$.
-   If each party multiplies its share by some public value $\alpha$, then since $\sum_i \alpha a_i = \alpha \sum_i a_i = \alpha a$ the parties obtain a secret sharing $\langle \alpha a \rangle$ of the value $\alpha a$.

In other words, the secret-sharing is linear; this is good because it means there is no communication cost for doing these operations. Unfortunately, we have to do a bit more work to multiply secret-shared elements.

#### Multiplication

In 1991, Donald Beaver \[2\] observed the following: suppose we want to compute $\langle x y \rangle$ given some $\langle x \rangle$ and $\langle y \rangle$, and we already have three secret-shared values, called a ‘triple’, $\langle a \rangle$, $\langle b \rangle$ and $\langle
 c \rangle$ such that $c = a b$. Then note that if each party broadcasts $x_i-a_i$ and $y_i - b_i$, then each party $P_i$ can compute $x-a$ and $y-b$ (so these values are publicly known), and hence compute

$$
z_i=c_i + (x-a) b_i + (y-b) a_i
$$

Additionally, one party (chosen arbitrarily) adds on the public value $(x-a)(y-b)$ to their share so that summing all the shares up, the parties get

$$
\sum_i z_i = c + (x-a)b + (y-b)a + (x-a)(y-b) = xy
$$

and so they have a secret sharing $\langle z \rangle$ of $xy$.

The upshot is that if we have lots of triples, then we can perform many multiplications on secret-shared data and hence we can compute any arithmetic circuit. (Note that a triple cannot be reused because this would reveal information about the secrets we are trying to multiply - i.e. since $x-a$ and $y-b$ are made public in the process above)

Importantly, observe that not only do these triples not depend on the input secret-shared values they are used to multiply, they are also completely independent of the circuit to be evaluated. This means that we can generate these triples at any point prior to evaluating the circuit. The values of $a$, $b$ and $c$ are not known by any parties when generated - each party only knows a share of each of 'some values' for which they are told this relation holds.

Moreover, if these triples are generated in the offline phase at some point before the circuit evaluation, since addition, scalar multiplication and field multiplication are inexpensive in terms of communication and computation, the online phase is both highly efficient and information-theoretically secure.

Combining the above, we have now established roughly how SPDZ evaluates an arithmetic circuit.

_**Next time**_: In [the final part](https://bristolcrypto.blogspot.co.uk/2016/11/what-is-spdz-part-3-spdz-specifics.html), we will look at what makes SDPZ different from other MPC protocols which achieve the same thing.


---
created: 2022-03-02T13:43:27 (UTC +08:00)
tags: []
source: https://bristolcrypto.blogspot.com/2016/11/what-is-spdz-part-3-spdz-specifics.html
author: 
---

# Bristol Cryptography Blog: What is SPDZ? Part 3: SPDZ specifics

> ## Excerpt
> This blog post is the final in a  series of three in which we describe what SPDZ is. The first part can be found here , and the second here ...

---
This blog post is the final in a series of three in which we describe what SPDZ is. The first part can be found [here](https://bristolcrypto.blogspot.co.uk/2016/10/what-is-spdz-part-1-mpc-circuit.html), and the second [here](https://bristolcrypto.blogspot.co.uk/2016/10/what-is-spdz-part-2-circuit-evaluation.html).

In this part, we consider what it is that makes SPDZ SDPZ.

### SPDZ MPC

Using the term SPDZ is perhaps a little misleading. There are actually a few protocols which we call SPDZ because they are really just improvements on the original. As the online phase of SPDZ is already ‘pretty good’, optimisations to the protocol normally involve improving the offline phase.

In what follows, we talk about the methods used in SPDZ for preprocessing (generating the triples) and then explain some improvements that have been made.

#### Original SPDZ

Here we will outline some of the methods in the original SPDZ protocol, as in \[5\].

_Preprocessing_

In the first SPDZ paper, the authors show how to use somewhat homomorphic encryption (SHE) to generate the triples. This is a (pretty expensive) public key cryptosystem in which the encryption operation is ring homomorphic - that is,

$$
\textsf{Enc}_{\textsf{pk}}(a)\boxplus\textsf{Enc}_{\textsf{pk}}(b)=\textsf{Enc}_{\textsf{pk}}(a+b)
$$

and

$$
\textsf{Enc}_{\textsf{pk}}(a)
 \boxdot \textsf{Enc}_{\textsf{pk}}(b) = \textsf{Enc}_{\textsf{pk}}(a 
\cdot b)
$$

(For those who know about FHE: we can do MPC with FHE, as you’d hope; however, at the time of writing no FHE scheme which is competitive with current secret-sharing MPC currently exists. SPDZ is what the authors call the ‘ideal use of FHE in MPC’: raw material is computed locally so that the online computations are minimal and communication overhead is small.)

During the offline phase, we also generate lots of random values which are used by the parties to provide input to the circuit.

_Sacrificing_

In order to check that the adversary has not introduced errors into the triples, for each triple we want to use we have to ‘sacrifice’ a triple by taking another triple from the preprocessing and checking that the third element is the product of the first two using Beaver’s method outlined in the [previous post](https://bristolcrypto.blogspot.co.uk/2016/10/what-is-spdz-part-2-circuit-evaluation.html). Fortunately the method by which triples are generated is very efficient.

_ZKPoPKs_

Zero-Knowledge Proofs of Plaintext Knowledge (ZKPoPKs) are a way of ensuring parties encrypt as they should: when encrypting in the SHE scheme, there are conditions on the plaintext and noise that need to be met to ensure the ciphertext is well-formed.

_MACs_

Before any preprocessing computation takes place, the parties agree on some MAC key $\alpha$ with which they MAC all their data and which they secret-share amongst themselves so that no individual party knows the MAC key.

This MAC key is used to MAC all the data in the circuit. For each secret-shared _value_, there is also a secret-shared MAC on that value.

After the circuit has been evaluated, the parties open the secret corresponding to the circuit output and also the MAC on it, and check that the MAC is correct. If an actively corrupt party cheats at any point in the circuit evaluation, the final MAC check reveals this has happened. Note that this means the parties could be computing on invalid data.

In the original paper, each party also MACked the global MAC key with a personal MAC key. This was not needed in subsequent works.

  

#### Updated SPDZ: SDPZ2

While the online phase of SPDZ was (almost) the same, in SPDZ2 \[4\] the authors made significant changes to the offline phase of the original protocol. We outline the major changes here.

They solved an open problem left by the first paper in giving a secure protocol which generated a shared SHE decryption key.

They also offered various performance enhancements, including the introduction of ‘squares’ and ‘bits’ in the preprocessing, which allowed faster online computations of specific operations in the circuit.

A major improvement was allowing MACs to be checked on data before the end of the computation. This involved checking the MAC of a public value that depended on the underlying data. The big advantage of this is that the preprocessed data MACked under the same key can still be used after this MAC check, but not in the original protocol since the check reveals the key.

The new protocol had covert and malicious variants, the former having lesser parameter requirements. A covertly secure protocol ensures the adversary wins with probability at most $1/n$ for some $n \in 
\mathbb{N}$. The reason for this was that it was believed more closely to match the real-world situation.

#### New SPDZ: MASCOT

The most recent improvements to the SPDZ protocol is a work known as MASCOT \[3\]. As with SPDZ2, the online phase of SPDZ was left as it was; this paper improves on the offline phase, showing how to use oblivious transfer (OT) to achieve much faster preprocessed data than by using SHE.  It is authored by Bristol’s own Marcel, Emmanuela and Peter, and appeared at [CCS](https://www.sigsac.org/ccs/CCS2016/) last week.

(Oblivious transfer is a process in which one party inputs two strings and a second party chooses exactly one; this is done in such a way that the first party doesn’t know which string the second chose, and the second does not learn anything about the string it didn’t choose.)

The authors of MASCOT note that the heavy machinery of public-key cryptography required in the original SPDZ paper to generate the preprocessing (namely, the use of SHE) prevents the communication from being a bottleneck since the local computation takes so long.

The protocol is actively secure and performs significantly faster than previous SPDZ2 implementations, with much lower communication overhead.

#### The Future of SPDZ

MASCOT has only just been released, and there are already whispers of a new paper. Watch this space...

  

#### References

\[1\] B. Applebaum, Y. Ishai, and E. Kushilevitz. [How to garble arithmetic circuits](http://www.eng.tau.ac.il/~bennyap/pubs/AGCfull.pdf). 52nd FOCS, pp120–129. IEEE Computer Society Press, 2011

\[4\] I. Damgard, M. Keller, E. Larraia, V. Pastro, P. Scholl, N. P. Smart. [Practical covertly secure MPC for dishonest majority - or: Breaking the SPDZ limits](https://eprint.iacr.org/2012/642). In ESORICS (2013), J. Crampton, S. Jajodia, and K. Mayes, Eds., vol. 8134 of Lecture Notes in Computer Science, Springer, pp. 1–18.

\[8\] J. Buus Nielsen, P. Nordholt, C. Orlandi, and S. Burra. [A new approach to practical active-secure two-party computation](https://eprint.iacr.org/2011/091). In Reihaneh Safavi-Naini and Ran Canetti, editors, Advances in Cryptology CRYPTO 2012, volume 7417 of Lecture Notes in Computer Science, pp681-700. Springer Berlin Heidelberg, 2012.

\[9\] A. Shamir. [How to Share a Secret](https://dl.acm.org/citation.cfm?id=359176). In Communications of the ACM, Volume 22 Issue 11, Nov. 1979, pp612-613.

\[10\] A. Yao. [How to generate and exchange secrets](https://dl.acm.org/citation.cfm?id=1382944). In SFCS '86 Proceedings of the 27th Annual Symposium on Foundations of Computer Science, pp162–167. IEEE, 1986.
