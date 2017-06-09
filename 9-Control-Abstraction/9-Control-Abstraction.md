# 9-Control-Abstraction.md
在Chapter 7我们提到过scala没有太多内置的control abstractions，因为it gives you the ability to create your own。在之前的chapter，我们已经学习了function values。在这个chapter，我们将介绍如何使用function values来创建新的control abstractions。同时还会介绍currying和by-name parameters。

### 9.1 reducing code duplication
所有函数被分割成common parts和non-common parts：
common parts在每一次函数调用时都是一样的。non-common parts，可能会在下次调用时不同。
common parts在函数的内部，non-common parts必须以参数提供。
当你使用function value作为参数时，the non-common part of the algorithm is itself some other algorithm。
这种函数的每一次调用，能传递不同的function value作为参数，调用时，能选择每一次传递的参数。
这种将function作为参数的高阶函数，能使代码更简洁。

高阶函数的一个优点是能减少代码复制。举例，你正在写一个文件浏览器，想要提供API允许用户通过规则搜索文件，如：

```
object FileMatcher { 
private def filesHere = (new java.io.File(".")).listFiles

def filesEnding(query: String) =
    for (file <- filesHere; if file.getName.endsWith(query)) 
        yield file
}
```
filesEnding方法通过调用private helper filesHere能获得所以文件的列表，然后filter出符合用户传递的规则的文件。filesHere是private的，filesEnding是唯一能提供给客户端的API。

目前还没有代码重复，但是随着需求增加，函数也会增加：


```
object FileMatcher { 
private def filesHere = (new java.io.File(".")).listFiles

def filesEnding(query: String) =
    for (file <- filesHere; if file.getName.endsWith(query)) 
        yield file
        
def filesContaining(query: String) =
    for (file <- filesHere; if file.getName.contains(query)) 
        yield file
        
def filesRegex(query: String) =
    for (file <- filesHere; if file.getName.matches(query)) 
        yield file  
}
```

function value能简化这个问题。将method name作为value进行传递是不允许的，但你能传递function value达到需要的效果。如：

```
def filesMatching(query: String, matcher: (String, String) => Boolean) = {
    for (file <- filesHere; if matcher(file.getName, query)) 
        yield file
}
```
在这种实现方法中，if使用matcher来检查query，检查的方法取决于matcher的参数。matcher是一个function，因此has a => in the type，接收两个参数：file name和query，返回boolean，因此函数类型是(String, String) => Boolean。

有了filesMatching这个helper method，我们能通过传递合适的参数，来简化代码：

```
def filesEnding(query: String) = 
    filesMatching(query, _.endsWith(_))
    
def filesContaining(query: String) = 
    filesMatching(query, _.contains(_))

def filesRegex(query: String) = 
    filesMatching(query, _.matches(_))
```
这里的function literals使用了占位符，可能理解起来很难，所以进行一个说明：
"_.endsWith(_)"相当于：

```
(fileName: String, query: String) => fileName.endsWith(query)
```
由于filesMatching中的matcher需要两个String参数，所以你不需要指定参数类型，可以直接写成

```
(fileName, query) => fileName.endsWith(query)
```
由于两个参数在函数体中只使用了一次，所以可以使用占位符代替

```
_.endsWith(_)
```

以上代码还能更精简，query被传递给filesMatching，然后直接传递给matcher函数，所以可以在调用时，可以直接去掉query。
This passing back and forth is unnecessary because the caller already knew the query to begin with!
简写后：

```
object FileMatcher { 
    private def filesHere = (new java.io.File(".")).listFiles
    
    private def filesMatching(matcher: String => Boolean) = 
        for (file <- filesHere; if matcher(file.getName)) 
            yield file
    
    def filesEnding(query: String) = 
        filesMatching(_.endsWith(query))
    
    def filesContaining(query: String) = 
        filesMatching(_.contains(query))
    
    def filesRegex(query: String) = 
        filesMatching(_.matches(query))
}
```
以上演示的代码通过first-class函数来消除代码重复问题。在java中，我们可能会创建一个interface，包含一个String为参数的函数，返回一个boolean，然后对filesMatching传递继承于interface的匿名内部类实例。虽然这种实现去除了代码重复，当是增加了代码。

而且，案例中也展示闭包能帮你减少代码重复：
之前的例子中function literals，如"_.endsWith(_)"and"_.contains(_)"，在运行时实例化成function value，并不是闭包，因为没有捕获free变量。它们使用_，意味着参数来自于函数，即：使用了两个bound变量，没有free变量。
相反"_.endsWith(query)"，包含了一个bound变量：被_替代的参数，一个free变量：query，这是因为scala支持使用闭包时，能让你直接将query参数在filesMatching中去除。


### 9.2 simplifying client code
之前的例子演示了，高阶函数在实现API时减少了代码重复。另一个用途是，在使用API时，直接使用高阶函数，如：

```
def containsNeg(nums: List[Int]): Boolean = { 
    var exists = false 
    for (num <- nums) 
        if (num < 0) 
            exists = true 
    exists 
}
```
使用高阶函数后：

```
def containsNeg(nums: List[Int]) = nums.exists(_ < 0)
```
exists方法代表了一个control abstraction。它是由scala library提供的有特殊用途的循环结构，与内置的while和for不同。在之前的章节，高阶函数filesMatching在object FileMatcher实现时，减少了代码重复。exists方法提供了类似用途，但exists is public in Scala's collections API，它减少的是API的client code。如果exists不存在，你可能会这么写：

```
def containsOdd(nums: List[Int]): Boolean = { 
    var exists = false 
    for (num <- nums) 
        if (num % 2 == 1) 
            exists = true 
    exists 
}
```
你会发现与刚才相比仅仅是if判断不同，所以你可以直接这么写：

```
def containsOdd(nums: List[Int]) = nums.exists(_ % 2 == 1)
```
在scala标准库里面，还有其他类似exists的其他looping方法，经常能简化代码。

### 9.3 currying
A curried function is applied to multiple argument lists, instead of just one.
非curried函数：

```
scala> def plainOldSum(x: Int, y: Int) = x + y 
plainOldSum: (x: Int, y: Int)Int

scala> plainOldSum(1, 2) 
res4: Int = 3
```
curried函数：

```
scala> def curriedSum(x: Int)(y: Int) = x + y 
curriedSum: (x: Int)(y: Int)Int

scala> curriedSum(1)(2) 
res5: Int = 3
```
当调用curriedSum时，实际上是调用传统函数back to back。第一次调用使用了单个Int参数x，为第二个函数返回了一个function value。第二函数使用了Int参数y。如下first函数展现了第一个传统函数调用的过程：

```
scala> def first(x: Int) = (y: Int) => x + y 
first: (x: Int)Int => Int
```
apply first函数to 1，也就是，调用第一个函数，传入1，返回第一个函数：

```
scala> val second = first(1) 
second: Int => Int = <function1>
```
apply second函数to 2，返回结果：

```
scala> second(2) 
res6: Int = 3
```
这里的first和second函数仅仅演示了currying的过程。他们与curriedSum没有直接关联。然而，有一种方法能curriedSum的second函数获得一个真正的引用。你能使用占位符to use curriedSum in a partially applied function expression。如：

```
scala> val onePlus = curriedSum(1)_ 
onePlus: Int => Int = <function1>
```
curriedSum(1)_中的_是一个占位符for the second parameter list。其结果是function的一个引用，当调用时，将它单独的Int参数加上1并返回：

```
scala> onePlus(2) 
res7: Int = 3
```
如下演示了将单独的Int参数加上2：

```
scala> val twoPlus = curriedSum(2)_ 
twoPlus: Int => Int = <function1>

scala> twoPlus(2) res8: Int = 4
```

### 9.4 writing new control structures
在first-class函数中，你能很有效地创建新的control structures。你只需要将函数作为参数。
如：

```
scala> def twice(op: Double => Double, x: Double) = op(op(x)) 
twice: (op: Double => Double, x: Double)Double

scala> twice(_ + 1, 5) 
res9: Double = 7.0
```
op的类型是Double => Double，意思为接收一个Double作为参数，返回另一个Double。

当你发现a control pattern repeated in multiple parts of your code，你应该想到实现一个新的control structure。之前我们见到的filesMatching就是一个特殊的control pattern。现在我们举个例子更广泛的使用coding pattern，如：

```
def withPrintWriter(file: File, op: PrintWriter => Unit) = { 
    val writer = new PrintWriter(file) 
    try { 
        op(writer) 
    } finally { 
        writer.close() 
    } 
}
```
调用时：

```
withPrintWriter( 
    new File("date.txt"), 
    writer => writer.println(new java.util.Date) 
)
```
这种方法的优点是，在函数最后直接关闭文件，而不是在用户代码中，因为用户很可能忘记关闭。这种技术叫做loan pattern，如withPrintWriter，是一个control-abstraction function，打开一个资源，将其loans进一个函数。也就是loans一个PrintWriter给函数op。当函数结束，不管是正常返回或者有异常，文件在finally中都会被真正关闭。

你可以通过使用花括号来替代圆括号，使得客户端代码更像一个内置的control structure。在scala里，任何一个函数的调用只传递一个参数时，你可以选择用花括号来包围参数：

```
scala> println("Hello, world!") 
Hello, world!

scala> println { "Hello, world!" } 
Hello, world!
```
这种用法只能在一个参数的情况下，传入两个参数就会报错：

```
scala> val g = "Hello, world!" 
g: String = Hello, world!

scala> g.substring { 7, 9 } 
<console>:1: error: ';' expected but ',' found.
        g.substring { 7, 9 } 
                        ^
```

使用这种方法目的是，使得客户端通过花括号来写function literals，这样使得方法调用更像一个control abstraction。
之前的withPrintWriter有两个参数，所以不能使用花括号。但是由于传递给withPrintWriter的函数是最后一个参数，所以你能通过currying将参数分离，如：

```
def withPrintWriter(file: File)(op: PrintWriter => Unit) = { 
    val writer = new PrintWriter(file) 
    try { 
        op(writer) 
    } finally { 
        writer.close() 
    } 
}
```
这时候就可以这么调用：

```
val file = new File("date.txt")

withPrintWriter(file) { 
    writer => writer.println(new java.util.Date) 
}
```

### 9.5 by-name parameters
之前展示的withPrintWriter方法，与内置的控制结构如if和while不同，其在花括号内使用了一个参数。传递给withPrintWriter的函数需要一个PrintWriter类型的参数。这个参数即"writer =>"展示的：

```
withPrintWriter(file) { writer => 
    writer.println(new java.util.Date) 
}
```
当你想实现像if或while，where there is no value to pass into the code between the curly braces，可以通过by-name参数。

加入我们想实现一个名为myAssert的dassertion construct，它会使用一个function value作为输入，并根据flag来决定做什么。如果flag被设置，myAssert会调用传入的函数，确认后返回true。如果flag被关闭，myAssert会什么也不做。
不使用by-name参数时：

```
var assertionsEnabled = true

def myAssert(predicate: () => Boolean) = 
if (assertionsEnabled && !predicate()) 
    throw new AssertionError
```
但是使用起来时有点怪：

```
myAssert(() => 5 > 3)
```
你可能会省略空参数列表()和=>，但是：

```
myAssert(5 > 3) // Won't work, because missing () =>
```
通过by-name参数，你可以用=>替代()=>，如：

```
def byNameAssert(predicate: => Boolean) = 
    if (assertionsEnabled && !predicate) 
        throw new AssertionError
```
现在你可以省略()，像这样调用：

```
byNameAssert(5 > 3)
```
by-name类型，仅在参数中使用，不能使用by-name变量或成员。
这时候，你可能会想要省略=>，如：

```
def boolAssert(predicate: Boolean) =
    if (assertionsEnabled && !predicate) 
        throw new AssertionError
```
这种形式是可行的，调用结果也一样。
但是这里有一个很重要的不同：
由于boolAssert参数的类型是boolean，boolAssert(5 > 3)里面的表达式在调用boolAssert前先被求值，即5>3返回true，然后传给boolAssert。
相反，由于byNameAssert的参数类型是=> Boolean，在调用前不会被执行，当一个apply方法会对5>3求值时，function value才会被创建，然后传给byNameAssert。

这里的区别是，当assertionsEnabled设置成false时，在byNameAssert中看不到异常，在boolAssert能看到，因为先进行的是assertionsEnabled的判断，还没到5>3的求值，如：

```
scala> val x = 5 
x: Int = 5

scala> var assertionsEnabled = false 
assertionsEnabled: Boolean = false

scala> boolAssert(x / 0 == 0) 
java.lang.ArithmeticException: / by zero
    ... 33 elided
    
scala> byNameAssert(x / 0 == 0)
```

### 9.6 小结
这个chapter介绍了如何使用scala丰富的函数来创建control abstractions。你能在代码中使用函数来提取common control patterns，同时scala库中的高阶函数能复用那些代码中common的control patterns。我们还介绍了使用currying和by-name参数，能让你的高阶函数跟简洁。

之前已经介绍了很多function的东西，接下来我们将介绍对象编程的特性。

