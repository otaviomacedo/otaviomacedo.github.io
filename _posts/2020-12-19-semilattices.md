---
layout: post
title:  "Semilattices in Tic-Tc-Toe"
date:   2020-12-19 06:40:00 +0000
categories: algebra
---
{% include mathjax.html %} 

## Program design

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
  def product(a: GameOutcome, b: GameOutcome): GameOutcome = 
    if (a == b) a else None

  def winner(positions: Seq[Int]): GameOutcome = 
    positions map cells reduce product

  val allLines = Vector(rows, columns, diagonals).flatten
  allLines map winner find(_.isDefined) getOrElse None
}
```
<!-- Briefly talk about the types: GameOutcome, Symbol -->

This is perfectly fine. The function `product` corresponds to the blue transformation in our diagram and `find(_.isDefined) getOrElse None` in the last line corresponds to the green one. This is the intuitive notion we have about what should happen and more or less matches the verbal description I've given for it. But there is a deeper connection between these two transformations that is lost in this implementation.

First off, we use `reduce` to go from a sequence of symbols to a single symbol in the blue transformation. It would be satisfyingly symmetrical if we could reframe the green one in term of reduction, as well. So, what would that function look like? With a bit of thought and experimentation, we can quickly arrive at the solution. But I would like to take a detour through the land of Abstract Algebra and Order Theory.

## Semilattices

The function `product` is a lot more interesting than it seems. Before we can take a closer look at it, though, I want to just adopt a different convention for this section, to make the text more readable: I'm replacing the name `product` with the symbol $$\ast$$ and, whenever you see $$a \ast b$$, translate it in your mind to `product(a, b)`.

Ok, so the operation $$\ast$$ has three noteworthy properties:

* __Idempotence__: $$a \ast a = a$$ for any symbol $$a$$. This is the very reason we created this function.
* __Commutativity__: $$a \ast b = b \ast a$$ for any two symbols $$a$$ and $$b$$.
* __Associativity__:  $$(a \ast b) \ast c = a \ast (b \ast c)$$ for any three symbols $$a$$, $$b$$ and $$c$$.

In Abstract Algebra, a set with an operation that satisfies these three properties form a structure called _semilattice_. The interesting thing about a semilattice is that they can be looked at from two different perspectives: as a set with an operation, as we just saw, or as a _partially ordered set_.

To understand what this means, consider all the possible pairs of symbols. There are $$3$$ different symbols, so $$3^2 = 9$$ combinations. Out of these $$9$$ pairs, let's keep only those in which the product of the two elements is equal to the first one. For example, the pair $$(\epsilon, {\normalsize\texttt{o}})$$ satisfies this condition because $$\epsilon \ast {\normalsize\texttt{o}} = \epsilon$$, whereas $$({\normalsize\texttt{o}}, \epsilon)$$ and $$(\times, {\normalsize\texttt{o}})$$ don't. If a given pair $$(a, b)$$ satisfies this condition, we say that $$a \leq b$$.

So, starting from the semilattice operation, we defined what "less than or equal to" means. And, if we know that $$a \leq b$$, and we had to put them in order, $$a$$ would come first. So, the "ordered" part is clear. In what sense, then, is it "partially" ordered? This refers to the fact that some pairs of elements cannot be compared this way. For example, neither $$\times \leq {\normalsize\texttt{o}}$$ nor $${\normalsize\texttt{o}} \leq \times$$ is true.

One of the logical consequences of this particular partial order (we could have a different order had we started with a different operation) is that $$\epsilon \leq a$$ for any symbol $$a$$. And this fact, in turn, implies that $$\times$$ and $${\normalsize\texttt{o}}$$ are not less than or equal to any other symbol. (Note how I carefully avoided saying "greater than".) Elements which are not less than or equal to any other are called _maximal elements_.


## Back to code

Previously we were looking for a defined element in a sequence and, if we didn't find any, we would return `None`. Now we can redefine this rule in terms of finding a maximal element. If there is any defined element in the sequence (either $$\times$$ or $${\normalsize\texttt{o}}$$), it is a maximal element. Any one will do for this application. If no element is defined (i.e., all we have is a bunch of `None`s), then `None` is the maximal.

With this new definition in mind we can refactor the `outcome` function, so that both transformations are implemented as some sort of reduction:

```scala
lazy val outcome: GameOutcome = {
  def product(a: GameOutcome, b: GameOutcome): GameOutcome = 
    if (a == b) a else None

  def maximal(a: GameOutcome, b: GameOutcome): GameOutcome = 
    if (product(a, b) == a) b else a

  def winner(positions: Seq[Int]): GameOutcome = 
    positions map cells reduce product

  val allLines = Vector(rows, columns, diagonals).flatten
  allLines map winner reduce maximal
}
```

<!-- TODO Make this statement stronger and more interesting -->
Of course, if this was a real world application, I wouldn't do this refactoring. The previous version communicates intent much better, even if we lose some symmetry. But this example shows just how often abstract algebraic structures like semilattices appear everywehere.

