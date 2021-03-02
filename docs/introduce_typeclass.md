# Introducing TypeClass

Type classes is a concept from Haskell lang, it introduced powerful polymorphism(多态) to scala. It's not a first-class citizen in the scala language, but other built-in mechanisms allow to writing them in Scala.

In shortly, type class can extends any behavior to any class, lik3e implementing interface in Java, but with the other mechanisms.

## Prepare


### Implicit Parameters

A method can have an implicit parameter list, marked by the implicit keyword at the start of the parameter list.

```scala
def add(implicit a: Int, b: String): Int = a + b.toInt
```

This method can be invoke by passing 2 parameters as usual, like `add(1, "2")`.
If the parameters in that parameter list are not passed as usual, Scala compiler will look if it can get an implicit value of the correct type, and if it can, pass it automatically.

```scala
implicit val intNum = 1
implicit val intString = "2"
add  // add(intNum, intString) the compiler will pass them automatically.
```

> Question: Does the following code snippet can works ?

```
implicit val intNum = 1
val intString = "2"
add
```

#### Find the implicit parameters in context

Scala will **firstly** look for implicit definitions and implicit parameters that can be accessed directly at the point the method with the implicit parameters is called.

So we can either declared implicit parameter in the scope like the example,
or import the implicit value from other package, such as `import cats.implicits._`

#### Find the implicit parameters in companion objects

Then it looks for members marked implicit in all the companion objects associated with the implicit candidate type.

```
object ThreadPool {
  implicit val threadPool = new ThreadPool
}

def useThreadPool(implicit threadPool: ThreadPool) = ???

useThreadPool // it can works
```

Scala compiler will find the threadPool in ThreadPool companion objects. So it will be compiled to `useThreadPool(ThreadPool.useThreadPool)`

#### Implicit parameters with typeparameter

Scala standard lib provide a trait - `Ordering[T]` which is used to order two instances a and b.

```
trait Ordering[T] { def compare(x: T, y: T): Int }
```

So `Ordering` can be used to get the maximum for a List.
```
def maximum[T](list: List[T])(implicit order: Ordering[T]): Option[T] = list match {
  case a :: Nil => Some(a)
  case a :: rest =>
    maximum(rest).map { max =>
      if(order.compare(a, max) == 1) Some(a) else Some(max)
    }
  case Nil => None
}
```

The `maximum` method can be short by syntactic sugar:

```scala
def maximum[T: Ordering](list: List[T]): Option[T] = {
  ...
  implicity[Ordering[T]].compare(a, max)
  ...
}
```

Using `implicity[Ordering[T]]` to get the instance of the implicit parameter.


### Implicit Conversions

How to add a new method - `toCapitalize` to `String` with scala?


Scala provide `implicit` to help to attach new method to a class.
```
implicit class RichString(value: String) {
  def toCapitalize = ???
}
```

When it's called like `"hello".toCapitalize`, and scala compiler find `String` don't have the `toCapitalize` method, scala compiler will transfer the `"hello"` to the `RichString` automatically. So it works like bellow after compiling.

```scala
new RichString("hello").toCapitalize
```

> Question: How to add a method `def show: String` to all class.

## Evelotion to typeclass

Firstly, we defined two classes.
```
case class Animal(name: String)
case class People(name: String)
```

Both people and animal can walk, the code can be like this.

```
implicit class WalkOps[T](obj: T) {
  def walk: String = obj match {
    case Animal(name) => s"Animal $name walk"
    case People(name) => s"People $name walk"
  }
}
Animal("cat").walk
People("bob").walk
```

So what if I create a new class `Car`, and want make it `walk`. So I need open the `WalkOps` to add a new `case` item. So it breach the open close principle.

### Implementation in Java

In tranditional Java, we can create an interface to make all class to has its own implementation.

So code can be like this.
```
trait Walker { def walk: String; }
def startWalk(t: Walker): Unit = println(t.walk)
```

The `t` in `startWalk`, can be `People`, `Animal` and other `Walker`, so we call it's `Polymorphism`.

### Another Polymorphism

Let's continue to do a small refactor for the previous `implicit WalkOps` example.

1. Firstly, we extract an `trait Walker[T]` to fill the `walk` implementation for different `T`.
```
trait Walke[T] { def walk(t: T): String  }
implicit class WalkerOps[T] (t: T){
 def walk(implicit walker: Walker): String = walker.walk(t)
}
```
2. Write implementation for different type.
```
implicit peopleWalkerInstance = new Walker[People] {
  def walk(t: People): String = s"People ${t.name} walking"
}
implicit animalWalkerInstance = new Walker[Animal] {
  def walk(t: Animal): String = s"Animal ${t.name} walking"
}
```
So it won't break the OCP law.
3. Let's use it.
```
People("bob").walk
def startWalk[T](t: T)(implicit walker: Walker[T]) = println(t.walk)
startWalk(Animal("dog"))
startWalk(People("lili"))
```
So `startWalk` can work with different type which has `Walker` instance. So it's kind of Polymorphism.
And the defination `startWalk` can be short by `def startWalk[T: Walk](t: T) = ???`.

This tech we called `TypeClass`.

### 3 factor of typeclass

Let's recap.

Firstly, we define a type class `trait Walke[T] { def walk(t: T): String  }`
Secondly, we create a `XXXOps` which contains some syntax operation.
```
implicit class WalkerOps[T] (t: T){
 def walk(implicit walker: Walker): String = walker.walk(t)
}
```
Finally, we need some instances to make `T.work` works.

## Practice to write typeclass - `Functor`

Let's see the example of `List.map`, `List(1,2,3).map(_.toString)`
We can find the signature of this `map` method is `def map[B](f: T => B): List[B]`.

We can find the same pattern in `Opiton[T]`, `def map[B](f: T => B): Option[B]`. Same as `Either[A, B]`, `def map[C](f: B => C): Either[A, C]`.

But all these classes don't not extends a trait with a method `map`, so how can we use type class to simplify this method.

```
def doubleAndToString[M[_]](v: M[Int]): M[String] = v match {
  case a: Option[Int] => a.map(v => (v * 2).toString)
  case a: List[Int] => a.map(v => (v * 2).toString)
  case a: Either[Throwable, Int] => a.map(v => (v * 2).toString)
}
```

Practise: Using typeclass to abstract the `map` method to type class.

`def doubleAndToString[M[_]: Functor](a: M[Int]): M[String] = a.map(v => (v*2).toString)`

## Heads to cats

[Cats](https://typelevel.org/cats/) is basic typeclass lib for scala. As it describe it introduce the `ad-hoc polymorphism` to scala.

> Where many object-oriented languages leverage subtyping for polymorphic code, functional programming tends towards a combination of parametric polymorphism (think type parameters, like Java generics) and ad-hoc polymorphism.

Cats provide some basic/common usage typeclass, its related syntax method and intances.

* `Functor[M[_]]` for `.map`
* `Monad[M[_]]` for `.flatMap`
* `Monoid[M]` for `.combine`
* `Show[M]` for `.show`
* `Eq[M]` for `.eqv`

### Heads to `Show`

Let's dig into typeclass `Show[M]`.

[Show](https://typelevel.org/cats/typeclasses/show.html)

> Show is an alternative to the Java toString method. It is defined by a single function show

This is the defination of show.
```scala
trait Show[A] {
  def show(a: A): String
}
```

#### Show[Int]

Cats already provide some intance for some instances. Let's try the int with Show.

```scala
import cats.instances.int._  // Firstly import the all the instances provided by cats which include `Show[Int]` instance.
implicity[Show[Int]].show(1) // Since typeclass instance will be defined as implicit.
```

Cats also provide some syntax for show so we can simplify the method by importing the show syntax
```scala
import cats.instances.int._  // import the int instances
import cats.syntax.show._ // import the all syntax for show

1.show  // looks like Int can have the method `show`
```

#### Show[A]

Cats also provide some other instances for other type, such as `Option`, `Either` and `Double`, we can import it one by one.

```scala
import cats.instances.option._
import cats.instances.either._
import cats.instances.bigInt._
```

Also we can import all of the instances provide by cats.  `import cats.instances.all._`

So the same as syntax, `import cats.syntax.all._`

Meanwhile, cats provide a package for us to import all syntax and instances, `import cats.implicits._`

> 面向对象编程就是以多态为手段来对源代码中的依赖关系进行控制的能力, 让高层策略性组件于底层实现性组件相分离, 底层组件可以被编译成插件, 实现独立于高层组件的开发和部署
