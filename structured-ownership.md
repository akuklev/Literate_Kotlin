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

# Syntactic sugar for objects

Let us introduce a special notation for `fun <L : Logger> foo(L : L)`:
```kotlin
fun foo(object L : Logger)
```

We cannot return a singleton reference, but instead we can pass it to a callback
```kotlin
fun bar(…, block : (object Logger)-> T) {…}

// Usage:

bar(args) fun(object L : Logger) {
  .. here we have and L : L
}
```

This situation leads to a callback hell. In Kotlin we already have a solution: suspend functions, that use callbacks under the hood, but look nicer.
Let us introduce object functions:
```kotlin
object fun bar(args) : Logger {...; return object : Logger {…}}

// Usage:
object L : Logger = bar(args)
...
```

# Type-safer builders & exclusive ownership

Object interfaces can be also used to prevent `this` from leaking in type-safe builders by
making the type of contexts for context receivers an object interface:
```kotlin
object interface ListBuffer<E> {…}
inline fun <E> buildList(builder : (object ListBuffer<E>).()-> Unit): List<E>
```

Let us introduce an additional modifier for arguments that only accept objects, types of which are still anonymous
at the call site. This condition guarantees that there are no other singleton references to this object, i.e.
we gain exclusive ownership. I propose using `my` as such modifier:
```kotlin
my f : MutableFile = open(file)
... perform io with f while being sure we have exclusive ownership till the end of scope
```

Consider one of the basic examples of type-safe builders from standard Kotlin documentation:
```kotlin
html {
  head {
    title("Sample page")
  }
  body {
    ...
  }
}
```

Until now we had no way enforcing that body can only be called after head, and neither of them cannot be called twice. Now that we can guarantee contexts to be exclusively owned, we could address this utilizing Kotlin's flow typing by introducing methods that switch the type of their host:
```kotlin
inline fun <E> html(builder : (my EmptyHtmlBuffer).()-> Unit): Html

interface HtmlBuffer {
  fun export() : Html
}

object interface EmptyHtmlBuffer : HtmlBuffer {
  @NextState(HtmlBufferWithHead)
  fun head(f : HeadBuffer()-> Unit)
}

object interface HtmlBufferWithHead : HtmlBuffer {
  @NextState(HtmlBufferWithHeadAndBody)
  fun body(f : BodyBuffer()-> Unit)
}

object interface HtmlBufferWithHeadAndBody : HtmlBuffer {}
```

# Capabilities

To introduce capability tracking, let us start by introducing a new visibility modifier `restricted` for classes,
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

# Scopes and managed references

Using our approach it is not possible to store singleton references inside collections (or, in fact, any containers). This is not a shortcomming, but a feature: in those cases we'll have to use managed references provided by object existence scopes such as `CoroutineScope`s for `Job`s, Rustacean lifetimes for variables, and ultimately also filesystems for files, databases for tables etc. 
This generalizes Kotlin's Structured Concurrency to Structured Ownership.


```kotlin
fun <object X> foo(...)
fun <X : Oi> foo(X : X, ...)

class Foo<object X : T>
class Foo<X : Oi>(val X : X, ...)
```

```kotlin
fun foo<L : &Logger>() === fun <L : Logger> foo(L : L),
with restriction that L can be used only in type parameters and to access L.InnerClasse types.
```

---

# Type providers

A type provider is a function of the following signature:
```kotlin
init object fun H2Db(connString : String) : GeneratedFinalClass<DbBase>
```

Type providers are meant to be used to connect standalone external resources as static objects like this:
```kotlin
restricted object Db = H2Db("jdbc:h2:coffees.h2.db")
```
The object `Db` will be initised on first access, but its type (also called `Db`) has
to be computed in compile-time, and that's precisely what the type provider function
does. `H2DB(connString : String)` will be executed in compile time, and generate
the future type of `Db` including its readable source (to be used for debugging
purposes). It can have side effects, in particular it can connect to the database
in compile-time, retrieve its schema (including its version number) and store it as
in a static field (i.e. as a `const val`) inside the newly generated type. The resulting
type is required to have a default constructor with no arguments, that will take care
of the late initialization (happens on the first access). In our case, the constructor
will probably connect to the database ensuring that its schema still matches the content
of `const val schema` retrieved in the compile time.

Assuming we can have type members as in Scala, we can also implement method
```kotlin
val users = Db.table("users")
```
that returns a value of an anonymous subtype of `Db.Table`, computed using `Db.schema`.

To provide its signature, we'll need the following
```kotlin
abstract class DbBase {
  type Table
  private init object fun Table(name : String) : GeneratedFinalClass<this.Table>
  public fun table(const name : String) : Table(name)
} 
```

Const modifier on an argument means that the argument has to be available in compile-time,
for example so it can be used to compute the type. Type provider methods (the ones declared as `init object fun`)
can be used in contexts where types are expected.

```kotlin
  type View
  private init object fun View(cols : this.ColSpec) : GeneratedFinalClass<this.View>
  public fun select(const cols : this.ColSpec) : View(cols)
```
