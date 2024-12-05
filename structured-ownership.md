# Structured Ownership

## Using anonymous classes to controll references

Whenever you use an object expression `object O : T {}` to create an anonymous object, you also create an anonymous class.
This circumstance can be used to control reference propagation and capture in compile time.
As a byproduct we will also be able to recreate a capability tracking system almost exactly as in proposed in Scala 3.

First let us introduce a new inheritance modifier `object` for interfaces and classes.
An `object interface Oi` has the property that it is forbidden to ever declare variables
or arguments of the type `Oi`, or to use type casts `( as Oi)`.
An `object class` is an abstract class sharing the same property.
They only can be used to create objects `object O : Oi {...}` and
as inheritance upper bounds for other interfaces/classes and in type parameters.

All descendants of object interfaces and object classes have to be object interfaces/classes
themselves, except for exact types of objects inherited from them, for instance if you declare
`object O : Oi {...}` is valid to write `val o : O = O`. Other than for object expressions
that create anonymous objects, there is no reason to do so as there is only one object of the
type `O` and we already can refer to it with `O`. Both variables `o : O` and `O : O` will be
from now on called sovereign references.

Now consider the following pecularity regarding anonymous objects inherited from object interfaces:
We cannot write `val o = obiect Oi {...}` since `val o : Oi` forbidden.
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
