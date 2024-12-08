Declarative programming is an approach striving to enable describing what you want to get rather than how to get it. It facilitates concise, transparent, straight-to-the-point code and is an indispensable tool for tackling inherent complexity. Here, we outline how to introduce declarative programming capabilities into Literage Kotlin, a language with great support for declarative `DSL`s including an `SQL`-like reactive query language. Full-scale implementation of our proposals brings the full power of functional logic programming (roughly “Haskell + Prolog”), while already limited support extends the query language with an accessible form of Datalog.

# Implicit definitions \label{let-blocks}

Implicit definitions enable declarative programming whenever objectives can be described by conditions. They are ubiquitous in mathematical texts, so supporting the widest possible class of them is highly desirable for a language used in academia. We propose the following notation:
```kotlin
|\textbf{\textcolor{dgreen}{let}}| x y : Float               |\textbf{\textcolor{dgreen}{let}}| gcd : Int                 |\textbf{\textcolor{dgreen}{try let}}| x ?t : Float
  x + 2y = 5                    n \% gcd = 0                   x = a + b·t
  x - y = 4                     m \% gcd = 0                   x = c + d·t
                                maximizing { gcd }
```

A `let` block contains conditions imposed on the indeterminates declared in its header. Conditions must uniquely determine the values of the indeterminates except for so-called existential variables (marked like `?t`), which are scoped within the block and not exposed. 
A `let` block can only be compiled if there is an appropriate solver for conditions of the given form on indeterminates of given types.
Solvers have to ensure the existence of a unique solution^[Take `a = c, b = d = 0` in the rightmost example. Its solution `x = c` qualifies as unique because `t` is existential.], either at compile-time (`let` blocks) or at run-time (`try let` blocks). At present, we envision three specialized solvers:
- the [*-semiring linear equation solver](https://r6.ca/blog/20110808T035622Z.html),
- the [mixed integer and real linear arithmetic solver](https://doi.org/10.1007/978-3-030-55754-6_14),
- an SAT/SMT (boolean satisfiability/satisfiability modulo theories) solver.

# Indeterminate functions \label{indeterminate-computations}

The expressivity of conditions, not only in `let` blocks but also in ordinary conditionals and filters, can be greatly improved with indeterminate functions. While ordinary functions compute _the_ value for given arguments, indeterminate functions describe what's _a_ value for given arguments:
```kotlin
data class Person(val father : Person, val mother : Person)
```
```kotlin
|\textbf{\textcolor{dgreen}{def a}}| Person.parent = anyOf(this.mother, this.father)              # Just 2 values
|\textbf{\textcolor{dgreen}{def an}}| Person.ancestor = anyOf(this.parent, this.ancestor.parent)  # Recursion!
```
```kotlin
persons ▸filter { it.ancestor = x }
```

We define what's _a_ `.parent` and _an_ `.ancestor` of a person to `▸filter` those with _an_ ancestor `x`. Such code is much leaner than any imperative alternative, leaving no place for a bug to hide.

A query `f : query (Xs)-> Y` either computes a value and returns it together with a deferred query for alternative values, or recognizes lack of value(s). A general indeterminate computable function `f : quest (Xs)-> Y` is a query that may fail to return at all. In both cases, `f` is not allowed to have side effects and can only be invoked inside other queries/indeterminate functions, or converted into potentially infinite streams `all{f} : (Xs)-> Sequence<Y>`.

# Indeterminate implicit definitions \label{verse-calculus}

[Verse calculus](https://simon.peytonjones.org/assets/pdfs/verse-icfp23.pdf), a novel approach based on introspectable indeterminate functions, provides unrestricted `let` blocks with arbitrary conditions expressible in terms of computable functions.

This extends applications of `let` blocks far beyond academic purposes, into the realm of complex real-world applications, where the inherent complexity of problems and systems leads to intricate, fragile, and error-prone code, unless systematically managed declaratively.

Inside indeterminate functions, existence and uniqueness restrictions for `let` blocks can be lifted, and solvers are merely required to produce an effectively exhaustive^[For every semi-decidable predicate `P` that holds for some solution, there must be at least one `x`$_n$ satisfying `P`. This condition reduces to exhaustiveness for enumerable types and types with semi-decidable equality.] stream of solutions^[The type `quest T` of indeterminate `T`-valued computations is an instance of the `?T` modality in linear logic.] `x`$_i$. Given a computable `f : (Int)-> Int`, indeterminate functions can employ `let` blocks like this:
```kotlin
|\textbf{\textcolor{dgreen}{let}}| x : Int
  f(x) = c
```
It is valid because one can brute-force a stream of solutions by successively applying `f` to all possible integers (0, ±1, ±2,..) until the result turns out to be equal to the constant `c`. One could have used multiple variables and conditions, an indeterminate `f`, any predicate instead of `(= c)` as long as it is expressible by a computable function (i.e. semi-decidable), and any type `X` instead of `Int` as long as it admits an effectively exhaustive sequence `x`$_i$ `: X` (i.e. “surveyable”)^[A condition satisfied by all enumerable and many relevant non-enumerable types like $\mathbb{R}$.].

The prime purpose of declarative programming is to provide self-evidently correct reference definitions, regardless of their efficiency, but brute-forcing is far too inefficient to ever be feasible in practice. Fortunately, Verse calculus offers an ingenious approach that turns out to be reasonably effective. In the case of queries applied to finitary data such as databases and data streams, there is a whole arsenal of highly efficient Datalog query optimizations in addition.

# Conclusion and outlook \label{conclusion}

By leveraging declarative programming, we can substantially improve the expressivity of Kotlin and extend its capabilities to a whole new level. These extensions make it a superior choice not only for literate programming and illustrating ideas in teaching and research, but also for a variety of real-world applications where the taming of inherent complexity is urgently needed.

Indeterminate functions in queries virtually eliminate the need for subqueries and dramatically reduce the code in size and complexity, making it easier to understand and less error-prone. Functional logic programming provides a tractable approach to dealing with vast combinatorial complexity, e.g. in conflict resolution for interacting software systems and constraint satisfaction.

When it comes to academic and educational use, implicit definitions help to keep code as close to the text as possible, or to provide straightforward equivalents for intricate algorithms so as to elucidate all subtleties and catch all the bugs while establishing the equivalence^[For instance, `gcd` as defined in §<#let-blocks> is an equivalent for the non-trivial Euclidean algorithm (except `n = m = 0`):
\textbf{\textcolor{dgreen}{\texttt{def}}}` gcd(n m : Nat) = if(m = 0) n else gcd(m, n \% m)`].

While being remarkably useful, both pure functional programming (à la Haskell) and logic programming (à la Prolog) have a reputation for being arcane academic gimmicks. The recently developed Verse calculus has finally made it possible to combine them into functional logic programming, which is generally assumed to be even less capable of gaining broad adoption in the foreseeable future. Our approach aims to wield the full power of functional logic programming without expecting the users to have any understanding of the underlying calculus. To this end, we discreetly introduce functional logic programming into a mainstream programming language using merely two non-intrusive, reader-friendly, and self-explanatory constructs.




