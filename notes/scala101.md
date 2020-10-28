# Scala 101 - Setup scala repo

## What you can learn
1. How to setup a scala repo from scratch(scalafmt, code coverage).
2. How to use idea to dev scala. Learn some productive shortcut for IDEA.
3. Use build.sbt in easy way.
https://github.com/bobzhangwz/scala101

## Preparation
1. Install openJDK8
2. Install sbt, https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Mac.html
3. Install scala plugin in IDEA
4. Run `sbt new scala/scala-seed.g8`
5. enter project and run sbt test to build

## Init project by SBT (25min)
1. Run `sbt new scala/scala-seed.g8` to create a project.
2. Enter into the new project directory, run command `sbt` to start sbt console, run the folling command in console
```
sbt console> compile
sbt console> run
sbt console> test
sbt console> testOnly *.HelloSpec
```

3. Open IDEA to import the project(go through all the file in project)
  - Config sbt compile parameter
  - Run Test
  - Run main
  - Assign environment in IDEA
4. Write 'hello world' code
```
object Main {
  def main(args: Array[String]): Unit = {
    print(sys.env)
  }
}
object Main2 extends App {
  println("hello world2")
}
```
5. Some usefull key binding
| command                 | key                   |
|--|--|
| Optimise imports        | `Ctrl + Option + O`   |
| Go to Declaration/Usage | `Command + B`         |
| Type Info               | `Ctrl + Shift + P`    |
| Go To Test              | `Shift + Command + T` |
| Parameter Info          | `Command + P`         |
| Show Context Actions    | `Option + Return`     |


View > showImplicit Hints
## Optimise build.sbt(25min)
### Add compile option to build.sbt.(5min)

https://docs.scala-lang.org/overviews/compiler-options/index.html
https://blog.ssanj.net/posts/2019-06-14-scalac-2.13-options-and-flags.html
https://gist.github.com/guilgaly/b73ad98384051ecdc7be39b1f053fc87
https://github.com/DavidGregory084/sbt-tpolecat

1. Add in build.sbt
```
scalacOptions in Test ++= Seq("-Yrangepos")
scalacOptions ++= Seq(
  "-encoding"
  ...
// COPY FROM
// https://gist.github.com/guilgaly/b73ad98384051ecdc7be39b1f053fc87
)
```
2. Reload sbt `sbt console> reload`

`Global / onChangedBuildSource := ReloadOnSourceChanges`

3. Compile with unused import to verify scalac option works
4. Introduce https://github.com/DavidGregory084/sbt-tpolecat

### Add scalafmt(10min)
https://scalameta.org/scalafmt/docs/installation.html#sbt
https://scalameta.org/scalafmt/docs/configuration.html
1. Create file `project/plugins.sbt`
2. Add one line: `addSbtPlugin("org.scalameta" % "sbt-scalafmt" % "2.3.2")`
3. Add `.scalafmt.conf`
```
maxColumn = 120
continuationIndent.callSite = 2
continuationIndent.defnSite = 2
continuationIndent.extendSite = 2
align.tokens = []
align.openParenDefnSite = false
align.openParenCallSite = false
verticalMultiline.arityThreshold = 120
newlines.alwaysBeforeTopLevelStatements = false
newlines.alwaysBeforeElseAfterCurlyIf = false
newlines.alwaysBeforeCurlyBraceLambdaParams = false
newlines.afterCurlyLambda = never
newlines.alwaysBeforeMultilineDef = false
danglingParentheses = true
rewrite.rules = [SortImports, SortModifiers]
version = 2.3.0
```
4. Reload sbt
```
sbt console> scalafmtCheck
sbt console> test:scalafmtCheck
sbt console> scalafmt
sbt console> test:scalafmt
Add alias to build.sbt
addCommandAlias("checkAndTest", "scalafmtCheck;test:scalafmtCheck;test")
```
5. Config scalafmt in IDEA(code style>scala)

### Add code coverage(10min)

https://github.com/scoverage/sbt-scoverage

`addSbtPlugin("org.scoverage" % "sbt-scoverage" % "1.6.1")`
1. Set in build.sbt
```
lazy val coverageExcludedClasses = List(
  "example.Hello"
)
lazy val root = (project in file("."))
  .settings(
    coverageEnabled in Test := true,
    coverageMinimum := 90,
    coverageFailOnMinimum := true,
    coverageExcludedPackages := coverageExcludedClasses.mkString(";"),
  )
```
2. Try it in sbt console
```
sbt console> coverage
sbt console> test
sbt console> coverageReport
sbt console> coverageOff
```
3. Add alias in  build.sbt
```
addCommandAlias("checkAndTest", "clean;scalafmtCheck;test:scalafmtCheck;coverage;test;coverageReport;coverageOff")
```

### Add library and write spec2 test(5min)
https://etorreborre.github.io/specs2/website/SPECS2-4.8.3/quickstart.html

```
libraryDependencies ++= Seq(
"org.specs2" %% "specs2-core" % "4.8.3" % Test,
"ch.qos.logback" % "logback-classic" % "1.2.+"
)
```

> What's the difference between '%' and '%%'
https://mvnrepository.com/artifact/org.specs2/specs2-core
https://github.com/etorreborre/specs2/tree/SPECS2-4.8.3/examples/jvm/src/test/scala/examples
```
import org.specs2._

/** This is the "Unit" style for specifications */
class HelloWorldSpec extends mutable.Specification {
  "This is a specification for the 'Hello world' string".txt
  "The 'Hello world' string should" >> {
    "contain 11 characters" >> {
      "Hello world" must haveSize(11)
    }
    "start with 'Hello'" >> {
      "Hello world" must startWith("Hello")
    }
    "end with 'world'" >> {
      "Hello world" must endWith("world")
    }
  }
}
```

# Round2

## Write word count

从文件中读入文章, 并统计出现频率最高的5个单词, 并输出; 如果输入错误, 文件不存在, 输出错误

1. Add the file under resources directory
2. read content from file

`Source.fromURL(this.getClass.getClassLoader.getResource("news.txt")).getLines.toList.mkString`

`fromURL` will throw NullPointerException if `geResource` is null

Can be resolved by Option or Try

```
val a = Option(null)  // a is None
val b = Try {  throw new Error("") }.toEither  // be is Left
b match {
   case Left(error) => println(error.getMessage)
   case Right(text) => print(text)
}
extract the words
 count the words,  groupBy  map{ case _ => xxx }  identity
 sort the words  andthen
 print the result
package io.thoughtworks.wordcount

case class WordFrequency(word: String, counts: Int)

class WordCount {
  def listWords(text: String): List[WordFrequency] = ???
  private[wordcount] def countWords(words: List[String]): Map[String, Int] = ???
  private[wordcount] def extractWords(text: String): List[String] = ???
}

```

```
package io.thoughtworks.wordcount
import org.specs2.mutable.Specification
import org.specs2.specification.AllExpectations
class WordCountSpec extends Specification with AllExpectations {
  "word count".title
  val wordcount = new WordCount

  "should extract words" >> {
    "from empty string" >> {
      wordcount.extractWords(" ") should_== List.empty[String]
      wordcount.extractWords("") should_== List.empty[String]
    }

    "from normal words " >> {
      wordcount.extractWords("on monday") should_== List("on", "monday")
    }

    "to lowercase" >> {
      wordcount.extractWords("On Monday") should_== List("on", "monday")
    }

    "ignore special character" >> {
      wordcount.extractWords("On, - \"Monday\" 2,000.") should_== List("on", "monday", "2000")
    }

    "from multi line" >> {
      wordcount.extractWords("On \n\n Monday \n 2,000") should_== List("on", "monday", "2000")
    }
  }

  "should count words" >> {
    wordcount.countWords(List("on", "monday", "on")) should_== Map("on" -> 2, "monday" -> 1)
  }

  "should list word frequency in order" >> {
    wordcount.listWords("on monday on") should_== List(
      WordFrequency("monday", 1),
      WordFrequency("on", 2)
    )
    wordcount.listWords("on monday on monday monday") should_== List(
      WordFrequency("on", 2),
      WordFrequency("monday", 3)
    )
  }
}

```

Example input file(resources):
```
Many older people are being "airbrushed" out of coronavirus figures in the UK, charities have warned.

The official death toll has been criticised for only covering people who die in hospital - but not those in care homes or in their own houses.

It comes after the government confirmed there had been virus outbreaks at more than 2,000 care homes in England.

Meanwhile, scientific advisers for the government will meet later to review the UK's coronavirus lockdown measures.

The evaluation will be passed to the government - but ministers have said it was unlikely restrictions would change.

On Monday, the UK's chief medical adviser said he would like "much more extensive testing" in care homes due to the "large numbers of vulnerable people" there.

Prof Chris Whitty told the daily Downing Street coronavirus briefing on that 92 homes in the UK reported outbreaks in one day.

The Department of Health and Social Care later confirmed 2,099 care homes in England have so far had cases of the virus.

The figures prompted the charity Age UK to claim coronavirus is "running wild" in care homes for elderly people.

"The current figures are airbrushing older people out like they don't matter," Caroline Abrahams, the charity's director, said.

The Office for National Statistics is due to release new figures on the number of deaths involving coronavirus at 09:30 BST, which include every community death linked to Covid-19 in England and Wales.

```


# Round3

`环境变量中指定要统计的数据源类型, 如 文件/网址 解析校验, 格式如下

```
INPUT=URL:https://raw.githubusercontent.com/bobzhangwz/scala101/master/src/main/resources/news.txt
INPUT=FILE:/Downloads/news.txt
```

解析失败或者变量不存在打印错误

读取环境变量:  `sys.env`

读取文本
```
Source.fromURL  Source.fromFile()
val url = Source.fromURL(new URL("https://raw.githubusercontent.com/bobzhangwz/scala101/master/src/main/resources/news.txt")).getLines().toList.mkString

val file = Source.fromFile(new File("/Downloads/news.txt")).getLines().toList.mkString
```

### Nice to have:  错误处理 (15分钟)

技巧练习, 给 String 增加 toURL 和 toFile 方法(提示 使用 implicit)
"http://www.baidu.com".toURL
"/Download/news.txt".toFile
技巧练习 给String增加  `convert[T]: Either[Throwable, T]` 方法(提示: typeclass)
```
"http://www.baidu.com".convert[URL]
"/Download/news.txt".convert[File]
```

### 思考和练习
完善错误处理(手动写Either 和 flatmap)
