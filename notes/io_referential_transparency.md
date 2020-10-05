# 引用透明

---

## 什么是函数式编程

- High Order Function
- Immutable data
  - 没有变量，只有常量，任何一个“变量”一旦被赋值，就不能再次被赋值
  - 不可变数据结构，数据一旦被创建出来，就不能被修改了
- Pure function

---

## 什么是纯函数

> 一个函数的输出只依赖于这个函数的输入，并没有副作用

---

### 这是不是纯函数

```
val sum = function(a, b) {
  console.log(`$a + $b = ` + (a + b))
  a + b
}
```

---

### 引用透明(referential transparency)

```scala
f(x) = x/2
y = f(1) // Result always is 0.5
```

---

> 任意函数，或者任意代码段，如果它可以被它的计算结果直接替代，仍然不影响任何调用它的程序，那么这个函数或代码段就是引用透明.

---

### Why referential transparency

- 更好的被推理(Readable)
- 更好的被测试(Testable)

---

### Question: Are they referential transparency

```scala
val divide: Int => Double = (a) => {
  if(a == 0) throw new Error("Can't divide 0")
  else a/2
}
```

---

## Eliminated NullPointerException and try catch in scala

---

### Option / Either

```
def readInt1: Either[Error, Int] = Right(1)
def readInt2: Either[Error, Int] = Right(2)

for {
  a <- readInt1
  b <- readInt2
} yield (a + b)
```

---

# IO ???

---

```
case class Player(name: String, score: Int)

def contest(p1: Player, p2: Player): Unit = {
  if(p1.score > p2.score)
    println(s"${p1.name} is winner")
  else if(p1.score < p2.score)
    println(s"${p2.name} is winner")
  else
    println(It's a draw)
}
```

---

### After refactor

```
def winner(p1: Player, p2: Player): Option[Player] = {
  if(p1.score > p2.score) Some(p1)
  else if(p2.score > p1.score) Some(p2)
  else None
}

def contest(p1, p2): Unit = winner(p1, p2) match {
 case Some(p) => println(s"${p.name} is winner")
 case None => println("It's a draw")
}
```

---

### More refactor

```
def winnerMsg(p: Option[Player]): String =
  p.map(l => s"${p.player} is a winner").getOrElse("it's a draw")
```

---

### So ====>

一个非纯函数 `A => B`, 可以拆分成一个纯函数 `A => D` 和一个非纯函数 `D => B`; 就是将副作用推到最外层

---

### New abstraction for IO

```
trait IO { def run: Unit }
def PrintLn(msg: String): IO = {
  new IO { def run = println(msg) }
}

def contest(p1: Player, p2: Player): IO = PrintLn(winnerMsg(winner(p1, p2)))

contest(p1, p2).run
```

---

### 如何用IO 连续执行两次?

```
println(10)
println(10)
```

---

### Evolution of IO monad(new编程范式)

```
sealed trait IO[A] { self =>
  def run: A
  def map[B](f: A => B): IO[B] = new IO { def run = f(self.run) }
  def flatMap[B](f: A => IO[B]): IO[B] = new IO[B] {
    def run = f(self.run).run
  }
}
object IO {
  def apply[A](f: =>A) = new IO[A] { def run = f }
}
```

---

```
val readLine: IO[String] = IO { Stdin.readLine }
def putLine(msg: String): IO[Unit] = IO { println(msg) }

val program = for {
  a <- readLine
  b <- readLine
  _ <- putLine(a)
  _ <- putLine(b)
} yield ()

program.run
```

---

```
def forever[A, B](a: IO[A]): IO[B] = {
  lazy val t: IO[B] = forever(a)
  a.flatMap(_ => t)
}

forever(putLine("hello world")) // stack overflow
def flatMap[B](f: A => IO[B]): IO[B] = new IO[B] {
  def run = f(self.run).run
}
```

---

```scala
sealed trait IO[A] {
  def flatMap[B](f: A => IO[B]): IO[B] = FlatMap(this, f)
  def map[B](f: A => B): IO[B] = flatMap(f andThen (Return(_)))
}

case class Return[A](a: A) extends IO[A]
case class Suspend[A](resume: () => A) extends IO[A]
case class FlatMap[A, B](sub: IO[A], k: A => IO[B]) extends IO[B]

def println(s: String): IO[Unit] =
  Suspend(() => Return(println(s)))
```
