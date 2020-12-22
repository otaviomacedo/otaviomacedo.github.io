---
layout: post
title:  "Semilattices in Tic-Tc-Toe"
date:   2020-12-19 06:40:00 +0000
categories: algebra
---
{% include mathjax.html %} 

<style type="text/css">
.center-image
{
    margin: 0 auto;
    display: block;
}
.wide { width: 100%; height: auto; }
</style>

Introduction bla bla bla

## Program design

The program consists of three main abstractions: the Player, which, as the name suggests, represents the players of the game, with two concrete implementations: User and Computer. User captures input from a human player while Computer implements the minimax algorithm to decide the best move. Board represents the state of the game at any given point (which marks are placed on which cells). And Game is responsible for controlling the flow of the game from beginning to end, alternately giving the turn to each player. 

![](/assets/tictactoe/class-diagram.svg){: .center-image }

In good functional programming style, everything is immutable, so Game doesn't mutate the boards but, instead, unravels a game history, i.e., produces a sequence of states in which each state is a different instance of Board. This sequence ends in a full board or in a state in which one of the players has won. So it's important to tell, of a given Board, whether there is a winner and who it is. Let's say the game reached this state:

![](/assets/tictactoe/board.png){: .center-image }

In this configuration, player X has completed the third row with its mark, so this is a final state in the game's history and X is the winner. 


![](/assets/tictactoe/reductions.svg){: .center-image .wide}

The transformation represented by the blue arrows generates the winner of each line (row, column or diagonal), that is, the symbol that appears three times in the line (or $$\Large␣$$ if there is no such symbol). The transformation represented by the orange arrows generates the first non-empty symbol found in the sequence; otherwise, it generates $$\Large␣$$. 

A possible implementation of this process in Scala is:

```scala
lazy val outcome: GameOutcome = {
  def product(a: GameOutcome, b: GameOutcome): GameOutcome = 
    if (a == b) a else None

  def winner(positions: Seq[Int]): GameOutcome = 
    positions map cells reduce product

  (rows ++ columns ++ diagonals) map winner find(_.isDefined) getOrElse None
}
```
<!-- Briefly talk about the types: GameOutcome, Symbol -->

The function `product` corresponds to the blue transformation in our diagram and `find(_.isDefined) getOrElse None`, in the last line, corresponds to the orange one. This is the intuitive notion we have about what should happen and more or less matches the verbal description I've given for it. This is perfectly fine, but there is a deeper connection between these two transformations that is lost in this implementation.

Observe how we use `reduce` to go from a sequence of symbols to a single symbol in the blue transformation. It would be satisfyingly symmetrical if we could reframe the orange one in term of another reduction. So, what would that function look like? With a bit of thought and experimentation, we can quickly arrive at the solution. But I would like to take a detour through the land of Abstract Algebra and Order Theory.

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

  (rows ++ columns ++ diagonals) map winner reduce maximal
}
```

<!-- TODO Make this statement stronger and more interesting -->
Of course, if this was a real world application, I wouldn't do this refactoring. The previous version communicates intent much better, even if we lose some symmetry. But this example shows just how often abstract algebraic structures like semilattices appear everywehere.

