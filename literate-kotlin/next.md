





* * *
`import SegmentsNotation` (used in geometry) to interpret runs of uppercase letters, possibly with indices, (`ABC`, `ABCD`, `X1X2`) as `Segments(A, B, C)`, `Segments(A, B, C, D)`, `Segments(X1, X2)`. Uppercase identifiers would still be available with backticks (`` `ABC` ``).

# Advanced syntactical elements

## Let blocks
We suggest introducing let-blocks. Let-block header contains a list of vals being defined, the following block contains a list of conditions those have to satisfy.  

```
let x y : Float
  x + 2y = 5
  x - y = 4
```

A let-block compiles if there is a compiler solver-plugin that supports given condition forms and succeeds if and only if there is a unique or a preferred solution.

We envision at least two solvers: Linear solver precisely as in Knuth's METAPOST (in particular, solves the example above) and, in the distant future, a deep unification solver as defined in [The Verse Calculus paper](https://simon.peytonjones.org/assets/pdfs/verse-icfp23.pdf) by Simon Peyton Jones, Guy Steele et al., that possesses enormous expressive power, elegantly subsuming both Prolog and Datalog.

## Pre-declaration blocks
Many functional languages allow one to declare multiple consecutive variables of the same type separating them by whitespaces
```kotlin
fun plus(x y : Int) : Int
```
and declaring name-based default type conventions module- or package-wide:
```kotlin
reserve z : Point, prefix n : Int, suffix count : Int
```
In scope of this declaration, identifier `z` with optional numeric indices (e.g. `z2`) will have the default type `Point`, and all multipart identifiers with the first part `n` or the last part `count` (e.g. `nUsers` and `pointCount`, but not `neighbour` or `account`) the default type `Int`. Generalized form of reserve blocks, [pioneered by Agda](http://agda.readthedocs.io/en/v2.7.0/language/generalization-of-declared-variables.html), allows significant reduction of polymorphic signatures.

## Reconciling method invocation and function applicative
In Kotlin, the method invocation `method(args)` is a complex notation. It supports optional arguments, named arguments, a variable number of tail arguments, and syntactic sugar for the last argument of functional type. It even allows omitting parentheses altogether while invocation still is implied. To disambiguate, methods cannot be referred to simply by their name, and the notation `::method` (or `class::method`) has to be used instead.

Application of callables (i.e. values of type `(args)-> R`) mimics method invocation with the exception that parentheses are mandatory and several subtle limitations. This approach
contradicts the usual mathematical practice, where it is customary to write `sin x` instead of `sin(x)` and `f a b` for `( f(a) )(b)`. We propose to use opt-in `import FunctionalNotation` adding the type former `X -> Y` (without parens around `X`) to introduce functions like `sin` that can be used as customary in mathematics and functional programming languages.

## Symbolic references

Custom symbolic operators are a pandora's box for programming languages: once you allow
them, library designers would use them to introduce unintelligible language dialects. Yet, they are unavoidable for academic applications. As a measure against abuse, we propose
to require importing all symbolic operators manually (no `import lib.*`), while their pronouncible names (like `not` for `¬`) are imported automatically. To do so we'll need
to allow symbolic references for operators. In Kotlin, operators are always referred to by their verbatim name. In mathematics, it is customary to allow symbolic references. We propose the following notation:
```
 ::(-) for ::minus        ::(- ) for ::unaryMinu        ::( --) for ::dec
```
Whitespaces on the right or left mark prefix or postfix operators respectively.

## Dual naming: verbose names and concise names
Naming things is hard both in programming and in mathematics. Objects and operations should have readable and self-explanatory names. However, verbose names may severely impair readability in formulas. Compare the following three variants of the same formula:
- `div(times(elementCount, plus(elementCount, 1)), 2)`,
- `elementCount * (elementCount + 1) / 2`, and
- `n·(n + 1) / 2`

Dual naming `` `verbose name`conciseName `` is a way to reconcile these contradictory requirements.

```kotlin
val `element count`n = ...
val (`height`x, `width`y) = o.getDimensions()
class List<`element type`T>
```

## Unicode abbrevations and custom operators
We propose using dual naming schema for definition of custom operators and fancy symbols. In that case, verbose tell how to read operators aloud and are used to provide ASCII synonyms to allow entering fancy symbols using standard keyboard layout.

```kotlin
enum class `Boolean`|$\mathbb{B}$| {`true`, `false`}

data class `Pair`(×)<out X, out Y>(val first : X, val second : Y)

val `factorial`( !) = fun(n : ℕ)
  when(n) { 0 ↦ 1; p⁺ ↦ n · p! }

val `conjugate`(+ ) = fun(c : ℂ)
  Complex(c.re, -c.im)
```

Now we can use $\mathbb{B}$ for `Boolean`, `X × Y` for `Pair<X, Y>`, `n!` for `factorial(n)`, `+c` for `conjugate(c)`.

Alternatively, if the concise name is a plain latin alphanumeric identifier, verbose name is allowed to contain special characters and placeholders:
```kotlin
fun <T> `if $c then $a else $b`ifelse(a b : T, c : |$\mathbb{B}$|) : T

fun `⌊$x⌋`floor(x : Float)
```

### Operator tightness
Expressions like `+n!` can be parsed both as `( +n )!` and `+( n! )`. With definitions as above, it is not a valid expression, it's a `syntax error: ambiguous expression`. However, one can specify the tightness for the operators. If `( !)` binds tighter than `(+ )`, `+n!` resolves into `+(n!)` and the other way around.

Infix operators may have different right and left tightness. For example, `(-)` binds tighter on the right than on the left: `a - b - c` resolves into `(a - b) - c`.

To specify tightness, we allow introducing abstract tightness levels called Operator Categories and allow declaring them to be tighter or weaker than some other levels. They must merely form a directed acyclic graph and do not have to be pairwise comparable.

In fact, an OperatorCategory is more than a mere label: it specifies how to deal with respective homogeneous operator chains. For example, `EqRel` is a large operator category that contains comparison operators and resolves their chains `a < b < c`  into `(a < b ‹and› b < c)`.

### Operators with parameters
Operators may have parameters, e.g. the indexed access operator `arr[i]` is a postfix operator with a parameter `( [$idx])` . In mathematics, many binary operators, including tensor product and semidirect product, have optional parameters rendered as subscripts or superscripts.

Using parser techniques developed for the Agda programming language, we can embrace this complexity without considerable diffiulties.

By combining custom `OperatorCategory` and operators with inner parameters, one can even embrace the notorious example of insane operator complexity: the METAPOST path notation:
```kotlin
draw a -- b -- c --cycle              — A triangle, (--)-lines are straight
draw a ~~ b ~~ c ~~cycle              — A circle through abc, (~~)-lines are curved
draw a ~~ b ~~ c ~- d -- e --cycle    — (~-) connect smoothly only on the left side

draw a ~~ b ~~[tension: 1.5, 1]~~ c ~~ d
draw a [curl: k]~~ c ~~[curl: k] d
draw a ~~ b [up]~~ c [left]~~ d ~~ e.
draw (0,0) ~~[controls: (26.8,-1.8), (51.4,14.6)]~~
 (60,40) ~~[controls: (67.1,61.0), (59.8,84.6)]~~ (30,50)
```





# Default behavior

Operator attribution
: Expressions such as `2 + 3` should be interpreted as `Int.plus(2, 3)` rather than `2.plus(3)`, i.e. arithmetic operators should be considered properties of companion objects rather than methods of values themselves.

# Perspective semantic developments

## Type classes
As we mentioned, operators on values, such as `(+)` and `(·)`, belong to
their types' companion objects. To provide types for the companion objects themselves, we need type classes. Type classes can be seen as parametrized abstract classes with additional syntactic sugar.

Consider the following definition of a monoid structure on a type `T`:
```kotlin
data class <T>.Monoid(val compose : (vararg xs : T)-> T)
  val unit = compose()    — Unit is the nullary composition

  contracts {
    unit ‹compose› x = x
    x ‹compose› unit = x
    x ‹compose› y ‹compose› z = x ‹compose› (y ‹compose› z)
    compose(x, *xs) = x ‹compose› compose(*xs)
  }
```

With such a definition, we now can write polymorphic functions like this:
```kotlin
fun <T : Monoid> square(x : T)
  x ‹T.compose› x
```
Here, in addition to the generic type `T`, one has its eponymous companion
object `T : <T>.Monoid`.  
With a dedicated syntax it is possible to import the composition operator directly:
```kotlin    
fun <T : Monoid(::(∘))> square(x : T)
  x ∘ x
```

Companion objects of polymorphic types (e.g. `List<T>`) have higher kinded type classes:
```kotlin
abstract class <`Container`F<_>>.Functor
  open fun <X, Y> F<X>.map(transform : (X)-> Y) : F<Y>
```  

Support for higher kinds and type class inheritance can be modeled directly after [Arend](https://arend-lang.github.io/).

## Dependent types and refinement types
Eventually, one should carefully introduce dependent types, following the defensive approach pioneered in Haskell, i.e. without destroying the phase distinction (between compile-time and run-time) and turning the whole language into a theorem prover.

Combining of such Kotlin features as type-safe builders and flow typing, with custom operators and dependent types, allows for DSLs of unprecedented sophistication. For instance, dependent types immediately allow embedding SQL-type queries almost verbatim:^[We also propose using ‘single `'word` string literals’ as they don't conflict with character literals `' '`.]
```kotlin
fun Table.select(cols : this.colsCtx.()-> List<t.Col>) : LazyTable
fun LazyTable.where(clause : this.ctx.()-> Boolean) : LazyTable

users ▸select { name, age, address -> 'userAddress } 
      ▸where { age > 18 }
```

Analogously, one should carefully introduce refinement types: types with logical predicates that allow to enforce important properties at compile time, as in [Liquid Haskell](https://ucsd-progsys.github.io/liquidhaskell/), [Rust](https://github.com/flux-rs/flux) and [Scala](https://github.com/fthomas/refined).

## Stateful builders
Why don't we combine two prominent Kotlin features: type-safe builders and flow-typing?
Invoking a builder method should be allowed to change the list of allowed methods:
```
interface Order.Unfinished : Order
  @NextState(Order.InTransit)
  fun process(x : PaymentProof) : OrderId 
  
interface Order.InTransit : Order
  @NextState(Order.Delivered)
  fun markAsDelivered() 

interface Order.Delivered : Order {}

...
order1.use
  markAsDelivered    — error, this method is not yet available!
  process(prf)       - ok!
  process(prf)       — error, the method `process` is not available anymore!
  markAsDelivered    — ok!
```

It is not in general possible to allow all objects to have stateful types due to aliasing: a third party can invoke a method that changes the object type. But we can introduce special syntax for non-externlizable unique references for three cases where they are
appropriate:
```kotlin
try(service1.use as h1, service2.use as h2)
  ... dealing with multiple contexts simultaneously

my obj = SimpleOrder(...)         — strictly local objects
val ob = build SimpleOrder(...)   — managed mutable objects, see below
```

## Runtime-introspectable coroutines
We suggest using labeled blocks (`name@ { code }`) in coroutines as runtime-introspectable execution states. If the job `j` is currently running inside of the labeled block `EstablishingConnection@`, we want `(j.state is EstablishingConnection)` to hold. The hierarchy of nested blocks in the coroutine should autogenerate a corresponding interface hierarchy.

Those states may also carry additional data that can be used to track the progress of the job. We suggest allowing visibility modifiers `public` and `internal` for top-level `var`s and `val`s as well as the ones in labeled blocks and labeled loops:
```kotlin
val j = launch
  ...prepare data
  Moving@ for (i in files.indices)
    public val progress = i / files.size
    fs.move(...)
  ...finalize
 
val u = launch
  ...
  when (val s = j.state)
    Moving ↦ println~ Moving files, \{s.progress · 100}% complete
  ...
```

Invoking `j.state` must create an instant snapshot of those properties; all properties must be data-only, i.e. of primitive or purely algebraic data type.

## Structured ownership and capability tracking
**Structured ownership** generalizes _structured concurrency_ with a general notion of managed objects and their existence scopes. Launched coroutines in Kotlin and shared mutable objects in Rust are instances of managed objects; coroutine scopes and lifetimes are their respective existence scopes. The behavior of _managed objects_ is governed by the rules of separation logic specific to their respective existence scopes^[Quantum fields in Physics can be seen as existence scopes of their field quanta (“quantum particles”) governed by rules of non-commutative separation logic describing creation, measurement, and anihilation operators.]. References to managed objects are best understood
as objects whose types that are [path-dependent](https://docs.scala-lang.org/scala3/book/types-dependent-function.html) on their respective existence scopes. Their proper handling also requires passing objects (coroutine scopes, lifetimes, etc.) as capabilities:
```kotlin
fun <cs : &CoroutineScope> example(b : cs.Ref<Int>)
```

The path dependency prevents exposing references into contexts lacking their existence scope. If the above function had received `cs` as an argument, it could have stored `b` along with `cs`, making the type `cs.Ref` accessible, but passing it as a compile-time parameter prevents this.

Shared resources can be seen as objects that can be passed exclusively as reified capabilites:
```
class Logger<reified fs: &FileSystem>()
  fun log(s : String) = ...write to a log file, using `fs`
```
This way the filesystem captured by the logger has to be mentioned in its type `Logger<fs>`.
By denoting closures capturing `c,…` as `T^<c,…>` we recover the [capability system from Scala 3](https://docs.scala-lang.org/scala3/reference/experimental/cc.html).

# Conclusion and outlook
In this memo, we have outlined the vision and rationale behind Literate Kotlin, a variant of Kotlin tailored for literate programming and academic use. By addressing the limitations of Kotlin in its current form, we aim to bridge the gap between the language's inherent strengths and the specific needs of educational and research contexts.

At first glance, our suggestions may seem like a wild potpourri, as they incorporate features required to suit the needs of a broad and diverse groups within the potential audience. Nevertheless, the suggested changes are the result of 20 years of exploration, thinking and trying, evaluating UX feedback, and trying again. It started as a one-man project, continued for several years as a joint effort with [Alexander Temerev](https://www.linkedin.com/in/temerev), who made a significant contribution to the ideas presented here, and finally became part of the author's activities at JetBrains Research.

While being radical, our proposals are superficial and for the most part easy to implement. We believe that by improving readability, expressiveness, and typographic quality according to our suggestions, Literate Kotlin can serve as a powerful tool for educators, researchers, and anyone who values clarity and precision in code presentation.

The adjustments to syntax and appearance, along with the suggested behavioral modifications and semantic extensions, are designed to make Literate Kotlin a viable alternative for those who currently rely on pseudocode or other languages for illustrative purposes. We are confident that these enhancements will not only benefit the academic community, but also contribute to the broader Kotlin ecosystem by promoting a more versatile and expressive language.

Drafts of this memo were enthusiastically received at the Department of Software Science at Radboud University Nijmegen and the Department of Informatics at Göttingen University. As we move forward, we invite the academic community to engage with Literate Kotlin, provide feedback, and contribute to its evolution. Together, we can realize the dream of making Kotlin a truly universal programming language.
