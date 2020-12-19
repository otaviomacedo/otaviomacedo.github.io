---
layout: post
title:  "Semilattices in Tic-Tc-Toe"
date:   2020-12-19 06:40:00 +0000
categories: algebra
---
{% include mathjax.html %} 

Structure:

* Introduction etc
* Brief explanation of the program's design
* Mention the need to compute the winner
* Describe how this should be done using a diagram
* Show the outcome method and how it relates to the diagram
* This is fine, but the deeper algebraic connection is lost
* Discuss:
  - First of all, both are reduce
  - How product is associative, commutative and idempotent -> semilattice
  - This induces an ordering
  - Every subset has a maximal
* Refactor with the maximal
* Conclusion: not too big of a deal, but it shows how algebraic structures appear everywhere

---

The game is structured into three main abstractions: Board, Game and Player, of which we have two implementations: Computer and User. Conceptually, a game starts with two players and an empty board and unravels it. In other words, it generates a sequence of states (the different board configurations as the game progresses, from beginning to end) corresponding to the alternating moves of the two players.

![](/assets/ttt-design.png)

Let's focus on the Board. As the name suggests, it describes a configuration of the game's board, that is, which marks are placed on which positions. For example, a particular instance of Board could have the configuration below:

<!-- ![Image showing a winning configuration]() -->

The main question we want Board to answer is: given this configuration, who is the winner (if any)? We can implement this by applying two consecutive transformations to the data:

<!-- ![Image showing the transformations: blue and green]() -->

In the blue transformation, given a sequence of marks, if they are all equal, we want to output that mark; otherwise, the mark representing the empty cell. In the green transformation, given a sequence of marks, we want to output the first one that we find that is not the empty cell (given the rules of the game, there can be at most one such mark); otherwise, the mark representing the empty cell.

A possible implementation of this process in Scala is:

```scala
lazy val outcome: GameOutcome = {
  def ⊙(a: GameOutcome, b: GameOutcome): GameOutcome = 
    if (a == b) a else None

  def winner(positions: Seq[Int]): GameOutcome = 
    positions map cells reduce ⊙

  val allLines = Vector(rows, columns, diagonals).flatten
  allLines map winner find(_.isDefined) getOrElse None
}
```
<!-- Briefly talk about the types: GameOutcome, Symbol -->

This is perfectly fine. The function `⊙` corresponds to the blue transformation in our diagram and `find(_.isDefined) getOrElse None` in the last line corresponds to the green one. This is the intuitive notion we have about what should happen and more or less matches the verbal description I've given for it. But there is a deeper connection between these two transformations that is lost in this particular implementation.

First off, we use `reduce` to go from a sequence of symbols to a single symbol in the blue transformation. It would be satisfyingly symmetrical if we could reframe the green one in term of reduction, as well. So, what would that function look like? With a bit of thought and experimentation, we can quickly arrive at the solution. But I would like to take a detour through the land of Abstract Algebra and Order Theory.

## Semilattices

The operation `⊙` has three important properties:

* __Idempotence__: for any symbol `a`, `⊙(a, a) == a`.
* __Commutativity__: for any two symbols `a` and `b`, `⊙(a, b) == ⊙(b, a)`.
* __Associativity__: for any three symbols `a`, `b` and `c`, `⊙(a, ⊙(b, c)) == ⊙(⊙(a, b), c)`.

In Abstract Algebra, sets with an operation that satisfies these three properties is called a semilattice. The interesting thing about a semilattice is that you can use its operation to define a relation ≤ between two elements. Using mathematical notation, this relation is defined as:

\\[
a \odot b = a \iff a \leq b    
\\]

In other words, if I take any two elements $$a$$ and $$b$$ and compute the result of the operation $$\odot$$ between them, the result may be $$a$$ or something else. If it's $$a$$, we say that $$a \leq b$$. And vice-versa: if, given two elements $$a$$ and $$b$$, if we know that $$a \leq b$$, then we can conclude that $$a ⊙ b = a$$. Now, one important caveat is that, given two elements $$a$$ and $$b$$, it may just happen that neither $$a \leq b$$ nor $$b \leq a$$ is true. We simply cannot compare them.

Mathematicians say that the operation induces a partial order on the `GameOutcome` set. So we have:

$$\texttt{\_} \leq a$$
$$a \leq a$$
* $$a \leq a$$
* $$a \leq a$$
* $$a \leq a$$
* $$a \leq a$$
