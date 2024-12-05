Structured Ownership and Capture Checking
=========================================


# Static capture checking

Singleton classes are the classes created by object declarations `object Obj : T` and
object expressions `object : T {…}` creating anonymous singleton types. Singleton 
references are identifiers _declared_ to have singleton types. By partially restricting
upcasts for values of singleton classes, we can implement static capture checking owing
to the fact that a value `x : X` cannot be exported beyond the scope where it is typable
without being upcasted.

Let us introduce a new inheritance modifier `object` for interfaces and abstract classes:
```kotlin
object interface Logger { fun log(s : String) }
```

With this modifier, `Logger` can be only used to create objects `object MainLogger : Logger {…}`,
extended by other object `object` interfaces and classes, and as an upper bound for static
type parameters, but cannot be used in casts `(x as Logger)` and declarations (`x : Logger`).

Object interfaces and classes can be subtypes of non-object interfaces and classes, but all
their subtypes have to be object interfaces/classes, except for singleton classes.

In scope of the declaration `object MainLogger : Logger {…}` it is possible to define a
singleton reference `val l : MainLogger = MainLogger`, but it is not possible to define
`val l = object : Logger {…}` for an anonymous logger, because the declaration `val l : Logger`
is forbidden for an object interface `Logger`. It is possible to upcast the result into a non-singleton
reference, e.g. `val l : Any = object Logger {…}`, but in this case it is not possible to
invoke `l.log()` as it is not a member of `Any`. It is also impossible to access it via
`(l as Logger).log()` since such casts are also forbidden. The only thing we can do is to
pass it into a generic function `foo(object : Logger {…})`, where
```kotlin
fun <L : Logger> foo(l : L) {... here l.log() is available }
```

This way `L` will be instantiated to the anonymous singleton type of the object just produced,
and making `l` a singleton reference. With `L` being a type parameter, `l` cannot be captured
or leak outside the scope.

# Unique references

Object interfaces can be also used to prevent `this` from leaking in type-safe builders by
making the type of contexts for context receivers an object interface:
```kotlin
object interface ListAccumulator<E> {…}
fun interface ListBuilder<E> { fun <C : ListAccumulator<E>> C.build() }
inline fun <E> buildList(builder : ListBuilder<E>): List<E>

// with syntactic sugar, it can be written

inline fun <E> buildList(builder : (object ListAccumulator<E>).()-> Unit): List<E>
```

Additionaly we want to introduce a modifier that allows instantiating static type parameters
only by anonymous singleton types, which guarantees uniqueness of the respective reference.

```kotlin
fun interface ListBuilder<E> {
  fun <new C : ListAccumulator<E>> C.build()
}
```

: object : Table {generate} 
// body can even have an additional guarantee that its refrence to the respective list builder is unique:
inline fun <E> buildList(body : (private ListBuilder<E>).() -> Unit): List<E>


The private modifier allows to pass only objects types of which are anonymous at the call site, which guarantees
the uniqueness of the reference.

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


A bit of syntactic sugar 
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

to use it, we need a new kind of functions akin to suspend functions:

object fun bar(args) : Logger {... return object : Logger {…}} 
```


```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```


---

```kotlin
fun foo<L : &Logger>() === fun <L : Logger> foo(L : L),
with restriction that L can be used only in type parameters and to access L.InnerClasse types.
```

my L : Logger



Note that with this approach it is not possible to store singleton references inside collections (or, in fact, any containers). This is not a shortcomming, but a feature: in those cases we'll have to use managed references provided by object existence scopes such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc. 
This generalizes Kotlin's Structured Concurrency to Structured Ownership.

# Computed types

```kotlin
class GeneratedClass<Parent>

...
fun init selectT(cols) : GeneratedClass<this.View>
fun select(const cols) : this.View & selectT(cols)
```

# Type providers
Static objects have two initialization phases: static initialization and late initialization 

```
object fun H2Db(const connString : String) {
  return object H2Db {
    const val schema = 
  }
}

```

```kotlin
restricted object DataSource : H2Db = H2Db("jdbc:h2:coffees.h2.db")
  // has a const schema : Schema inside

val users = DataSource.table("users")
  // Checks that schema.version matches the actual version
  // uses schema to compute an anonymous subtype of inner object class DataSource.Table
```

# Scopes and managed references
