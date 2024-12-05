Capture checking & Structured Ownership
=======================================


# Static capture checking

Singleton classes are the classes created by object declarations `object Obj : T` and
object expressions `object : T {…}` creating anonymous singleton types. Singleton 
references are identifiers _declared_ to have singleton types. By partially restricting
upcasts for values of singleton classes, we can implement static capture checking owing
to the fact that a value `x : X` cannot be exported beyond the scope where it is typable
without being upcasted.

Let us introduce a new inheritance modifier `object` for interfaces and classes:
```kotlin
object interface Logger {…}
```

With this modifier, `Logger` cannot be used in type casts ~~`( as Logger)`~~,
and in declarations of arguments, variables, fields, and properties ~~`x : Logger`~~.
An `object class` is an abstract class sharing the same restrictions.

Object classes and interfaces can be used create objects `object MainLogger : Logger {…}`,
inherited, and used as upper bounds for static type parameters. They can be subtypes of non-object
interfaces and classes, but all their subtypes have to be object interfaces/classes, except for 
singleton classes.

In particular, in scope of the declaration `object MainLogger : Logger {…}` it is possible to
define a singleton reference `val l : MainLogger = MainLogger`. However, for named objects
there is no reason to do so as there can be only object of the type `MainLogger` and we
already can refer it as `MainLogger`.

For anonymous objects inherited from object interfaces or classes it is not possible to define
`val o = object : Logger {…}` since `val o : Logger` is forbidden. We can either upcast the
result into a non-singleton reference, e.g. `val o : Any = object Logger {…}`,
or pass it into a generic function `foo(object : Logger {…})`, where
```kotlin
fun <L : Logger> foo(l : L) {…}
```

This way `L` will be instantiated to the anonymous singleton type of the object just produced,
and making `l` a singleton reference. However, the ist type `L` will be bound as a static parameter
of `foo<L>` preventing it from capturing `l` or otherwise propagating it outside the scope as
a singleton reference. This approach gives a great control over singleton references.

Let us introduce a bit of syntactic sugar:
```kotlin
fun foo(object L : Logger) === fun <L : Logger> foo(L : L)
```

If we want to “return” an object from a function, we can use a callback instead
```kotlin
fun bar(…, block : (object Logger)->T) : T
...

bar(args) fun(object L : Logger) {
  .. here we have and L : L
}

To spare indentation, we can also introduce notation similar to `using` in C#
object L : Logger = bar(args)
... // the rest of the scope is turned into a callback
```

Note that with this approach it is not possible to store sovereign references inside collections. This is not a shortcomming, but a feature: in those cases we'll have to use managed references provided
by object existence scopes such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc. 
This generalizes Kotlin's Structured Concurrency to Structured Ownership.

---

```kotlin
fun foo<L : &Logger>() === fun <L : Logger> foo(L : L),
with restriction that L can be used only in type parameters and for path-dependent-types L.MemberType
```






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

l.filter fun<:SystemLogger> { SystemLogger.trace(it); it > 0}
```

This way we reuse the extant type parameter system to provide syntax and semantics for capabilities.



```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```

# Scopes and managed references
