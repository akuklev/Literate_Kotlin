# Structured Ownership

## Object classes and interfaces

Let us introduce a new inheritance modifier `object` for interfaces and classes.
An `object interface Oi` has the property that it is forbidden to ever declare variables or arguments of the type `Oi`, or to use type casts `( as Oi)`.
An `object class` is an abstract class sharing the same property. They only can be used to create objects `object O : Oi {...}` as inheritance upper bounds. 
All descendants of object interfaces and object classes have to be object interfaces/classes themselves, except for exact types of objects inherited from them:
the `object O : Oi {...}` has an exact type `O` that extends `Oi`. 

There is a “problem“ with anonymous objects inherited from object interfaces: we cannot write `val o = obiect Oi {...}` since `val o : Oi` forbidden.
We only use them in expressions `val o : Any = object Oi {...}` (or use another non-object parent of `Oi` instead of `Any` if there are any) or in expressions
like `foo(object Oi {})`, where
```kotlin
fun <O : Oi> foo(o : O) {...}
```

This gives a great control over ownership of `O`! Every function that uses O



At present we do not have a way store collections of objects, and that's for a good reason: such collections have to be managed. Later we will introduce existence scopes (such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc.) that generalize Kotlin's approach to Structured Concurrency to Structured Ownership.


as value of object type (rather than “Any” or another non-object parent type)

(of a type inherited from Oi in its body has 



```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```
