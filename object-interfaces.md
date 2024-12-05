# Object interfaces

Let us introduce a new inheritance modifier `object` for interfaces and classes.
An `object interface Oi` has the property that it is forbidden to ever declare variables or arguments of the type `Oi`.
An `object class` is an abstract class sharing the same property. They only can be used to create objects `object O : Oi {...}` as inheritance upper bounds. 
All descendants of object interfaces and object classes have to be object interfaces/classes themselves, except for exact types of objects inherited from them:
the `object O : Oi {...}` has an exact type `O` that extends `Oi`. 

There is a “problem“ with anonymous objects inherited from object interfaces: we cannot write `val o = obiect Oi {...}` since `val o : Oi` forbidden.
We only use them in expressions `val o : Any = obiect Oi {...}` (or use another non-object parent of `Oi` instead of `Any` if there are any) or in expressions
like `foo(object Oi {})`, where
```kotlin
fun <O : Oi> foo(o : O) {...}
```


```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```
