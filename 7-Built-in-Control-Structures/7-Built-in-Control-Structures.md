# 7-Built-in-Control-Structures.md

目前scala内置的控制结构有：if，while，for，try，match和function calls，非常的少，因为与其基于基础语法一个个添加控制结构，scala直接将它们集成在类库里(Chapter 9 会详细介绍)，本章将仅介绍几个内置控制结构。

scala的控制结构都有返回值，这是函数式语言的特色，在命令式语言里也有类似，如三元操作符?:，scala里则为if。

### 7.1 if expressions
与其他语言类似，scala的if如：

```
var filename = "default.txt" 
if (!args.isEmpty) 
    filename = args(0)
```

由于scala的if有返回值，所以可以改写filename的初始化。

```
val filename =
    if (!args.isEmpty) 
        args(0) 
    else 
        "default.txt"
```

这种写法的优点有两个：
1、通过返回值将filename定义为val，使用val是函数式风格，与java中的final一样，这样能让读者清楚的知道filename不会被改变，而不是查找全文看filename什么时候被改变。
2、通过返回值，可以直接使用表达式来代替变量，如：

```
println(if (!args.isEmpty) args(0) else "default.txt")
```


### 7.2 while loops
与其他语言类似，scala的while如：

```
def gcdLoop(x: Long, y: Long): Long = { 
    var a = x 
    var b = y 
    while (a != 0) { 
        val temp = a 
        a = b % a 
        b = temp 
    } 
    b
}
```

do-while如：

```
var line = "" 
do {
    line = readLine()
    println("Read: " + line) 
} while (line != "")
```

while和do-while结构称为循环，不是表达式，因为没有返回具体值，而是类型为Unit的空值，写作()。
()的示范如下：

```
scala> def greet() = { println("hi") } 
greet: ()Unit

scala> () == greet() 
hi 
res0: Boolean = true
```

特别注意空值的使用，以下为错误示范：

```
var line = "" 
while ((line = readLine()) != "") // This doesn't work!
    println("Read: " + line)
```

由于"line = readLine()"返回总是为()，所以不等于""，与java中不同。

以下两段代码实现相同功能，相比之下递归不易理解，但是使用while循环会不断重新赋值，所以需要对代码进行更多推敲验证。

```
def gcdLoop(x: Long, y: Long): Long = { 
    var a = x 
    var b = y 
    while (a != 0) { 
        val temp = a 
        a = b % a 
        b = temp 
    } 
    b
}

def gcd(x: Long, y: Long): Long = 
    if (y == 0) x else gcd(y, x % y)
```

### 7.3 for expressions
**collections迭代**
如：

```
val filesHere = (new java.io.File(".")).listFiles
for (file <- filesHere) 
    println(file)
```

每一次迭代，重新初始化一个名为file的val值，因为filesHere为Array[File]，所以file类型为File，println(file)时，会自动调用File的toString方法。在scala里，不应该写成这样：

```
// Not common in Scala...
for (i <- 0 to filesHere.length - 1) 
    println(filesHere(i))
```
这种写法总是会纠结于边界问题，应该完全避免。

for循环简单示例：

```
for (i <- 1 to 4) //含边界
    println("Iteration " + i) 
Iteration 1 
Iteration 2 
Iteration 3 
Iteration 4

for (i <- 1 until 4) //不含边界
    println("Iteration " + i) 
Iteration 1 
Iteration 2 
Iteration 3
```

**筛选**

```
for (file <- filesHere if file.getName.endsWith(".scala")) 
    println(file)
```

```
for ( 
    file <- filesHere 
    if file.isFile 
    if file.getName.endsWith(".scala") 
) println(file)
```

**嵌套循环**

```
def grep(pattern: String) =
    for ( 
        file <- filesHere 
        if file.getName.endsWith(".scala"); 
        line <- fileLines(file) 
        if line.trim.matches(pattern) 
    ) println(file + ": " + line.trim)

这里必须写分号，不然只能将圆括号替换成花括号：
def grep(pattern: String) =
    for {
      file <- filesHere
      if file.getName.endsWith(".scala")
      line <- fileLines(file)
      if line.trim.matches(pattern)
    } println(file + ": " + line.trim)

```

**绑定中间变量**

```
def grep(pattern: String) =
for { 
    file <- filesHere 
    if file.getName.endsWith(".scala") 
    line <- fileLines(file) 
    trimmed = line.trim 
    if trimmed.matches(pattern) 
} println(file + ": " + trimmed)
```
**生成新collection**

```
val filesHere = (new java.io.File(".")).listFiles
for (file <- filesHere)
    println(file)
for (file <- filesHere if file.getName.endsWith(".scala"))
    println(file)
  //
def fileLines(file: java.io.File) =
    scala.io.Source.fromFile(file).getLines().toList
val forLineLengths =
    for {
      file <- filesHere
      if file.getName.endsWith(".scala")
      line <- fileLines(file)
      trimmed = line.trim
      if trimmed.matches(".*for.*")
    } yield trimmed.length
```
在Chapter 23将对for循环进行详细描述。

### 7.4 exception handling with try expressions
**抛出异常**

```
val half =
    if (n % 2 == 0) 
        n / 2 
    else
        throw new RuntimeException("n must be even")
```
异常has type Nothing，将会在Section 11.3中介绍。

**捕获异常**

```
import java.io.FileReader 
import java.io.FileNotFoundException 
import java.io.IOException

try { 
    val f = new FileReader("input.txt") 
    // Use and close file 
    } 
catch { 
    case ex: FileNotFoundException => // Handle missing file 
    case ex: IOException => // Handle other I/O error 
}
```
在scala里，并不用声明异常，可以直接使用

**finally**

```
val file = new FileReader("input.txt") 
    try {
        // Use the file 
    } finally {
        file.close() // Be sure to close the file 
}
```
使用finally，确保non-memory资源，如file，socket，database connectio被关闭。另外使用loan pattern可以更简洁，将会在Section 9.3中介绍。

**返回一个值**

```
def urlFor(path: String) =
try { 
    new URL(path) 
} 
catch {
    case e: MalformedURLException =>
        new URL("http://www.scala-lang.org") 
}
```
返回值为try中url，或exception抛出时catch中url，如果抛出exception时没有捕获，将没有返回值。

```
def f(): Int = try return 1 finally return 2
返回2，finally的return，会使try和catch里的值失效
def g(): Int = try 1 finally 2
返回1
```

### 7.5 match expressions

```
val firstArg = if (args.length > 0) args(0) else ""
firstArg match { 
    case "salt" => println("pepper") 
    case "chips" => println("salsa") 
    case "eggs" => println("bacon") 
    case _ => println("huh?") 
}
```
默认case是_，下划线是一个通配符，作为表达未知值的占位符，在scala被频繁使用。
并不用写break，其作为隐式存在。
与java最大的区别在于，scala的match可以有返回值，如：

```
val firstArg = if (!args.isEmpty) args(0) else ""
val friend =
    firstArg match { 
        case "salt" => "pepper" 
        case "chips" => "salsa" 
        case "eggs" => "bacon" 
        case _ => "huh?" 
    }
println(friend)
```

### 7.6 living without break and continue

java写法：

```
int i = 0; 
boolean foundIt = false; 
while (i < args.length) {
    // This is Java
    if (args[i].startsWith("-")) { 
        i = i + 1; 
     continue; 
    } 
    if (args[i].endsWith(".scala")) {
     foundIt = true;
        break; 
    } 
    i = i + 1;
}
```
scala中，尽量使用if替代continue，布尔值替代break，如：

```
var i = 0 
var foundIt = false
while (i < args.length && !foundIt) {
    if (!args(i).startsWith("-")) {
        if (args(i).endsWith(".scala")) 
            foundIt = true 
    } 
    i = i + 1 
}
```
其实用递归函数是最容易理解的：

```
def searchFrom(i: Int): Int =
    if (i >= args.length) -1 
    else if (args(i).startsWith("-")) searchFrom(i + 1) 
    else if (args(i).endsWith(".scala")) i 
    else searchFrom(i + 1)

val i = searchFrom(0)
```
如果非要使用break，scala提供了以下写法：

```
import scala.util.control.Breaks._ 
import java.io._

val in = new BufferedReader(new InputStreamReader(System.in))

breakable { 
    while (true) { 
        println("? ") 
        if (in.readLine() == "") 
            break 
    } 
}
```


### 7.7 viriable scope

```
def printMultiTable() = {
    var i = 1 // only i in scope here
    while (i <= 10) {
        var j = 1 // both i and j in scope here
        while (j <= 10) {
            val prod = (i * j).toString // i, j, and prod in scope here
            var k = prod.length // i, j, prod, and k in scope here
            while (k < 4) {
                print(" ")
                k += 1
            }
            print(prod)
            j += 1
        }
        // i and j still in scope; prod and k out of scope
        println()
        i += 1
    }
    // i still in scope; j, prod, and k out of scope
}
```

```
val a = 1; 
{
    val a = 2 // Compiles just fine
    println(a) 
} 
println(a)
```
打印2然后打印1。使用花括号，可以定义，定义过的相同名字的变量。但是易读性差，不建议这么写。

### 7.8 refactoring imperative-style code

```
def printMultiTable() = {
    var i = 1 // only i in scope here
    while (i <= 10) {
        var j = 1 // both i and j in scope here
        while (j <= 10) {
            val prod = (i * j).toString // i, j, and prod in scope here
            var k = prod.length // i, j, prod, and k in scope here
            while (k < 4) {
                print(" ")
                k += 1
            }
            print(prod)
            j += 1
        }
        // i and j still in scope; prod and k out of scope
        println()
        i += 1
    }
    // i still in scope; j, prod, and k out of scope
}
```
改写为：

```
// Returns a row as a sequence 
def makeRowSeq(row: Int) =
    for (col <- 1 to 10) yield { 
        val prod = (row * col).toString 
        val padding = " " * (4 - prod.length) 
        padding + prod 
    }

// Returns a row as a string 
def makeRow(row: Int) = makeRowSeq(row).mkString

// Returns table as a string with one row per line 
def multiTable() = {
    val tableSeq = // a sequence of row strings 
       for (row <- 1 to 10) 
       yield makeRow(row)
    tableSeq.mkString("\n")
}
```
当使用while循环，由于输出到标准输出，会有side effect。重构成函数式代码后，最终返回string，对string进行输出，将逻辑和输出分开，有利于单元测试。

尽量不要使用while和var，函数式编程中通常使用val，for，helper function。


### 7.9 小结
scala内置的控制结构非常简洁，由于有返回值，所以很符合函数式编程，而且可以与function literal很好结合，下一章将介绍。


