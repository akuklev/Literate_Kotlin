We have a dream of making Kotlin a programming language suitable for all purposes in any context. Unfortunately, Kotlin in its current form is poorly suited for literate programming and lags far behind Python when it comes to illustrating ideas in tutorials and research papers. In this memo, we draft a Kotlin variant for literate programming and academic/educational use instead of ad hoc pseudocode.

When writing a computer science research paper or an educational tutorial, it's fine to spend days polishing code snippets for optimal readability, conciseness, and typographic perfection. Such applications value readability over writability, expressiveness over simplicity, principled considerations over practical concerns, and the avoidance of boilerplate and visual clutter at almost any cost. This seems to contradict one of the cornerstones of Kotlin: a remarkable balance between readability and writability, expressiveness and simplicity, orderliness and pragmatism, innovation and conservatism. But it turns out that the necessary changes, while fairly radical, are limited to syntax and default behavior. Literate Kotlin, the variant of Kotlin presented in this memo, can be seen as an alternative interface to the same underlying language.

The first two sections of the memo are devoted to syntax and appearance. The third section suggests some adjustments to the default behavior. In the last part, we discuss potential extensions pertaining to semantics that we believe will also benefit Kotlin itself in the long run.

* Objects, capabilities, and structured ownership
* Maintainable type providers for Kotlin
* Literate Kotlin
* Declarative Kotlin
* Academic Kotlin

# Basic syntax and appearance
In 1984, Donald Knuth introduced literate programming, a practice of working not just on the source code but on a well-written and well-structured expository paper from which the source code can be extracted. The ultimate result should be the expository paper, which carefully walks through all the nooks and crannies of the source code, explaining the ideas, and documenting the reasoning behind certain decisions. It is, at the same time, both an essay interspersed with code snippets and a source code interleaved by accompanying text.

Existing programming languages treat the accompanying text as a second-class citizen, as 'comments' bashfully fenced with freakish digraphs like `/* … */`. Markup languages used for writing computer science research papers (e.g. (La)TeX) and tutorials (e.g. HTML and Markdown) take the opposite approach, treating code snippets as second-class citizens. We propose a balanced approach treating code and text on a par. Before presenting it, however, we need to explain our approach to blocks and literals.

## Blocks
We propose restricting the use of braces only for inline blocks and using the off-side rule for multiline blocks. The indentation-based structure sticks out above everything else, so it should take precedence over comments, quoted literals, and brackets. **This approach massively speeds up incremental parsing: blocks can be recognized instantly, without prior parsing, and processed independently.** 

We propose to fix the block indentation to two whitespaces once and for all, any other indent (1 or >2) continues the previous line:

```kotlin
fun example(files : List<File>,
            target : File)
  ...
  return something + somethingElse + somethingOther +
   yetSomething + rest
```

IDEs should provide visual reading aid for consequent dedents by displaying end marks (`■`):
```kotlin
fun main(args : List<String>)
  for (arg in args)
    println(arg)
  ■
```

At the end of large indentation regions, labeled end marks (e.g. `■ main`) should be used.

## Unquoted literals
In Kotlin, trailing functional arguments enjoy special syntax: `a.map({ println(it) })` is simply `a.map { println(it) }`.
Trailing `String` arguments (in general, `AdditionalContext.()-> String<INTERPOLATION_STYLE>`) deserve special syntax too. We suggest unquoted literals should begin with a left-flanking `~` followed by a whitespace or a line break. They end by the next line of indentation level less or equal to that of the line the literal starts. Line breaks can be `\`-escaped, `\{…}`-syntax used for type-based (e.g. `String<SQL>`) JSR 430-like safe interpolation.

```kotlin
fun greet(name : String)
  println~ Hello, \{name}!
```

This approach also works nicely with property lists:

```kotlin
address: Address
  house:~
    Olaf Taanensen 
    Tordenskiolds 24
  city:~ Oslo
```

## Comments
Our proposal from the first section implies mandatory indentation for all non-inline blocks. Thus, all remaining unindented lines are top-level definitions (`class …`, `object …`, …) and directives (`package …`, `import …`). These necessarily begin with an annotation or a keyword. Annotations readily begin with an `@`, and it won't be too much pain to prepend `@` to top-level keywords: `@import` already looks familiar from CSS, `@data class` and `@sealed class` make perfect sense anyway, as most modifier keywords are nothing but inbuilt annotations.

In this way, every code line either starts with an `@`, or is an indented line following a code line (with possibly one or more blank lines in between). Let us require the compiler to skim all the lines that do not meet this specification. These other lines can now be used for the accompanying text written “as is” without fencing. We suggest using a flavour of LaTeX-hybrid Markdown (`\usepackage[smartHybrid]{markdown}`): it has excellent readability while providing all of the power of (La)TeX, the golden standard for writing technical and scientific papers.

Freely interleaving the code and accompanying text, without fencing, is the perfect fit for literate programming. The very same file can be either fed into a Kotlin compiler to produce a binary or into a Markdown/TeX processor to produce an expository paper.

Sometimes, it is still desirable to comment on a single line. Since at least 1958, em-dashes ` — ` surrounded by whitespaces have been used for single-line comments to separate code and text. It seems to be a typographically perfect solution, but the standard PC keyboard layout lacks em-dash. Ada, Agda, Eiffel, Elm, Haskell, Lua, SQL, and several other languages use double dash `--` as an ASCII substitute for em-dashes, but this is incompatible with the C-style decrement operator. We think it is OK to allow non-ASCII
characters as long as they have ASCII synonyms. Since the standard Mac OS keyboard layout
and some others do include em dashes, we suggest using them, with mandatory whitespaces around, as the signle-line comment marker. A single backtick can be used as
its 'ASCII-synonym': mandatory whitespaces disambiguate from any other valid
usages in Kotlin.

## Commenting out

To ‘comment out’ parts of code we suggest using nestable `{-  -}`-notation that
denotes deletions in Markdown. Such brakets can be used either inline or on separate lines of matching indentation.

```kotlin
foo(arg1{-, arg2 -})
{-
bar(...)
...
-}
baz(...)
```

## Kotlin flavoured markdown
The default hybrid mode of the TeX `\usepackage{markdown}` package was not entirely satisfactory for our purposes, so we developed replacements for several of its options: `smartHybrid` (refines `hybrid`), `fencedEnvs` (refines `fencedDivs`), and `offsideCode` (refines `fencedCode`).

**`fencedEnvs`** is an improvement of the `fencedDivs` option that works as follows:

```
:::boxed           ⟩      \begin{boxed}
Some _text_.       ⟩        Some \emph{text}.
:::                ⟩      \end{boxed}
```

For envs that come in numbered and unnumbered variants (e.g. `theorem`, `figure`, `table`), titlecase name is used for the numbered variant and lowercase for the unnumbered one.

**`smartHybrid`** is an improvement of the `hybrid` option: just like `hybrid` allows TeX commands in Markdown, but uses the percent sign (e.g. `%newpage`) as sigil instead of `\` backslashes to avoid collisions with `\`-escaping in Markdown. Indented blocks starting with `%EnvName` are translated into LaTeX environments with their content preserved verbatim.

```
:::Figure[hb] Sample Caption            ⟩ \begin{figure}[hb]\caption{Sample Caption}
%tikzpicture                            ⟩ \begin{tikzpicture}
  \draw[gray, thick] (-1,2) -- (2,-4);  ⟩   \draw[gray, thick] (-1,2) -- (2,-4);
  \draw[gray, thick] (-1,-1) -- (2,2);  ⟩   \draw[gray, thick] (-1,-1) -- (2,2);
  \filldraw[black] (0,0) circle (2pt);  ⟩   \filldraw[black] (0,0) circle (2pt);
:::                                     ⟩ \end{tikzpicture}\end{figure}
```
Usage of percent signs does not cause problems, as they are used in TeX only for comments.

**`offsideCode`** recognizes indented blocks starting with `@keyword` (e.g. `@class List<T>`) as code blocks. That's how we implement document generation for Literate Kotlin sources! Just typeset them with a preamble containing `\usepackage[offsideCode,…]{markdown}`.

We have not yet implemented a Literate Kotlin → HTML processor, but we intend to translate
LaTeX-commands and environments into HTML tags `<figure parameters="hb"><caption>Sample caption</caption><tikzpicture data="..."/></figure>` that can be then rendered by any of the respective frameworks like Vue.js, Riot.js, etc. It would not be possible to match TeX in typographical perfection because no web browser engine supports or even plans to support proper hyphenation, microtypography, [tab stops](https://www.dickimaw-books.com/latex/thesis/html/tabbing.html), etc. in foreseeable future. Instead of all that HTML provides interactivity. Eventually we hope to develop a documentation generator that turns Literate Kotlin into interactive online documentation with interactive code snippets, similar to <https://kotlinlang.org/docs/kotlin-tour-hello-world.html>.

## Plain text notebooks
Jupyter-style notebooks can be seen as an interactive form of literate programming. The expository paper can and should contain runnable code samples to illustrate usages of the code being explained and test cases for each non-trivial function. These should be optimally displayed as runnable, editable, debbugable blocks with rich (visual, animated, interactive) output, that's what notebooks are build from. Since we see such blocks as an element of literate programming, we want to provide plain text syntax for them:
```kotlin
@run sampleFunction(1, 3)

@run 1 + 2 + 3
@expect 6

@run `Named sample`:
  val a = 1 + 2
  a + 3

@run(collapsed: true, autoexec: false)
  someLenghtyComputation()
```

# Syntactic and typographic sugar

## Pipeline notation
In mathematics and functional programming, it's fairly common to use the right pointing black triangle for inverse application, i.e. `x ▸ foo ▸ bar ≔ bar(foo(x))`, which gives
an intuitive pipeline notation. We suggest displaying `x.let f` as `x ▸ f` and `x?.let f` as `x ▸? f`. Whitespaces are mandatory to disambiguate from the syntax we propose in the next paragraph.

In contrast to purely functional languages, pipelines in Kotlin primarily consist of method invocations. In Kotlin, `obj.foo(…)` can mean both invocation of the method `foo` and application of the property `foo` of a callable type. Following the long tradition started by PL/I in the late 60s, we propose to display dots `▸` when invoking methods. It helps disambiguating between properties and methods, and leads to typographically perfect pipeline syntax:

```kotlin
fun example(files : List<File>,
            target : File)
  files ▸filter
    it.size > 0 &&
    it.type = "image/png"
  ▸map { it.name }
  ▸withIndex ▸fold(0) fun(acc, item) 
    ...
  ■
```

Notably, moving the safe call question mark to the right allows displaying `…OrNull` methods as `…?`, e.g. `▸first?` instead of `.firstOrNull`, `a[i]?` instead of `a.getOrNull(i)`, etc.

## Ad hoc infix operators
Pipeline notation provides an aesthetically pleasing way to act on one object, but sometimes several objects have to be fused, which is
best expressed by infix operators. We propose turning any binary (or vararg) function into an infix operator with chevrons (not `<`angular brackets`>`!): 
```
       a ‹and› b         2 ‹Nat.plus› 3         users ‹join(::id)› customers
```

Ad hoc infix operators make a perfect complement to the pipeline notation.

## Reducing type annotations
Many functional languages allow one to declare multiple consecutive variables of the same type separating them by whitespaces
```kotlin
fun plus(x y : Int) : Int
```
and declaring name-based default type conventions module- or package-wide:
```kotlin
reserve z : Point, prefix n : Int, suffix count : Int
```
In scope of this declaration, identifier `z` with optional numeric indices (e.g. `z2`) will have the default type `Point`, and all multipart identifiers with the first part `n` or the last part `count` (e.g. `nUsers` and `pointCount`, but not `neighbour` or `account`) the default type `Int`. Generalized form of reserve blocks, [pioneered by Agda](http://agda.readthedocs.io/en/v2.7.0/language/generalization-of-declared-variables.html), allows significant reduction of polymorphic signatures.

## Compliance with functional notation
In Kotlin, the method invocation `method(args)` is a complex notation. It supports optional arguments, named arguments, a variable number of tail arguments, and syntactic sugar for the last argument of functional type. It even allows omitting parentheses altogether while invocation still is implied. To disambiguate, methods cannot be referred to simply by their name, and the notation `::method` (or `class::method`) has to be used instead.

Application of callables (i.e. values of type `(args)-> R`) mimics method invocation with the exception that parentheses are mandatory and several subtle limitations. This approach
contradicts the usual mathematical practice, where it is customary to write `sin x` instead of `sin(x)` and `f a b` for `( f(a) )(b)`. We propose to use opt-in `import FunctionalNotation` adding the type former `X -> Y` (without parens around `X`) to introduce functions like `sin` that can be used as customary in mathematics and functional programming languages.

## Compliance with mathematical notation
To improve readability, reduce ambiguities, and comply with established mathematical notation, we require mandatory whitespaces around all infix operators and relations including `n : Int`, but excluding `a·b`, `a..b`, and `a..<b`.

Multiplication should be displayed as `·`, comparison operators as `≤`, `≥`, `=`, `≠`, logical operators as `¬`, `∧`, `∨`, arrow in function literals as `{ x ↦ x + 1 }`, the assignment operator as `≔` when introducing a fresh name (e.g. `val a ≔ 5`), and by left-flanking colon `key: value` otherwise.

Custom symbolic operators are a pandora's box for programming languages: once you allow
them, library designers would use them to introduce unintelligible language dialects. Yet, they are unavoidable for academic applications. As a measure against abuse, we propose
to require importing all symbolic operators manually (no `import lib.*`), while their pronouncible names (like `not` for `¬`) are imported automatically. To do so we'll need
to allow symbolic references for operators. In Kotlin, operators are always referred to by their verbatim name. In mathematics, it is customary to allow symbolic references. We propose the following notation:
```
 ::(-) for ::minus        ::(- ) for ::unaryMinu        ::( --) for ::dec
```
Whitespaces on the right or left mark prefix or postfix operators respectively.

Additionally, we propose two opt-in features:
- `import CoefficientNotation` (used in algebra) to interpret `2x` for `2·x`
- `import SegmentsNotation` (used in geometry) to interpret runs of uppercase letters, possibly with indices, (`ABC`, `ABCD`, `X1X2`) as `Segments(A, B, C)`, `Segments(A, B, C, D)`, `Segments(X1, X2)`. Uppercase identifiers would still be available with backticks (`` `ABC` ``).

## Dual naming: verbose names and concise names
Naming things is hard both in programming and in mathematics. Objects and operations should have readable and self-explanatory names. However, verbose names may severely impair readability in formulas. Compare the following three variants of the same formula:
- `div(times(elementCount, plus(elementCount, 1)), 2)`,
- `elementCount * (elementCount + 1) / 2`, and
- `n·(n + 1) / 2`

Dual naming `` `verbose name`conciseName `` is a way to reconcile these contradictory requirements.

```kotlin
val `element count`n ≔ ...
val (`height`x, `width`y) ≔ o.getDimensions()
class List<`element type`T>
```

## Unicode abbrevations and custom operators
We propose using dual naming schema for definition of custom operators and fancy symbols. In that case, verbose tell how to read operators aloud and are used to provide ASCII synonyms to allow entering fancy symbols using standard keyboard layout.

```kotlin
enum class `Boolean`𝔹 {`true`, `false`}

data class `Pair`(×)<out X, out Y>(val first : X, val second : Y)

val `factorial`( !) ≔ fun(n : ℕ)
  when(n) { 0 ↦ 1; p⁺ ↦ n · p! }

val `conjugate`(+ ) ≔ fun(c : ℂ)
  Complex(c.re, -c.im)
```

Now we can use 𝔹 for `Boolean`, `X × Y` for `Pair<X, Y>`, `n!` for `factorial(n)`, `+c` for `conjugate(c)`.

Alternatively, if the concise name is a plain latin alphanumeric identifier, verbose name is allowed to contain special characters and placeholders:
```kotlin
fun <T> `if $c then $a else $b`ifelse(a b : T, c : 𝔹) : T

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


## Let blocks
We suggest introducing let-blocks. Let-block header contains a list of vals being defined, the following block contains a list of conditions those have to satisfy.  

```
let x y : Float
  x + 2y = 5
  x - y = 4
```

A let-block compiles if there is a compiler solver-plugin that supports given condition forms and succeeds if and only if there is a unique or a preferred solution.

We envision at least two solvers: Linear solver precisely as in Knuth's METAPOST (in particular, solves the example above) and, in the distant future, a deep unification solver as defined in [The Verse Calculus paper](https://simon.peytonjones.org/assets/pdfs/verse-icfp23.pdf) by Simon Peyton Jones, Guy Steele et al., that possesses enormous expressive power, elegantly subsuming both Prolog and Datalog.


# Default behavior
Having the most expressive, readable, intuitive, and aesthetically pleasing syntax is not enough to make an appealing replacement for “pseudocode”, as long as the language excibits perplexing behavior only justified by backwards compatibility with quirks and hacks in earlier languages.

We suggest that the following changes to the default behavior of Kotlin might be both required for literate use and be reasonablty simple to introduce.

Pythonic integers
: Pseudocode assumes the default integer type `Int` to be overflow-free as in Python, while fixed-width 'integers' are denoted by `Int8` to `Int64`. As in Python, `(/)` should denote the proper division regardless of operand types; integer division requires a distinct operator `(//)`.

Operator attribution
: Expressions such as `2 + 3` should be interpreted as `Int.plus(2, 3)` rather than `2.plus(3)`, i.e. arithmetic operators should be considered properties of companion objects rather than methods of values themselves.

# Perspective semantic developments

## Type classes
As we mentioned, operators on values, such as `(+)` and `(·)`, belong to
their types' companion objects. To provide types for the companion objects themselves, we need type classes. Type classes can be seen as parametrized abstract classes with additional syntactic sugar.

Consider the following definition of a monoid structure on a type `T`:
```kotlin
data class <T>.Monoid(val compose : (vararg xs : T)-> T)
  val unit ≔ compose()    — Unit is the nullary composition

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
  ... dealing with multiple contexts simultanously

my obj ≔ SimpleOrder(...)         — strictly local objects
val ob ≔ build SimpleOrder(...)   — managed mutable objects, see below
```

## Runtime-introspectable coroutines
We suggest using labeled blocks (`name@ { code }`) in coroutines as runtime-introspectable execution states. If the job `j` is currently running inside of the labled block `EstablishingConnection@`, we want `(j.state is EstablishingConnection)` to hold. The hierarchy of nested blocks in the coroutine should autogenerate a corresponding interface hierarchy.

Those states may also carry additional data that can be used to track progress of the job. We suggest allowing visibility modifiers `public` and `internal` for top-level `var`s and `val`s as well as the ones in labeled blocks and labeled loops:
```kotlin
val j ≔ launch
  ...prepare data
  Moving@ for (i in files.indices)
    public val progress = i / files.size
    fs.move(...)
  ...finalize
 
val u ≔ launch
  ...
  when (val s ≔ j.state)
    Moving ↦ println~ Moving files, \{s.progress · 100}% complete
  ...
```

Invoking `j.state` must create an instant snapshot of those properties; all properties must be data-only, i.e. of primitive or purely algebraic data type.

## Structured ownership and capability tracking
**Structured ownership** generalizes _structured concurrency_ with a general notion of managed objects and their existence scopes. Launched coroutines in Kotlin and shared mutable objects in Rust are instances of managed objects; coroutine scopes and lifetimes are their respective existence scopes. The behavior of _managed objects_ is governed by the rules of separation logic specific to their respective existence scopes^[Quantum fields in Physics can be seen as existence scopes of their field quanta (“quantum particles”) governed by rules of non-commutative separation logic describing creation, measurement, and anihilation operators.]. References to managed objects are best understood
as objects whose types that are [path-dependent](https://docs.scala-lang.org/scala3/book/types-dependent-function.html) on their respective existence scopes. Their proper handling also requires passing objects (coroutine scopes, lifetimes, etc.) as capabilities:
```kotlin
fun <cs : CoroutineScope> example(b : cs.Ref<Int>)
```

The path dependency prevents exposing references into contexts lacking their existence scope. If the above function had received `cs` as an argument, it could have stored `b` along with `cs`, making the type `cs.Ref` accessible, but passing it as a compile-time parameter prevents this.

Shared resources can be seen as objects that can be passed exclusively as reified capabilites:
```
class Logger<reified fs: FileSystem>()
  fun log(s : String) ≔ ...write to a log file, using `fs`
```
This way the filesystem captured by the logger has to be mentioned in its type `Logger<fs>`.
By denoting closures capturing `c,…` as `T^<c,…>` we recover [the capability system from Scala 3](https://docs.scala-lang.org/scala3/reference/experimental/cc.html).

# Conclusion and outlook
In this memo, we have outlined the vision and rationale behind Literate Kotlin, a variant of Kotlin tailored for literate programming and academic use. By addressing the limitations of Kotlin in its current form, we aim to bridge the gap between the language's inherent strengths and the specific needs of educational and research contexts.

At first glance, our suggestions may seem like a wild potpourri, as they incorporate features required to suit the needs of a broad and diverse groups within the potential audience. Nevertheless, the suggested changes are the result of 20 years of exploration, thinking and trying, evaluating UX feedback, and trying again. It started as a one-man project, continued for several years as a joint effort with [Alexander Temerev](https://www.linkedin.com/in/temerev), who made a significant contribution to the ideas presented here, and finally became part of the author's activities at JetBrains Research.

While being radical, our proposals are superficial and for the most part easy to implement. We believe that by improving readability, expressiveness, and typographic quality according to our suggestions, Literate Kotlin can serve as a powerful tool for educators, researchers, and anyone who values clarity and precision in code presentation.

The adjustments to syntax and appearance, along with the suggested behavioral modifications and semantic extensions, are designed to make Literate Kotlin a viable alternative for those who currently rely on pseudocode or other languages for illustrative purposes. We are confident that these enhancements will not only benefit the academic community, but also contribute to the broader Kotlin ecosystem by promoting a more versatile and expressive language.

The early drafts of this memo were enthusiastically received at the Department of Software Science at Radboud University Nijmegen, the Department of Informatics at Göttingen University, and the Department of Mathematics at TU Dresden. As we move forward, we invite the academic community to engage with Literate Kotlin, provide feedback, and contribute to its evolution. Together, we can realize the dream of making Kotlin a truly universal programming language.
