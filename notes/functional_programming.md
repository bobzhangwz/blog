# Functional Programming

## Functional Programming is a *Programming Paradigm*

### What's *Programming Paradigm*???

[Programming paradigm](https://www.zhihu.com/question/50556311)


> 托马斯.库恩提出“科学的革命”的范式论之后，Robert Floyd在1979年图灵奖的颁奖演说中使用了编程范式一词。编程范式一般包括三个方面，以OOP为例：
    1. 学科的逻辑体系——规则范式：如类/对象、继承、动态绑定、方法改写、对象替换等等机制。
    2. 心理认知因素——心理范式：按照面向对象编程之父Alan Kay的观点，“计算就是模拟”。OO范式极其重视隐喻（metaphor）的价值，通过拟人化，按照自然的方式模拟自然。
    3. 自然观/世界观——观念范式：强调程序的组织技术，视程序为松散耦合的对象/类的集合，以继承机制将类组织成一个层次结构，把程序运行视为相互服务的对象们之间的对话。


编程范式是编程语言的一种分类方式，它并不针对某种编程语言。就编程语言而言，一种编程语言也可以适用多种编程范式。


### 一门语言可以对应一到多个编程范式，但是一般都具有偏向性

#### Procedural Programming
```
  var animial = "dog"
  if(animal == "dog") console.log("wangwang")
  else if(animal == "cat") console.log("miao miao")
```

#### Object-Oriented Programming

```
  interface Shout { void shout(); }
  class Dog implement Shout { override public void shout() { System.out.println("wang wang"); } }
  class Cat implement Shout { override public void shout() { System.out.println("miao miao"); } }

  Shout animal = new Dog()
  animal.shout()
```

#### XX oriented Programming

理解一种编程范式，首先看看它的*编程思路*

- 面向切面
- 面向对象
- 面向过程
- Reactive Programming

范式总是对应几个核心的概念，在一个范式上增加新的概念，它就可以演变成另外一个范式，因为解决问题的思路可能完全不一样了。
比如函数式编程范式中添加并发概念，整个编程范式就会演变成并发编程范式；
从面向过程式的编程范式中增加类、对象、实例等概念，就会演变成面向对象编程范式。

## 面向函数编程的编程思路


>  In computer science, functional programming is a *programming paradigm*—a style of building the structure and elements
  of computer programs—that treats computation as the evaluation of mathematical functions and avoids changing-state and
  mutable data. It is a declarative programming paradigm, which means programming is done with expressions[1] or declarations[2] instead of statements.
  In functional code, the output value of a function depends only on the arguments that are input to the function,
  so calling a function f twice with the same value for an argument x will produce the same result f(x) each time.
  Eliminating side effects, i.e. changes in state that do not depend on the function inputs,
  can make it much easier to understand and predict the behavior of a program,
  which is one of the key motivations for the development of functional programming.

1. High Order Function, 函数是头等变量
1. Immutable data，数据是不可变的(Immutability)
2. Pure function, 一个函数的输出只依赖于这个函数的输入，然后返回输出之外，再也不做其它的事情.(No side effect)

### Immutable data

FP is about transforming data, not about maintaining state. There is much less "state" stuff in FP.

#### 没有变量，只有常量，任何一个“变量”一旦被赋值，就不能再次被赋值。

```
var age = 18
happy_new_year()
print(age)  //What is the output
```

#### 不可变数据结构，数据一旦被创建出来，就不能被修改了。

```
public class SomeActivity extends Activity {
  private final List<...> mDataList = new ArrayList<>();
  void someMethod() {
    mDataList.add(...)
  }
}
```

#### Pure Function/No side effect

Predictable

引用透明

> Question: does javascript can write in a functional style

## Scala with functional

- High Order function
- *Case class*, *val*, *immutable.Map/List*,

### Special in Scala

- 编译型/静态语言/强类型
- Type parameter
- Implicit

### Polymorphism in Scala

- Inherit
- Typeclass
