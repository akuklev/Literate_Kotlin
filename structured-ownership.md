Capture checking & Structured Ownership
=======================================


# Static capture checking

Singleton classes are the classes created by object declarations `object Obj : T`
and object expressions `object : T {…}` creating anonymous singleton types. By
partially restricting upcasts for values of singleton classes, we can implement
static reference capture checking owing to the fact that a value `x : X` cannot
be exported beyond the scope where it is typable without being upcasted.

## Capabilities

We'll start by introducing a new visibility modifier `restricted` for classes,
interfaces and objects. It makes those types invisible in nested scopes except
as upper bounds for type parameters:
```kotlin
restricted class X {…}

class Y(…) {… X is not not visible here }
```

It can happen that we have a variable of a restrained type:
```kotlin
restricted data class X(val n : Int)
val x = X(1)

class Y(…) {… Here, x : Any, x.n is inaccessible }
```

We can explicitly pass the type `X` to regain the _capability_ access members of `x`.

```kotlin
restricted data class X(val n : Int)
val x = X(1)

class Y<S : X>(…) {… we can use (x as S).n}

val y = Y<X>(args) // Here we pass the original X as S
```

Now let me use another names so you can see the point:
```kotlin
restricted object System {… lots of methods for IO}

fun foo() {… here, System : Any, no methods can be used }
fun <S : System> bar() {… here we can use (System as S) to access all the methods}
```

This last case deserves syntactic sugar that allows to simply write `System` instead
of `(System as S)`:
```kotlin
fun <:System> main() {
    System.out.println("Hello world!")
}

l.filter fun<:Logger> { Logger.trace(it); it > 0}
```

This way we reuse the extant type parameter system to provide syntax and semantics for capabilities.


## Static capture checking via anonymous singleton classes

Whenever you use an object expression `object : T {…}` to create an anonymous object,
you also create an anonymous class. This circumstance can be used for static reference
capture checking.

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
