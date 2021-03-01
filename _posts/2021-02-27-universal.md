---
layout: post
title:  "Does this function accept itself?"
date:   2021-02-17 19:45:00 +0000
categories: decidability
---


<!-- This paragraph is confusing. Rewrite it. -->
In the [previous post][regex], I showed why it's impossible to implement a function that tells whether another function recognizes a regular expression. And the short answer was: because it would imply that another thing is possible: to tell, given a function and a string, whether the function accepts the string. I ended the post mentioning that the latter is also impossible. In this post, I will explain why.

As in the previous post, we are only considering predicates (functions from `String` to `Boolean`). Let's look at one of the examples again:

```scala
object zerosAtTheEdges extends (String => Boolean) {
  override def apply(s: String): Boolean = s.matches("01*0")
}
```

And let's assume that a universal decider exists and we can simply import it from a library. Some usage examples:

```scala
scala> universalDecider(zerosAtTheEdges, "0110")
val res0: Boolean = true

scala> universalDecider(zerosAtTheEdges, "11")
val res1: Boolean = false

scala> universalDecider(zerosAtTheEdges, "a random string")
val res2: Boolean = false
```

## Function psychology

Now that we have our universal decider, we can use it as a building block to solve larger problems. Remember that all our functions map strings to booleans. Given that any function can be represented by a string (after all, this what source code is), it is a legal move to pass the source code of a function to any function. More interestingly, we can pass the source code of a function to itself! For example, consider this piece of code:

```scala
val serialized = """object zerosAtTheEdges extends (String => Boolean) {
  override def apply(s: String): Boolean = s.matches("01*0")
}"""

println(zerosAtTheEdges(serialized))
```

We are passing the source code of `zerosAtTheEdges` to `zerosAtTheEdges` and printing the result. Evidently, this will print "false", because a piece of Scala code does not match the regular expression "01*0". Now suppose we wrote a function (in Scala) that validates Scala code. What would happen? The source code of the validator is a valid piece of Scala code. The validator, by definition, accepts any valid Scala code as input. Therefore, unlike `zerosAtTheEdges`, the validator is a function that accepts its own source code as input.

So the problem we are interested in is to tell how a function would react when faced with its own source code; will it accept that string or not? So what we want is to implement a function with this signature:

```scala
def isSelfLoathing(sourceCode: String): Boolean = ???
```

This is what we want from `isSelfLoathing`: you pass some source code to it. The function that this source code compiles to may or may not accept its own source code as input. If it does not, we will say that the function loathes itself. It can't bear to look at its own image in the mirror and may return false or go into a spiral of existential angst and loop forever, never to return to the society of functions again! In that case, `isSelfLoathing` gives the right diagnosis: true (i.e., the fuction loathes itself). If, on the other hand, the function has no self-image problems and accepts its own source code as input, `isSelfLoathing` returns false. This is the contract we expect the function to satisfy.

## Implementation

Now that we have the contract for the function nailed down, it's time to implement it. Before that, let's just define two auxiliary functions, without bothering with their implementations:

```scala
def compile(sourceCode: String): (String => Boolean) = ???

def serialize(fn: String => Boolean): String = ???
```

`compile`, as the name suggests, takes a string that contains some source code and compiles that code to produce a function. `serialize` does the opposite: given a function, it generates its corresponding source code.

This is what it looks like:

```scala
// Assuming sourceCode is a well formed piece of Scala code
def isSelfLoathing(sourceCode: String): Boolean =
  !universalDecider(compile(sourceCode), sourceCode)
```

Pretty straightforward. We compile the source code, pass the compiled and the source to the universal decider and return the negation of that. And we can verify that this works with some unit tests:

```scala
class SelfLoathingTest extends AnyFunSuite {
  test("Function does not accept itself") {
    assert(isSelfLoathing(serialize(zerosAtTheEdges)))
  }

  test("Function accepts itself") {
    assert(!isSelfLoathing(serialize(scalaValidator)))
  }
}
```

This is all well and good, but now the QA engineer looked at our code and said: "hang on a minute!" (as QAs usually do). "What about this edge case?"

```scala
isSelfLoathing(serialize(isSelfLoathing))
```

## Contract revisited

It's not self-evident what we should expect in the edge case in which `isSelfLoathing` is diagnosing itself. In order to reason more systematically about this, we need to formalize the contract we have described only informally so far. One way to do this is to create what mathematicians call an _axiomatic system_, that is, a set of base rules, which we take for granted (the _axioms_), and derive logical consequences from there.


**Axiom 1.**
```scala
isSelfLoathing(code) == true

// if, and only if:

compile(code)(code) == ⊥
```

For conciseness, I'm abusing the Scala notation a bit here. The symbol `⊥` denotes that the function either returns false or loops forever. What this axiom is saying is that `isSelfLoathing` should return true exactly when the function does not accept its own code as input. This is precisely the formalization we needed. But we still need a second axiom.


**Axiom 2.**
```scala
compile(serialize(fn))(x) == fn(x) // for any fn and x
```

In other words, if you take any function `fn`, serialize it and compile it again, you will obtain another function that behaves exactly like `fn` for any input you pass to both of them. In the mathematical jargon, each of these functions are said to be the _inverse_ of the other.


## Addressing the edge case

Back to the edge case, there are only two options, of course: true and false. Let's assume it should be true. Going back to our contract and using Axiom 1, we get that:

```scala
isSelfLoathing(serialize(isSelfLoathing)) == true // (1)
```

implies that:

```scala
compile(serialize(isSelfLoathing))(serialize(isSelfLoathing)) == ⊥
```

This looks complicated, but it's nothing more than a purely mechanical substitution of symbols. Wherever we see `code` in Axiom 1, we are replacing it with `serialize(isSelfLoathing)` here. The axiom is generic and applies to everything that you could reasonably regard as `code`. This edge case is just a particular application of it.

By Axiom 2, we know that `compile` is the inverse of `serialize`. As such, they cancel each other out, allowing us to simplify the equation above to:

```scala
isSelfLoathing(serialize(isSelfLoathing)) == ⊥ // (2)
```

This is the heart of the matter: by starting from the equation (1) and reasoning our way through, we arrived at equation (2), which is the negation of (1). That is, if we assume that equation (1) is true, we conclude that equation (1) is false! And you can easily see that, if we start out with equation (2) instead, we'll fall into the same trap. Let me restate this: logically, there are only two possible assertions we can make:

```scala
assert(isSelfLoathing(serialize(isSelfLoathing)))
assert(!isSelfLoathing(serialize(isSelfLoathing)))
```

And both are nonsensical. In either case, if the test passes, we are actually asserting the opposite of what it says. There is no escape.

## Conclusion

In the fist post of this series, we started from the problem of deciding whether a given function accepts a regular expression. We didn't know how to implement that, or even whether it was even possible. But we just assumed it was possible and started using it as if it already existed. We wished it into existence. This is sometimes called "programming by intention". 

One thing we found out was that we could use that function as a component of a larger one. The larger function, which we called a "universal decider", would receive two parameters (another function and a string) and decide whether the given function accepts the given string. Then we repeated the process and used the universal decider inside an even larger function: one that receives another function and gives a verdict about whether it accepts itself as input. This process seemed natural, progressing by small and comprehensible steps, but one that eventually led us to a dead end.

Disappointing as it is, the dead-end is the crucial information here. It tells us that the whole path we've taken is invalid (and, consequently, so are all the functions we've built along the way). This is how many proofs of undecidability work in computer science. If you can show that one problem leads to another in way we did here (the first problem is said to be _reducible_ to the second), and the second problem is undecidable, then the problem you start with is also undecidable.


[regex]: https://otaviomacedo.github.io/decidability/2020/12/11/regex.html