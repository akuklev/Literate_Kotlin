Structured Ownership
====================


# Objects, References and Capabilities

Singleton classes are the classes created by object declarations `object Obj : T` and
object expressions `object : T {…}`. Using refined approach to singleton classes, it
is possible to recover the capability checking system as proposed for Scala 3, and to
control references in a manner similar to ??.

## Capture checking

Normally, inner scopes inherit all identifiers from their parent scopes.
Let us introduce a new visibility modifier `restrained` to prevent this behavior.
```kotlin
restrained class X : T {…}
restrained val n = 1

class Y(…) {… X and n are not visible here }
```

If neccessary, we can pass them explicitly:
```kotlin
restrained class X : T {…}
restrained val n = 1

class Y<X : T>(n : Int, …) {… some type X and some value x are visible here }

val y = Y<X>(n, args) // Here we pass the right X and n
```



---

Consider the following code:
```kotlin
val l : List<Int> = ...

l.filter { it > n }    // Error: n is not visible inside
```

Restrained visibility can be also applied to classes:
```kotlin
restrained class X : T {…}
class Y(…) {… Obj not visible here }

val y = Y(args)
```

We can explicitly propagate X into Y:
```kotlin
restrained class X : T {…}
restrained val x = 1

class Y(…) {… X and x are not visible here }

val y = Y(args)
```

```kotlin
  restrained object Obj : T {…}
  class Cls {/* Obj is invisible here */}
  fun foo() {/*  here  */}
  l.filter {/* or here */}
```

Let us introduce the following notation to explicitly propagate types into inner scopes:
```kotlin
restrained object Obj : T {…}
class Cls<&Obj> {/* Only the type Obj is visible here */}
fun <&Obj> foo() {/*  here  */}
l.filter fun<&Obj> {/* and here */}```
```

A `restrained val name = val` is visible only inside its immediate scope, but not in the downstream scopes; it has to be passed as a parameter manifestly^[Restrained fields, properties inside classes as well as nested and inner classes do not make those completely invisible, just force to use `this::Outer.` to access them, or to use fully qualified names for nested classes.].

We want this visibility modifier to be also available objects both inside functions and classes, and on the top level. 
```
restrained object Filesystem
```



## Using singleton classes to control references


Whenever you use an object expression `object : T {…}` to create an anonymous object,
you also create an anonymous class. This circumstance can be used to control reference
propagation and capture in compile time.

First let us introduce a new inheritance modifier `object` for interfaces and classes.
An `object interface Oi` cannot be used in type casts ~~`( as Oi)`~~, and as a type in
declarations of argument, variables, fields, and properties ~~`x : Oi`~~. An `object
class` is an abstract class sharing same restrictions.

Object classes and interfaces can used create objects `object Obj : Oi {…}`, inherited,
and used as upper bounds for type parameters. Object classes and interfaces can themselves
extend non-object interfaces and classes, but all their descendants have to be object
interfaces/classes, except for singleton classes created by object declarations
`object Obj : Oi {…}` and anonymous classes created by object expressions `object : Oi {…}`.

In particular, in scope of the declaration `object Obj : Oi {...}` is possible to
define `val o : Obj = Obj`. However, for named objects there is no reason to do so
as there is only object of the type `Obj` and we already can refer to it with `O`.
Both `o : Obj` and `Obj : Obj` will befrom now on called sovereign references.

Now consider the following pecularity regarding anonymous objects inherited from object interfaces:
We cannot write `val o = obiect : Oi {...}` since `val o : Oi` forbidden.
We only use anonymous objects in expressions like `val o : Any = object Oi {...}`
(or use another non-object parent of `Oi` instead of `Any` if there are any)
producing non-sovereign references to `O` or in expressions like `foo(object Oi {})`, where
```kotlin
fun <O : Oi> foo(o : O) {...}
```

This gives a great control over sovereign references to `O`! Indeed, every function that uses a sovereign reference to `O` and any object that captures a sovereign reference to `O`, must directly or inderectly obtain `<O>` as a compile-time type parameter.

(Here example with file handle that cannot be exposed)

Note that with this approach it is not possible to store sovereign references inside collections. This is not a shortcomming, but a feature: in those cases we'll have to use managed references provided
by object existence scopes such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc. 
This generalizes Kotlin's Structured Concurrency to Structured Ownership.

## Tracking Capabilities

Let us introduce a new visibility modifier `restrained`. A `restrained val name = val` is visible only inside its immediate scope, but not in the downstream scopes; it has to be passed as a parameter manifestly^[Restrained fields, properties inside classes as well as nested and inner classes do not make those completely invisible, just force to use `this::Outer.` to access them, or to use fully qualified names for nested classes.].

We want this visibility modifier to be also available objects both inside functions and classes, and on the top level. 
```
restrained object Filesystem
```

```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```

# Scopes and managed references
