---
title: Bipartite Perfect Matching in Deterministic NC
url: https://blog.computationalcomplexity.org/2026/07/bipartite-perfect-matching-in.html
published: "2026-07-19T14:27:01Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-1087736660426121662
---

# Bipartite Perfect Matching in Deterministic NC

*Nutan Limaye and Thore Husfeldt guest post on the [new deterministic parallel algorithm for bipartite perfect matching](https://eccc.weizmann.ac.il/report/2026/100/) by Abhranil Chatterjee, Sumanta Ghosh, Rohit Gurjar, Roshan Raj and Thomas Thierauf.*

The post will try to explain three main things about the result. What is the result? Why is it important? And finally, how did the authors prove it?

We will assume that the reader is an undergraduate student in CS (i.e., the reader knows basics of discrete mathematics, linear algebra, and algorithm design).

### What?

The main result can be stated in just one line! Bipartite Perfect Matching can be solved in NC. Let's now understand what each of these terms means.

**Bipartite Perfect Matching.** A bipartite graph has two disjoint sets of vertices, say \\(L, R\\), and any edge connects one vertex of \\(L\\) and one vertex of \\(R\\). A matching in a graph is a subset of edges such that no two edges have a vertex in common. A perfect matching is a matching in which each vertex of the graph appears exactly once. Let's take the following example. Here is a bipartite graph.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEht907VnEsDY23g2f-dFeB5qss7PCLiwlyi_PVnWmaEHytILfVuPXFLzLOWvKTxRKt6HA541vsCWEvaZKN4UFOkcM-RThPqZyAhsMoWkOrdvizV07zQgNMfMR-NiwBGt2aUskGdD3nrvag8yotvlLfRIZJKggiqJWXu4s0xJGYddXjFXTe01Lus/s320/graph.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEht907VnEsDY23g2f-dFeB5qss7PCLiwlyi_PVnWmaEHytILfVuPXFLzLOWvKTxRKt6HA541vsCWEvaZKN4UFOkcM-RThPqZyAhsMoWkOrdvizV07zQgNMfMR-NiwBGt2aUskGdD3nrvag8yotvlLfRIZJKggiqJWXu4s0xJGYddXjFXTe01Lus/s354/graph.png)

It has two perfect matchings. One is \\(\\{(v\_1, w\_1), (v\_2, w\_3), (v\_3, w\_2)\\}\\) and the other is \\(\\{(v\_1, w\_2), (v\_2, w\_1), (v\_3, w\_3)\\}\\).

We will assume that the graph is given as a matrix, known as the bi-adjacency matrix. The *bi-adjacency* matrix of a bipartite graph has \\(A\_{ij} = 1\\) whenever \\((i,j)\\in E(G)\\). For our running example, here is how we will receive the input: \\\[ A\_G = \\begin{bmatrix} 1 & 1 & 0 \\\ 1 & 0 & 1 \\\ 0 & 1 & 1 \\end{bmatrix}\\,. \\\] Now, we are ready to define the Bipartite Perfect Matching (BPM for short) problem. *Given a bipartite graph as a bi-adjacency matrix, check whether there is a perfect matching in the graph or not.*

**NC.** The next term we need to explain is NC. It is a complexity class named after [Nick Pippenger](https://en.wikipedia.org/wiki/Nick_Pippenger), which consists of a class of problems solvable by parallel algorithms that use polynomially many processors and have parallel running time that is considerably smaller than polynomial. More specifically, the parallel running time is polylogarithmic. You can simply think of this as a class of problems that have efficient parallel algorithms.

Let us begin with a simple example. Suppose we want to multiply \\(n\\) numbers, \\(x\_1,\\ldots,x\_n\\). One approach is to first multiply \\(x\_1\\) and \\(x\_2\\), then multiply the result by \\(x\_3\\), and continue in this way until we reach \\(x\_n\\). This is a sequential algorithm.

A parallel algorithm proceeds differently. It first pairs up the numbers and multiplies all pairs simultaneously. This produces \\(n/2\\) numbers, giving a new instance of the same problem with only half as many inputs. We then repeat the process: pair up the remaining numbers, multiply each pair in parallel, and continue until only one number remains.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg0Gv3IXCzcofNfiBEbSJuvmtJ7LctXMSG-knvaS4fJmE2L0qD-Tudj1cptuiHn-ur9p-mE7PKgycwMRNkqjFyl-QbbFJyo2bIlfiwY9E-nfmvrV3U8wHjSOzD8yVdAAS3bva-c-vlkQkTfACC5bO9MXApgmlfBiHBL1qltLlFa51e6UKKTCLOJ/w400-h113/nc-example.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg0Gv3IXCzcofNfiBEbSJuvmtJ7LctXMSG-knvaS4fJmE2L0qD-Tudj1cptuiHn-ur9p-mE7PKgycwMRNkqjFyl-QbbFJyo2bIlfiwY9E-nfmvrV3U8wHjSOzD8yVdAAS3bva-c-vlkQkTfACC5bO9MXApgmlfBiHBL1qltLlFa51e6UKKTCLOJ/s1758/nc-example.jpg)

Consider another example. Given an integer matrix \\(A\\), suppose we wish to compute its determinant; we refer to this as the DET problem. Sequentially, the determinant can be computed efficiently using Gaussian elimination. More surprisingly—and this is far from obvious—the problem also admits an efficient parallel algorithm (DET \\(\\in\\) NC). (Here are some references: [Csanky's paper](https://cs.uwaterloo.ca/~r5olivei/courses/2025-spring-cs860/C76-fast-parallel-determinant.pdf), [Berkowitz' paper](https://cs.uwaterloo.ca/~r5olivei/courses/2021-winter-cs487/berkowitz-paper.pdf).) We will use this statement as a black box many times below.

So, overall the new result states that given a bi-adjacency matrix of a graph, checking whether it has a perfect matching or not can be solved efficiently by a parallel algorithm.

### Why it matters: TL;DR

- BPM is an extremely well-studied graph problem because it arises naturally in many practical scenarios. The problem has been a focus of intense research for more than 5 decades. (See an excellent introduction to matching and related problems [here](https://www.ams.org/books/chel/367/chel367-endmatter.pdf) and [here](https://math.mit.edu/~goemans/18433S13/matching-notes.pdf).)

- Derandomization is a central problem in theoretical computer science. (For a deep dive into derandomization see [this](https://www2.cs.sfu.ca/~kabanets/papers/derand_survey.pdf) or [this](https://people.seas.harvard.edu/~salil/pseudorandomness/pseudorandomness-Aug12.pdf) survey.) It asks whether randomness truly adds computational power or merely provides a convenient shortcut. Bipartite Perfect Matching has long been known to admit a randomized NC algorithm. Finding a deterministic NC algorithm for the problem therefore fits naturally into the broader derandomization program. In fact, bipartite perfect matching is one of the most natural and simply stated problems in this setting. Its resolution provides a particularly striking example of randomness being removed from an efficient parallel algorithm.

### How did the authors solve the problem?

Polynomial-time algorithms for this problem---Kuhn's algorithm, Hopcroft--Karp, or matching-via-max-flow with Ford--Fulkerson---are a staple of undergraduate algorithms courses and have been known since the 1960s. They work by repeatedly finding an *augmenting path*: a path between two unmatched vertices that alternates between edges outside and inside the current matching; augmenting along it grows the matching by one edge (Berge's theorem)---an inherently sequential process. The matching that admits an augmenting path at step \\(k\\) depends on exactly which edges were chosen at steps \\(1\\) through \\(k-1\\); there's no obvious way to precompute or guess ahead of time which augmentations will happen. Thus, the classical algorithms for the problem are sequential.

**The randomized NC algorithm for BPM.** As a warm up, let us see the randomized NC algorithm for BPM. The algorithm is as follows.

- Start with the given bi-adjacency matrix \\(A\_G\\) of the bipartite graph \\(G\\).

- Replace each \\(1\\) entry by a random number from the range \\(\[1, 100n\]\\), where \\(n\\) is the number of vertices in \\(G\\). This creates a new matrix \\(A\_G'\\).

- Compute DET of \\(A\_G'\\).

- If it is non-zero, then accept else reject.

Using the fact that DET \\(\\in\\) NC as a black box, it is straightforward to see that the algorithm above runs in randomized NC. But why is it correct? Establishing correctness requires a connection between determinants and perfect matchings. At a very high level, the terms of the determinant of \\(A\_G'\\) are in one-to-one correspondence with the perfect matchings of \\(G\\). Thus computing the determinant leads to checking the existence of a matching. For more rigorous details about this connection, we refer the reader to Lovász' [paper](https://www.math.uwaterloo.ca/~harvey/W11/1979-Lovasz-OnDeterminantsMatchingsAndRandomAlgs.pdf).

The next question is whether the randomness can be removed. Informally, the randomization is needed to prevent cancellations in the determinant. Recall that computing a determinant involves both additions and subtractions. Thus, even if the graph has a perfect matching, different terms in the determinant expansion may cancel, causing the determinant to vanish. This would make the algorithm incorrect.

To avoid this, each nonzero entry is replaced by a random value. With high probability, the resulting determinant is nonzero whenever a perfect matching exists. The challenge, then, is to achieve the same effect deterministically: how can we prevent these cancellations without relying on randomness?

The first major breakthrough came about a decade ago, in [work](https://eccc.weizmann.ac.il/report/2015/177/) by Fenner, Gurjar, and Thierauf -- notice the overlap with the authors of the new result. Their approach traded randomness for parallel resources: they showed that the algorithm could indeed be derandomized, but only by allowing many more processors. Specifically, their algorithm required quasi-polynomially many processors. Recall that a polynomial function grows as \\(n^c\\) for some constant \\(c\\), whereas a quasi-polynomial function may grow as \\(n^{\\log n}\\). In the latter case, the exponent of \\(n\\) is itself a growing function of \\(n\\), rather than a fixed constant.

#### The new approach

Let us reiterate the question we posed after reviewing the randomized NC algorithm.

*How can we prevent these cancellations without relying on randomness?*

The authors say, the answer lies in coding theory! At a very high level, the algorithm can be described as follows.

- Start with the given bi-adjacency matrix \\(A\_G\\) of the bipartite graph \\(G\\).

- Replace each \\(1\\) entry in location \\((i,j)\\) with a fixed *thin and tall* matrix \\(V\_{ij}\\) to produce a new matrix \\(A\_G'\\).

- Compute DET of \\(A\_G'\\).

- If it is non-zero, then accept else reject.

If you feed this algorithm to a computer, it will complain: `Cannot compute DET because AG' is not a square matrix.` But this is not a big deal. One could simply say, check whether it has full rank (either row-rank or column rank, whichever is smaller). This problem is also solvable in NC.

However, instead of replacing the original adjacency matrix, they first create an intermediate matrix \\(\\widehat{A\_G'}\\) from \\(A\_G\\) by padding \\(A\_G\\) with \\(n(n-1)\\) columns of unit vectors. For our example, \\\[ \\widehat{A\_G'} = \\begin{bmatrix} 1 & 1 & 0 \\quad & 1 & 1 \\,& 0 & 0\\, & 0& 0\\\ 1 & 0 & 1 \\quad & 0 & 0 \\,& 1 & 1\\, & 0& 0\\\ 0 & 1 & 1 \\quad & 0 & 0 \\,& 0 & 0\\, & 1& 1\\\ \\end{bmatrix} \\\] Now, in \\(\\widehat{A\_G'}\\), replace each non-zero entry \\((i,j)\\) with a thin and tall matrix \\(V\_{ij}\\) to obtain \\(\\widehat{A\_G}\\). The matrix dimension and the padding length are chosen such that the overall matrix \\(\\widehat{A\_G}\\) becomes almost square. It now has slightly fewer rows than columns.

Note that the entire algorithm is deterministic. The main work is to show that this deterministic replacement works. Specifically, the authors show that

\\(G\\) has a perfect matching if and only if \\(\\widehat{A\_G}\\) has full row rank.

While we don't plan to present the proof here, the goal is to give you a high-level outline of the approach. In order to do that, let us first see what these matrices \\(V\_{ij}\\) look like. This is exactly where the connection to coding theory comes up! Each matrix is indeed a Folded Vandermonde matrix, which comes up in folded Reed-Solomon codes and subspace designs.

Fix \\(\\gamma\\in \\mathbb{F}\\) that has a large order, i.e. it needs to be raised to a very large power before it circles back to itself. (This is relevant when we are over finite fields.) For parameters \\(r,D\\in \\mathbb{N}\\), the *folded Vandermonde* matrix is the \\(D\\times r\\) matrix whose \\(ij\\)th entry is \\(\\left(\\alpha\\gamma^{(j-1)}\\right)^{i-1}\\): \\\[ V(\\alpha\_{}, \\gamma) = \\begin{bmatrix} 1 & 1 & \\cdots & 1 \\\ \\alpha\_{} & \\alpha\_{}\\gamma & \\cdots & \\alpha\_{}\\gamma^{r-1} \\\ \\alpha\_{}^2 & (\\alpha\_{}\\gamma)^2 & \\cdots & (\\alpha\_{}\\gamma^{r-1})^2 \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\vdots & \\vdots & \\vdots & \\vdots \\\ \\alpha\_{}^{D-1} & (\\alpha\_{}\\gamma)^{D-1} & \\cdots & (\\alpha\_{}\\gamma^{r-1})^{D-1} \\\ \\end{bmatrix} \\\] The parameters \\(r,D\\) are chosen such that the matrix is thin and tall. For our example \\(G\\) with \\(n=3\\), the matrix ends up as having \\(D=n(n^3+1-n)=75\\) rows and \\(r=n^3+1=28\\) columns. To obtain \\(\\widehat{A\_G}\\) from \\(\\widehat{A\_G'}\\), replace all the \\(1\\) entries from each column \\(j\\) of \\(\\widehat{A\_G'}\\) by \\(V(\\alpha\_j, \\gamma)\\), where all \\(\\alpha\\)s are distinct.

Swastik Kopparty and Shubhangi Saraf [recently announced](https://eccc.weizmann.ac.il/report/2026/119/) a variation of this approach.

**How does the proof proceed from here?** One direction of the proof is quite easy. When there is no perfect matching, it is quite easy to see that the matrix cannot achieve full row rank. This is because, when there is no perfect matching, then by Hall's Theorem we know that there is a *Hall blocker*. That is, there is a set \\(S \\subseteq L\\) such that if you look at the neighbours of \\(S\\), denoted as \\(N(S) \\subseteq R\\), then \\(\|S\| > \|N(S)\|\\). This deficit in the size of the neighbour set for one of the subsets in \\(L\\) suffices to observe that the number of columns with nonzero entries is strictly less than the number of rows. Thus, one cannot have full row rank.

For the other direction, things are more intricate. The proof proceeds by proving the contrapositive. They show that if the row rank of the matrix is not full, then there will be a subset \\(S \\subseteq L\\) such that \\(\|S\| > \|N(S)\|\\).

To prove this, they start with a vector \\(v\\) that certifies the linear dependence between the rows of \\(\\widehat{A\_G}\\). This vector is viewed as a tuple of \\(n\\) vectors of dimension \\(D\\) each. Now, they prove two things about it.

- First, they show that the non-zero elements of the tuple, say \\(\\mathcal{P}\\), are linearly independent vectors.

- Second, they view these vectors as polynomials and consider the span of these linearly independent polynomials. They count the *folded roots* of the span of \\(\\mathcal{P}\\) in two ways to derive the fact that \\(\|S\| > \|N(S)\|\\).

Both the parts above invoke a Lemma due to Guruswami and Kopparty which proves an upper bound on the number of folded roots of a linear space of polynomials.

### Conclusion

Here are some final thoughts.

- The paper proves many more results. The proofs are self-contained and well-written. So, I highly encourage you to read it.

- Venkat Guruswami gave a keynote at ICALP 2026, where he explained how one can view the idea of subspace designs as a derandomization tool. He explained subspace designs by analogy with the more familiar notion of hash functions: subspace designs play for linear spaces a role similar to that played by hash functions for sets. He then sketched the BPM in NC proof from this perspective.

- One big open problem is resolved by humans. Going forward, will we have such only-human proofs? In fact, the authors first developed an algorithm for the problem, then used AI to help adapt the underlying idea to a broader setting. That is, they used AI to extend human ideas. Currently, this seems to be a trend in TCS. What can we expect going forward?
