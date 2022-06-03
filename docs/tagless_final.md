# Functional Programming

1. High Order Function, 函数是头等变量
1. Immutable data，数据是不可变的(Immutability)
2. Pure function, 一个函数的输出只依赖于这个函数的输入，然后返回输出之外，再也不做其它的事情.(No side effect)

# SOLID 设计原则

# Algebra & Interpretation & ZIO

# F[Int]

# What's effect

# What's abstraction

# How organise

```scala
for {
 _ <- IO { log.info("start doing something") }
 _ <- IO { ... }
 _ <-
}
```
