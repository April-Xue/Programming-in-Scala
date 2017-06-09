# 8-Functions-and-Closures.md
Besides methods, which are functions that are members of some object, there are also functions nested within functions, function literals, and function values. This chapter takes you on a tour through all of these flavors of functions in Scala.
### 8.1 methods
在object里面定义的函数被称为方法。
### 8.2 local functions
理论上，程序应该被设计成由许多小函数组成，
优点是不同的函数能灵活的运用，来解决不同的困难，同时每个小函数应该尽量精简，能被简单理解。
缺点是当不同的函数被放在一起时，就要将helper函数隐藏起来，不被客户端调用。

在java里，我们主要使用private方法，scala里也适用。但是scala提供了不同的解决方法：通过函数内定义函数，能做到像本地变量一样，称为本地函数，只能在自己的作用域内被调用，如：

```
def processFile(filename: String, width: Int) = {
    def processLine(filename: String, width: Int, line: String) = {
    if (line.length > width) 
        println(filename + ": " + line.trim)
    }
    val source = Source.fromFile(filename) 
   for (line <- source.getLines()) { 
        processLine(filename, width, line) 
   }
}
```
由于本地函数能直接使用其包围函数的参数，所以可以简化：

```
def processFile(filename: String, width: Int) = {
    def processLine(line: String) = { 
        if (line.length > width) 
            println(filename + ": " + line.trim) 
    }
    val source = Source.fromFile(filename) 
    for (line <- source.getLines()) 
        processLine(line)
}
```

### 8.3 first-class functions
scala里有first-class函数,不仅能将其定义为函数和进行调用，还能将函数写成unnamed literal，作为value一样在程序中传递。function literal会被编译进class，当实例化时是一个function value。function literal存在于source code，function value在运行时作为object，就像class(source code)和object(runtime)的区别，如：

```
(x: Int) => x + 1
```

=>表示将左边的x转换为右边的x+1。

function value是对象，所以能将其存为变量，也能使用括号来调用他们，如：

```
scala> var increase = (x: Int) => x + 1 
increase: Int => Int = <function1>

scala> increase(10) 
res0: Int = 11
```

通过花括号能组成含有多个语句的function literal，如：

```
scala> increase = (x: Int) => { 
        println("We") 
        println("are") 
        println("here!") 
        x + 1 
        } 
increase: Int => Int = <function1>

scala> increase(10) 
We 
are 
here!
res2: Int = 11
```
另一个例子，如filter方法：

```
scala> val someNumbers = List(-11, -10, -5, 0, 5, 10) 
someNumbers: List[Int] = List(-11, -10, -5, 0, 5, 10)

scala> someNumbers.foreach((x: Int) => println(x)) 
-11 
-10 
-5 
0 
5
10

scala> someNumbers.filter((x: Int) => x > 0)
res4: List[Int] = List(5, 10)
```

### 8.4 short forms of function literals
之前的filter允许简写成：

```
scala> someNumbers.filter((x) => x > 0) 
res5: List[Int] = List(5, 10)
```
省略参数类型之后，编译器知道x必须是整数，被称为target typing。
还可以将圆括号省略，简写成：

```
scala> someNumbers.filter(x => x > 0) 
res6: List[Int] = List(5, 10)
```

### 8.5 placeholder syntax
还有使filter更简洁的方法，只要每个参数在function literal只出现一次，我们可以使用_作为占位符，来作为一个或多个参数。如：

```
scala> someNumbers.filter(_ > 0) 
res7: List[Int] = List(5, 10)
```
你可以将_当成是一个"blank"，需要被"filled in"。这个blank在每一次函数被调用时，都会被一个参数filled in。与如下等同：

```
scala> someNumbers.filter(x => x > 0) 
res8: List[Int] = List(5, 10)
```
有时候当你使用_来作为参数是，编译器不能有足够的信息来推断参数类型，如：

```
scala> val f = _ + _ 
<console>:7: error: missing parameter type for expanded 
function ((x$1, x$2) => x$1.$plus(x$2))
    val f = _ + _ 
              ^
```
你可以通过圆括号指定参数类型，如：

```
scala> val f = (_: Int) + (_: Int) 
f: (Int, Int) => Int = <function2>

scala> f(5, 10) 
res9: Int = 15
```
_+_将函数扩展成带有两个参数，但是只有每个参数在func literal里存在一次，才能使用如此简单地写法。多个_表示多个参数，二不是重复地使用单个参数。

### 8.6 partially applied functions
之前的例子中通过_替代独立的参数，还可以用_替换整个参数列表。如：

```
someNumbers.foreach(println _)
```
与如下相同：

```
someNumbers.foreach(x => println(x))
```
这个例子中，_不再是单个参数，而是一个占位符替代了整个参数列表。注意：需要将println和_用空格分开，不然编译器认为你在调用println_。

当你使用_时，其实是在写partially applied函数。但你调用一个函数时，传递需要的函数，称为apply函数to参数。如：

```
scala> def sum(a: Int, b: Int, c: Int) = a + b + c 
sum: (a: Int, b: Int, c: Int)Int
```
我们能apply函数sum to参数1，2，3，如：

```
scala> sum(1, 2, 3) 
res10: Int = 6
```
partially applied函数是一个表达式，你不必提供函数需要的所有参数，可以仅提供部分，甚至不提供。如sum _为一个partially applied函数：

```
scala> val a = sum _ 
a: (Int, Int, Int) => Int = <function3>
```
scala编译器从sum _实例化一个不含三个整数参数的function value，为a赋值一个引用。当调用这个function value，会重新调用sum，传入三个参数：

```
scala> a(1, 2, 3) 
res11: Int = 6
```
内部其实是：a指向由编译器从sum _自动生成的function value实例。最终调用了这个function value的apply方法(也是自动生成的)，即：

```
scala> a.apply(1, 2, 3) 
res12: Int = 6
```
***
另一个案例来理解partially applied function：

```
scala> val b = sum(1, _: Int, 3) 
b: Int => Int = <function1>
```
编译器会自动生成一个新的function class，其apply方法只接收一个参数，如：

```
scala> b(2) 
res13: Int = 6
其实调用的是sum(1, 2, 3)
```
如果你是用_来替代一个partially applied function的所有参数，你可以直接省略_，如：

```
someNumbers.foreach(println _)
直接写成
someNumbers.foreach(println)
```
仅当编译器知道foreach需要一个函数作为参数，这种才被允许。有时候，当不需要一个函数时，会导致错误，如：

```
scala> val c = sum 
<console>:8: error: missing arguments for method sum; 
follow this method with `_' if you want to treat it 
as a partially applied function 
    val c = sum 
            ^
```

### 8.7 closures
之前的例子中，all the examples of function literals have referred only to passed parameters。如(x: Int) => x > 0，函数体内是用的唯一变量是x，为函数的一个参数。但是你可以将变量定义在其他地方，如：

```
(x: Int) => x + more // how much more?
```
这里出现了more变量，其是一个free变量，因为function literal没有给予它真正意义。想法，x是一个bound变量，因为它在函数内有一样：被函数定义为自己的参数。

why the trailing underscore？
scala在partially applied functions上的语法，强调了scala与传统函数式语言(Haskell或ML)上的一个不同。在这些语言里面，partially applied functions被认为是普通的case，而且在使用时有很强的类型检查语法。

举个例子，say you mistook the drop(n: Int) method of List for tail(), and therefore forgot you need to pass a number to drop. You might write, "println(drop)"。由于scala已经采用传统的函数式传统，partially applied functions在任何地方都是ok的，这段代码会进行类型检查。然后，你会发现println的输出声明为<function>。drop会被认为是一个function object。由于println接收任何类型，所以编译会ok，但是会得到不一样的结果。

为了避免这种情况，scala要求你specify function arguments that are left out explicitly, 即使看起像_一样简单。scala只允许在类型已经确定的情况下可以将_省略。

另一方面，只有more被定义了，function literal就能正常工作，如：

```
scala> var more = 1 
more: Int = 1

scala> val addMore = (x: Int) => x + more 
addMore: Int => Int = <function1>

scala> addMore(10) 
res16: Int = 11
```
在function literal中创建的function value被称为closure。
一个没有free变量的function literal，如(x: Int) => x + 1，被称为closed term，term是a bit of source code。因此一个function value在运行时就被创建，在严格意义上不能称为closure，因为(x: Int) => x + 1is already closed as written。
有free变量的，如：(x: Int) => x + more，是一个open term。任何一个function value在(x: Int) => x + more运行时创建，会被捕获，需要一个more的binding。这个function value，包含捕获变量more的一个引用，被称为一个closure。因为这个function value是这个open term在结束时的最终产物。
当more改变时，结果也会发生改变，闭包能看见more的改变。scala的closure会自己捕获变量，而不是value的引用。相反的，闭包改变了more的值，对more也是可见的。如：

```
scala> val someNumbers = List(-11, -10, -5, 0, 5, 10) 
someNumbers: List[Int] = List(-11, -10, -5, 0, 5, 10)

scala> var sum = 0 
sum: Int = 0

scala> someNumbers.foreach(sum += _)

scala> sum 
res19: Int = -11
```
当someNumbers.foreach(sum += _)对sum不断的+=，sum能在外部感知到。

假如closure使用了一个函数的一个本地变量，而且这个函数被多个地方调用，会出现怎么样的结果呢？

```
scala> def makeIncreaser(more: Int) = (x: Int) => x + more

scala> val inc1 = makeIncreaser(1) 
inc1: Int => Int = <function1>

scala> val inc9999 = makeIncreaser(9999) 
inc9999: Int => Int = <function1>

scala> inc1(10) 
res20: Int = 11

scala> inc9999(10) 
res21: Int = 10009
```
每次调用这个函数时，会创建一个新的闭包。当调用makeIncreaser(1)，一个闭包就创建了，此时more的值为1，就被闭包捕获了。scala会将这些东西排列好，所以被捕获的参数生存在heap外部，不是在stack。

### 8.8 special function call forms
**Repeated parameters**

```
scala> def echo(args: String*) = 
        for (arg <- args) println(arg) 
echo: (args: String*)Unit

scala> echo()

scala> echo("one") 
one

scala> echo("hello", "world!") 
hello 
world!
```
在函数内部，repeated parameter的类型是Array，所以String*实际上是Array[String]，但是不能传入一个Array，如：

```
scala> val arr = Array("What's", "up", "doc?") 
arr: Array[String] = Array(What's, up, doc?)

scala> echo(arr) 
<console>:10: error: type mismatch; 
    found : Array[String] 
    required: String 
            echo(arr) 
                 ^
```
为了实现该功能，你可以定义成：

```
scala> echo(arr: _*) 
What's 
up 
doc?
```

**Named arguments**
在通常的函数调用中，参数通过按顺序匹配被一个个调用：

```
scala> def speed(distance: Float, time: Float): Float = 
        distance / time 
speed: (distance: Float, time: Float)Float

scala> speed(100, 10) 
res27: Float = 10.0
```
这个例子中，100匹配给distance，10匹配给time。

Named arguments允许在函数中不按顺序传递参数，通过传递时指定参数名实现。如：

```
scala> speed(distance = 100, time = 10) 
res28: Float = 10.0
```
也可以将positional and named 参数混合使用，此时positional参数come first。
Named arguments通常在combination with default parameter values中使用。

**Default parameter values**
scala允许你为函数的参数设置默认值。当调用函数时，参数可以省略，将使用默认值。如：

```
def printTime(out: java.io.PrintStream = Console.out) = 
    out.println("time = " + System.currentTimeMillis())
```
当调用printTime()，对out参数来说没有传递值，out仍会被赋予默认值。我们仍然可以传入参数，如printTime(Console.err)。

另一个例子：

```
def printTime2(out: java.io.PrintStream = Console.out,
                divisor: Int = 1) = 
    out.println("time = " + System.currentTimeMillis()/divisor)
```
printTime2可以被调用为printTime2()，此时两个参数将会使用默认值，但是仍可以传递一个参数，另一个参数使用默认值，如：

```
printTime2(out = Console.err)

printTime2(divisor = 1000)
```

### 8.9 tail recursion
第一个例子：

```
def approximate(guess: Double): Double = 
    if (isGoodEnough(guess)) guess 
    else approximate(improve(guess))
```
递归函数经常在搜索问题中被使用。你可能想让加快性能，如：

```
def approximateLoop(initialGuess: Double): Double = { 
    var guess = initialGuess 
    while (!isGoodEnough(guess)) 
        guess = improve(guess) 
    guess 
}
```
两种表达下，第一个函数更好，而且我们测试运行时间，两者是一样的。

看起来递归调用，比简单地从头到尾的循环更消耗性能。但是，在上面的例子里，scala编译器能apply一个重要的优化。像以上这种例子，当在最后一步调用自身，称为尾递归。scala编译器能检测到尾递归，在更新完函数参数为新值后，将其替换为a jump back to the beginning of the function。

建议多使用尾递归，这样能更优雅和简洁，但用尾递归解决问题时，并不会有额外的运行消耗。

**跟踪尾递归函数**
尾递归函数不会为每次的调用，建立一个新的stack frame，每一次的调用会在一个单独的frame中执行。This may surprise a programmer inspecting a stack trace of a program that failed.
如：

```
scala> boom(3) 
java.lang.Exception: boom!
    at .boom(<console>:5) 
    at .boom(<console>:6) 
    at .boom(<console>:6) 
    at .boom(<console>:6) 
    at .<init>(<console>:6) 
...
```
例子中的while循环和尾函数在编译后的代码，本质上是一样的。两个函数都会编译成13指令的java字节码。你会看到isGoodEnough和improveare在方法中被调用，但是approximate没有，scala编译器优化了递归调用：

```
public double approximate(double);
    Code:
        0: aload_0 
        1: astore_3 
        2: aload_0 
        3: dload_1 
        4: invokevirtual #24; //Method isGoodEnough:(D)Z 
        7: ifeq 12 
        10: dload_1 
        11: dreturn 
        12: aload_0 
        13: dload_1 
        14: invokevirtual #27; //Method improve:(D)D 
        17: dstore_1 
        18: goto 2
```
如果将boom变成递归调用：

```
def bang(x: Int): Int =
    if (x == 0) throw new Exception("bang!") 
    else bang(x - 1)
```
会得到：

```
scala> bang(5) 
java.lang.Exception: bang!
    at .bang(<console>:5) 
    at .<init>(<console>:6) ...
```
这次你只会看到一个stack frame。你可能会认为bang在调用自身前崩溃了，但并不是。你可以使用以下参数，来关闭scala shell或scalac编译器的stack trace：

```
-g:notailcalls
```
此时，你会得到一个较长的stack trace：

```
scala> bang(5) 
java.lang.Exception: bang!
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .<init>(<console>:6) ...
```
由于JVM指令集在实现高级尾递归时非常困难，所以scala的尾调用使用受限制。scala只能优化直接调用本身的递归，而不能优化间接递归：

```
def isEven(x: Int): Boolean = 
    if (x == 0) true else isOdd(x - 1) 
def isOdd(x: Int): Boolean =
    if (x == 0) false else isEven(x - 1)
```
如果最后的调用是function value，也不会进行优化，如：

```
val funValue = nestedFun _ 
def nestedFun(x: Int) : Unit = { 
    if (x != 0) { println(x); funValue(x - 1) } 
}
```
funValue引用了一个function value，本质上包装了对nestedFun的调用。当对一个参数apply function value，it turns around and applies nestedFun to that same argument，并返回结果。但这种情况，scala并不会优化。
Tail-call的优化，只能在一个方法或嵌套函数在最后一步调用自身的情况下，并不能通过一个function value或其他中介。

### 8.10 小结
本章介绍了scala的函数。
除了方法之外，
还提供了local functions, function literals, and function values。
除了正常的函数调用，scala还提供了partially applied functions and functions with repeated parameters。
如果可能的话，将函数调用实现成尾调用，不仅有好看的形式，还有与while循环同样的性能。
下一章将介绍how Scala's rich support for functions helps you abstract over control.


