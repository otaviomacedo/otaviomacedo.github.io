---
layout: post
title:  "Centrality (change this title)"
date:   2021-03-28 12:00:00 +0000
categories: graph-theory
---

<!-- Introduction about measuring the importance of nodes in a grah. Mention the term centrality -->

One natural and simple way to define centrality is by using the **degree** of each node as the measure, that is, the number of other nodes connected to it. In a social network, for example, popular people, such as politicians and TV celebrities, know a lot of people. You could argue that they have a lot of influence because of that. And this is true, but it misses the fact that some nodes may have a lot of influence even if they are not well connected, as long as their connections are. An assistant to the Prime Minister can exert more influence than someone who know a lot of unimportant people.

This observation leads to a better measure, in which the influence of a node depends on the influence of its neighbors blablablablabl. 

## Computation (change this title)

This idea makes intuitive sense, but there is an obvious complication to it: how do you compute this metric? To know my influence, I need to take into account the influence of all my friends. But to compute _their_ influence I need to know, among other things, my influence. The situation seems hopelessly circular: everything depends on everything else. It's like that quip about dictionaries, in which every word is defined by other words and each of those words is, in turn, defined by other words and so on.


And you may ask: "is it even possible to come up with any assignment of influences that is consistent across any given graph?" The surprising fact is that, not only is it possible, but we can use linear algebra to exploit this circularity and find an elegant solution. The first thing we need to do is define this metric more precisely. Let's say we have a graph like this:

{: .center}
![](/assets/neighbors.svg)

and we want to compute the influence $$x$$ of node 7. We want to define it as the sum of the influences of its neighbors multiplied by some non-zero constant, which we're going to call $$\alpha$$:

$$x_7 = \alpha(x_2 + x_6 + x_8)$$

In this equation, we are only including the neighbors of 7, but we can express the same thing in terms of all the nodes, using binary coefficients to turn them on or off, so to speak:

$$x_7 = \alpha(0x_1 + 1x_2 + 0x_3 + 0x_4 + 0x_5 + 1x_6 + 0x_7 + 1x_8)$$

For non-neighbors, we multiply their influence by 0. For neighbors, we multiply by 1. Applying the same idea to every node:

$$
\begin{eqnarray}
x_1 &=& \alpha(a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n)\\

x_2 &=& \alpha(a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n)\\

&\vdots&\\

x_n &=& \alpha(a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nn}x_n)
\end{eqnarray}
$$

where $$a_{ij}$$ is 1 if nodes $$i$$ and $$j$$ are connected and 0 otherwise. When dealing with systems of linear equations like this, it's more convenient to represent it in terms of matrix multiplication:

$$
\alpha
\begin{bmatrix} 
a_{11} & a_{12} & \cdots & a_{1n} \\
a_{21} & a_{22} & \cdots & a_{2n} \\
\vdots  & \vdots  & \ddots & \vdots  \\
a_{n1} & a_{n2} & \cdots & a_{nn}
\end{bmatrix}
\begin{bmatrix}
x_1 \\ x_2 \\ \vdots \\ x_n
\end{bmatrix}
=
\begin{bmatrix}
x_1 \\ x_2 \\ \vdots \\ x_n
\end{bmatrix}
$$

The matrix with the $$a_{ij}$$ coefficients is called the **adjacency matrix** of the graph. Let's call this matrix $$\mathbf{A}$$ and the vector with the influence measures of every node, $$\mathbf{x}$$. Then the equation above can be more succintly rewritten as:

$$\alpha\mathbf{Ax} = \mathbf{x}$$

Given that $$\alpha$$ is non-zero, we can multiply both sides by $$\lambda = 1/\alpha$$:

$$\mathbf{Ax} = \lambda\mathbf{x}$$

Here is where the beauty of linear algebra comes into the picture. Let's forget for a moment what these matrices and vectors mean and just focus on the algebraic aspect of this equation. What it's saying is that we want to find a vector such that, multiplying it by a matrix (on the right) is the same as multiplying it by a single value. A vector that satisfies this property is called an **eigenvector** of the matrix; and the value that multiplies it is called an **eigenvalue**. 

In general, a matrix with real numbers may not have any eigenvetors composed only of real numbers (but if we are willing to accept complex numbers, any matrix will have at least one pair of eigenvalue-eigenvector). Moreover, an $$n$$ by $$n$$ matrix may have as many as $$n$$ distinct such pairs.

Fortnuately, for the type of matrix we are discussing here (square matrices with only non-negative entries), we know that there is one eigenvector composed only of non-negative entries. Its corresponding eigenvalue is the largest one (in absolute value) among all the eigenvalues. This is the vector we are looking for, also known as the **dominant eigenvector**. This is known as the **[Perron-Frobenius Theorem]** Each entry in the dominant eigenvector is the centrality measure that solves the equation above.

This is the core idea behind algorithms like PageRank, used by Google to, well, rank every page on the web according to its influence. Along with other computations, this rank influences the order of the search results.

## Weights

The graphs we've been considering only have unweighted edges. Any pair of vertices either is connected or not. And that's why the entries of the adjacency matrix are either 0 or 1. But we can extend this idea of eigenvector centrality to weighted graphs. Their corresponding adjacency matrix, in this case, can contain any non-negative integer, representing the weight of the edges.

For example, there is a website called [meetup.com][meetup], in which users can join groups of interest and meet other people. This is an example of a graph in which we have two types of vertices: users and groups (also called **bipartite**). A user may be a member of many groups and each group may have many members. But we can decide to focus only on the groups and define that there is an edge between two groups if they have share at least one member.

This is what the graph of meetup groups in the Nashville, TN, area in the United States looks like (credit to Stephen Bailey, who posted the data on [Kaggle][kaggle]):

![Nashville meet-up groups](/assets/nashville.png)

This graph has 602 vertices and 6692 edges. The weight of each edge (represented in the figure by the thickness of the line) is the number of users the adjacent groups have in common. Computing the dominant eigenvector for this graph and taking its largest element, we find out that the most central group is [Eat Love Nash][eatlovenash]. In their own words, "it's a laid back social group just doing fun get togethers. We're open to all sorts of different activities from dining out to hiking to trivia.", which kind of makes sense for a very central group.

The image above and the eigenvalue/eigenvector computation was done in [Gephi], an amazing open source graph application, with all sorts of features and plugins.


[meetup]: https://meetup.com
[kaggle]: https://www.kaggle.com/stkbailey/nashville-meetup
[eatlovenash]: https://www.meetup.com/EatLoveNash/
[dance]: https://www.meetup.com/steppingoutsocialdance/
[Gephi]: https://gephi.org
[Perron-Frobenius Theorem]: https://en.wikipedia.org/wiki/Perron–Frobenius_theorem