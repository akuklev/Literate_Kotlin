# Structured Ownership

## Object classes and interfaces

Let us introduce a new inheritance modifier `object` for interfaces and classes.
An `object interface Oi` has the property that it is forbidden to ever declare variables or arguments of the type `Oi`, or to use type casts `( as Oi)`.
An `object class` is an abstract class sharing the same property. They only can be used to create objects `object O : Oi {...}` as inheritance upper bounds. 
All descendants of object interfaces and object classes have to be object interfaces/classes themselves, except for exact types of objects inherited from them:
the `object O : Oi {...}` has an exact type `O` that extends `Oi`. In a scope where `object O : Oi {...}` is defined, we it is valid to write `val o : O = O`, 
but there is no reason to do so as there is only one object of the type `O` and we already can refer to it with `O`. Both variables `o : O` and `O : O` will be
from now on called sovereign references.

Now consider the following pecularity regarding anonymous objects inherited from object interfaces:
We cannot write `val o = obiect Oi {...}` since `val o : Oi` forbidden. We only use anonymous objects in expressions like `val o : Any = object Oi {...}`
(or use another non-object parent of `Oi` instead of `Any` if there are any) producing non-sovereign references to `O` or in expressions like `foo(object Oi {})`, where
```kotlin
fun <O : Oi> foo(o : O) {...}
```

This gives a great control over sovereign references to `O`! Indeed, every function that uses a sovereign reference to `O` and any object that captures a sovereign reference to `O`, must directly or inderectly obtain `<O>` as a compile-time type parameter.

(Here example with file handle that cannot be exposed)

Note that with this approach it is not possible to store sovereign references inside collections. This is not a shortcomming, but a feature: in those cases we'll have to use managed references provided
by object existence scopes such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc. 
This generalizes Kotlin's Structured Concurrency to Structured Ownership.

## Capabilities

```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```
