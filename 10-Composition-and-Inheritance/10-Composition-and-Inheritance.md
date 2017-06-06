# 10-Composition-and-Inheritance.md
Chapter 6已经介绍了一些scala面向对象编程的基础，这章将深入介绍一些细节。

我们将在class间对比两个基础的关系：composition和inheritance。
composition意思是一个class拥有另一个class的引用，并使用它完成自己的的需求。
inheritance is the superclass/subclass relationship。

### 10.1 a two-dimensional layout library
在这章的案例里，我们会创建一个library，为了构建和展示二维layout element。每一个元素会代表一个矩形。为了方便，这个library会提供名为elem的factory方法，它会通过传递数据来构建新的元素。
如：

```
elem(s: String): Element
```
我们看到，元素会被构建成Element类型。你将能够对一个element调用above或beside，来得到一个新的element，实现结合两者。以下的例子将通过两个column，构建一个更大的element，每个高度为2：

```
val column1 = elem("hello") above elem("***") 
val column2 = elem("***") above elem("world") 
column1 beside column2
```
结果为：

```
hello *** 
*** world
```
这个例子很好的介绍了一个系统里，object能被一些简单的parts，通过composing operator进行创建。在这章，我们会定义一些类，使得element object能由array，line和rectangle构建。这些基础的element object将会是简单的parts。我们也将定义composing operator，如：above和beside。这些composing operator经常被称为combinators，因为他们经常将element连接成新的element。

### 10.2 abstract classes
我们首先的任务是定义类型Element，它代表着layout element。由于element是由二维矩阵字符串组成的，所以需要一个contents，指向一个layout element，所以其类型为Array[String]，如：

```
abstract class Element { 
    def contents: Array[String] 
}
```
在这个类里面，contents这个方法被定义成没有具体实现。换句话说，这个方式是类Element的abstract member。一个类含有abstract member必须被定义成abstract，如：

```
abstract class Element ...
```
abstract修饰符表名这个类有abstract member且没有具体实现。所以你不能实例化一个abstract类。

需要注意的是contents方法在类Element里面没有具体实现。在scala里面一个方法没有具体实现，就是abstract的。那些有具体实现的方法被称为concrete。

### 10.3 defining parameterless methods
下一步，我们将添加两个方法来展示width和height。height方法返回了lines的数量，width方法返回了第一各line的长度，或当没有时，返回0。

我们注意到这三个方法都没有参数，所以可以将圆括号省略：

```
def width(): Int
def width: Int
```
这些parameterless methods在scala里很常见，相反def height(): Int被称为empty-paren methods。建议当无参数或能access mutable state时，使用parameterless methods，This convention supports the uniform access principle。
我们能直接将method定义成field，如下：

```
abstract class Element { 
    def contents: Array[String] 
    val height = contents.length 
    val width = 
        if (height == 0) 0 else contents(0).length 
}
```
对客户端来说，这是相同的，但是定义成field更快，因为当类被初始化时，field已经被计算好了。另一方面，field需要额外的内存，所以需要考虑什么时候定义成method或field。

到目前为止，有一个问题，java里面没有实现uniform access principle,所以java里，是string.length()，不是string.length。scala为了解决这个gap，你能将parameterless method重写为一个empty-paren method，反之亦然。一个函数没有参数时，可以将empty parentheses省略。
如：

```
Array(1, 2, 3).toString 
"abc".length
```
结论，在scala里，一个method没有参数或没有side effect，最好定义成parameterless methods。如果有side effect，不能省略圆括号，不然的话看起来像定义了一个field。

### 10.4 extending classes
我们需要创建新的element object。由于Element是抽象的，所以不能被实例化，因此我们需要继承Element并实现抽象方法contents，如：

```
class ArrayElement(conts: Array[String]) extends Element { 
    def contents: Array[String] = conts 
}
```
extend关键词有两个效果：类ArrayElement会继承类Element的所有non-private成员，且it makes the type ArrayElement a subtype of the type Element。ArrayElement此时是Element的subclass，Element是ArrayElement的superclass。在不写extend的情况下，编译器会默认继承scala.AnyRef，在java里则是java.lang.Object。因此类Element隐式继承类AnyRef。如图所示：

![20170530149613169044577.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613169044577.png)

继承意味着所有superclass的成员也是subclass的成员，除了以下两种情况：
1、私有成员不是
2、在subclass里面已经实现了一个成员，且与superclass内的同名。这种情况下我们称subclass重写了superclass的成员。如果subclass里的成员是concrete，而superclass里的是abstract的，我们称为concrete成员实现了abstract成员。

如ArrayElement的contents方法重写(或称为实现)了Element的abstract method contents，而直接继承了width和height。所以可以直接使用，如：

```
scala> val ae = new ArrayElement(Array("hello", "world")) 
ae: ArrayElement = ArrayElement@39274bf7

scala> ae.width 
res0: Int = 5
```
由于是继承关系，所以实例化时可以这么写：

```
val e: Element = new ArrayElement(Array("hello"))
```
在10.1，我们在ArrayElement和Array[String]的关系称为composition relationship，是因为类ArrayElement is "composed" out of类Array[String]。scala编译器会在，为ArrayElement产生的二进制类里面放入一个field，其指向了传入的array。

### 10.5 overriding methods and fields
The uniform access principle只展示一方面，scala相比java，对待field和method更加的uniformly。另一个不同是，在scala里面，field和method属于同一个namespace。这允许field重写parameterless method。如：

```
class ArrayElement(conts: Array[String]) extends Element { 
    val contents: Array[String] = conts 
}
```
在scala的一个类里面，不能同时定义同名的field和method，而java里面是可以的。如：

```
// This is Java 
class CompilesFine { 
    private int f = 0; 
    public int f() { 
        return 1; 
    } 
}

But the corresponding Scala class would not compile:

class WontCompile { 
    private var f = 0 // Won't compile, because a field 
    def f = 1 // and method have the same name 
}
```
java的四个namespace是field，method，type和package，而scala只有两个namespace：

```
1、values (fields, methods, packages, and singleton objects)
2、types (class and trait names)
```
由于在一个namespace里面，所以可以使用val field直接重写parameterless method。

### 10.6 defining parametric fields

```
class Cat {
  val dangerous = false
}
class Tiger(param1: Boolean, param2: Int) extends Cat {
  override val dangerous = param1
  private var age = param2
}
```

### 10.7 invoking superclass constructors
现在我们已经有了两个类：abstract类Element和concrete类ArrayElement。客户端可能想创建一个由给定string组成的一个single line的layout element，如：

```
class LineElement(s: String) extends ArrayElement(Array(s)) {
  override val width = s.length
  override val height = 1
}
```
由于LineElement继承ArrayElement，而ArrayElement的构造器需要传入一个参数，所以LineElement需要给它的superclass的主构造器传入一个参数，如：LineElement传给ArrayElement一个Array(s)。
现在继承层级关系变成了这样：

![20170530149613536769621.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613536769621.png)

### 10.8 using override modifiers
如果重写parent类里的concrete成员，需要override修饰符，如果是abstract成员，则可选，如果不是parent类里面的成员，则不能写。

### 10.9 polymorphism and dynamic binding
如Section 10.4中展示的以下这种情况，称为polymorphism(多态)：

```
val ae = new ArrayElement(Array("hello", "world"))
```
多态意思为"many shapes" or "many forms"。
目前我们已经看到Element的两种form：ArrayElement和LineElement。我们能通过定义subclass来创建更多form，如：

```
class UniformElement( 
    ch: Char, 
    override val width: Int, 
    override val height: Int 
) extends Element { 
    private val line = ch.toString * width 
    def contents = Array.fill(height)(line) 
}
```
我们能进行如下定义：

```
val e1: Element = new ArrayElement(Array("hello", "world")) 
val ae: ArrayElement = new LineElement("hello") 
val e2: Element = ae 
val e3: Element = new UniformElement('x', 2, 3)
```
目前的层级关系如下：

![20170530149613722630261.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613722630261.png)

接下来我们介绍dynamically bound，如：

```
abstract class Element { 
    def demo() = { 
        println("Element's implementation invoked") 
    } 
}

class ArrayElement extends Element { 
    override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}

class LineElement extends ArrayElement { 
    override def demo() = { 
        println("LineElement's implementation invoked") 
    } 
}
class UniformElement extends Element
```
同时我们定义一个方法来调用demo：

```
def invokeDemo(e: Element) = { 
    e.demo() 
}
```
不同的调用，不同的结果：

```
scala> invokeDemo(new ArrayElement) 
ArrayElement's implementation invoked

scala> invokeDemo(new LineElement) 
LineElement's implementation invoked

scala> invokeDemo(new UniformElement) 
Element's implementation invoked
```
如果不重写demo，会默认使用superclass的demo。

### 10.10 declaring final members
当我们希望一个member不能重写时，我们可以使用final，如：

```
class ArrayElement extends Element { 
    final override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}
```
现在LineElement重写demo时会报错。
如果希望一个class不能被继承，也可以使用final，如：

```
final class ArrayElement extends Element { 
    override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}
```
现在LineElement继承ArrayElement时会报错。

### 10.11 using composition and inheritance
之前LineElement继承ArrayElement，由于我们指向使用contents，我们可以继承Element，如：

```
class LineElement(s: String) extends Element { 
    val contents = Array(s) 
    override def width = s.length 
    override def height = 1 
}
```
现在层级关系如下：

![20170530149613841950053.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613841950053.png)

### 10.12 implementing above, beside and toString

```
abstract class Element { 
    def contents: Array[String]
    
    def width: Int = 
        if (height == 0) 0 else contents(0).length
    
    def height: Int = contents.length
    
    def above(that: Element): Element = 
        new ArrayElement(this.contents ++ that.contents)
    
    def beside(that: Element): Element =
        new ArrayElement( 
            for ( 
                (line1, line2) <- this.contents zip that.contents 
            ) yield line1 + line2 )
    
    override def toString = contents mkString "\n"
}
```


### 10.13 defining a factory object

```
object Element {
    def elem(contents: Array[String]): Element = 
        new ArrayElement(contents) 
    
    def elem(chr: Char, width: Int, height: Int): Element = 
        new UniformElement(chr, width, height) 
    
    def elem(line: String): Element =
        new LineElement(line)
}
```

```
import Element.elem 
abstract class Element {
    def contents: Array[String] 
    
    def width: Int = 
        if (height == 0) 0 else contents(0).length 
    
    def height: Int = contents.length 
    
    def above(that: Element): Element = 
        elem(this.contents ++ that.contents) 
        
    def beside(that: Element): Element = 
        elem( 
            for ( 
            (line1, line2) <- this.contents zip that.contents 
            ) yield line1 + line2 
        ) 
    override def toString = contents mkString "\n"
}
```
### 10.14 heighten and widen

```
object Element {
    private class ArrayElement( 
        val contents: Array[String] 
    ) extends Element
    
    private class LineElement(s: String) extends Element { 
        val contents = Array(s) 
        override def width = s.length 
        override def height = 1 
    }
    
    private class UniformElement( 
        ch: Char, 
        override val width: Int, 
        override val height: Int 
    ) extends Element { 
        private val line = ch.toString * width 
        def contents = Array.fill(height)(line) 
    }
    
    def elem(contents: Array[String]): Element = 
        new ArrayElement(contents)
    
    def elem(chr: Char, width: Int, height: Int): Element = 
        new UniformElement(chr, width, height)
    
    def elem(line: String): Element = 
        new LineElement(line)
}
```

```
import Element.elem 
abstract class Element { 
    def contents: Array[String]
    
    def width: Int = contents(0).length 
    def height: Int = contents.length
    
    def above(that: Element): Element = { 
        val this1 = this widen that.width 
        val that1 = that widen this.width 
        elem(this1.contents ++ that1.contents) 
    }
    
    def beside(that: Element): Element = { 
        val this1 = this heighten that.height 
        val that1 = that heighten this.height 
        elem( 
            for ((line1, line2) <- this1.contents zip that1.contents) 
                yield line1 + line2
        ) 
    }
    
    def widen(w: Int): Element =
        if (w <= width) this 
        else {
            val left = elem(' ', (w - width) / 2, height)
            val right = elem(' ', w - width - left.width, height)
            left beside this beside right 
        }
    
    def heighten(h: Int): Element =
        if (h <= height) this 
        else {
            val top = elem(' ', width, (h - height) / 2)
            val bot = elem(' ', width, h - height - top.height)
            top above this above bot 
        }

    override def toString = contents mkString "\n"
}
```
### 10.15 putting it all together

```
import Element.elem

object Spiral {
    val space = elem(" ") 
    val corner = elem("+")
    
    def spiral(nEdges: Int, direction: Int): Element = {
        if (nEdges == 1) 
            elem("+") 
        else { 
            val sp = spiral(nEdges - 1, (direction + 3) % 4) 
            def verticalBar = elem('|', 1, sp.height) 
            def horizontalBar = elem('-', sp.width, 1) 
            if (direction == 0) 
                (corner beside horizontalBar) above (sp beside space) 
            else if (direction == 1) 
                (sp above space) beside (corner above verticalBar) 
            else if (direction == 2) 
                (space beside sp) above (horizontalBar beside corner) 
            else 
                (verticalBar above corner) beside (space above sp)
        }
    }
    
    def main(args: Array[String]) = { 
        val nSides = args(0).toInt 
        println(spiral(nSides, 0)) 
    }
}
```
### 10.16 小结


