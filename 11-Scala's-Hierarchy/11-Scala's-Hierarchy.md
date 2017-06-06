# 11-Scala's-Hierarchy.md
### 11.1 scala's class hierarchy
在scala里面，每一个类继承一个共有的superclass Any。由于每个类是都Any的subclass，所以Any里的method能在任何一个object里被调用。scala还定义了所以类的subclass，Null和Nothing。

由于每个类都继承于Any，所以能使用==，!=，equals进行比较；能被hashed，使用##或hashCode；能被formatted，使用toString。由于==和!=被定义成final，所以在subclass内不能被重写，但是equals方法和==一样，和!=相反，所以subclass可以通过重写equals方法来达到需求。

```
final def ==(that: Any): Boolean 
final def !=(that: Any): Boolean 
def equals(that: Any): Boolean 
def ##: Int def hashCode: Int 
def toString: String
```
根类Any由两个subclass：AnyVal和AnyRef。
AnyVal是scala里所有value class的父类，目前有9个value class：Byte， Short， Char， Int， Long， Float， Double， Boolean， and Unit。这些class的实例被写成literal，如42时Int的一个实例。你不能通过new来创建这些类的实例，如：

```
So if you were to write:
scala> new Int

you would get:
<console>:5: error: class Int is abstract; cannot be instantiated
            new Int 
            ^
```
第9个value class，Unit，和java中的void一样，被用于一个方法does not otherwise return an interesting result。Unit有一个single instance value，被写作()。

在Chapter 5中提到过，这些value class支持算数和布尔运算符作为方法。如，Int有+和*方法，Boolean有||和&&方法。value class同时也继承Any的所有方法，如：

```
scala> 42.toString 
res1: String = 42

scala> 42.hashCode 
res2: Int = 42

scala> 42 equals 42 
res3: Boolean = true
```

value class是扁平的，它们都是AnyVal的subclass，但是互不为subclass。在不同的value class type之间存在着隐式转换。如scala.Int能在需要时隐式转换成scala.Long。

如Section 5.10中提到的，隐式转换也被用于add more functionality to value types。如：

```
scala> 42 max 43 
res4: Int = 43

scala> 42 min 43 
res5: Int = 42

scala> 1 until 5 
res6: scala.collection.immutable.Range = Range(1， 2， 3， 4)

scala> 1 to 5 
res7: scala.collection.immutable.Range.Inclusive 
    = Range(1， 2， 3， 4， 5)

scala> 3.abs 
res8: Int = 3

scala> (-3).abs 
res9: Int = 3
```
min，max，until，to和abs都被定义在scala.runtime.RichInt类里面，在Int和RichInt之前存在一个隐式转换。当对Int调用一个方法，其不存在与Int但存在于RichInt时，隐式转换就会生效。其他value class也存在这种情况。

另一个root class的subclass是AnyRef，This is the base class of all reference classes。之前提到过，在java里面，AnyRef只是类java.lang.Object的一个别名。所以在java和scala写的所有类，都继承于AnyRef。在java中，java.lang.Object就是AnyRef的实现。虽然我们Object和AnyRef都可以使用，但是scala里建议使用AnyRef。

### 11.2 how primitives are implemented
scala里面是怎么实现这些的？事实上，scala保存integer的方式和java一样，作为as 32-bit words。这有利于JVM的效率和java库的灵活性。标准操作如加法和除法，被作为原始操作来实现。当一个integer需要被seen as a (Java) object，scala使用java.lang.Integer，这会发生在，当对integer调用toString或对Any类型的变量赋值一个integer。Integers of type Int are converted transparently to "boxed integers" of type java.lang.Integer whenever necessary.

这和java 5里的auto-boxing相似，但是有所不同，scala里面的boxing比java更less visible。如：

```
// This is Java 
boolean isEqual(int x， int y) { 
    return x == y; 
} 
System.out.println(isEqual(421， 421));
```
你会得到true，但是使用如下：

```
// This is Java 
boolean isEqual(Integer x， Integer y) { 
    return x == y; 
} 
System.out.println(isEqual(421， 421));
```
你会得到false。因为421被boxed两次，所以x和y是两个不同的object。因为== means reference equality on reference types，由于Integer是一个reference type，所以结果是false。java里面，primitive types and reference types明显不同，这也展示了java不是纯面向对象的。

但是在scala里面：

```
scala> def isEqual(x: Int， y: Int) = x == y 
isEqual: (x: Int， y: Int)Boolean

scala> isEqual(421， 421) 
res10: Boolean = true

scala> def isEqual(x: Any， y: Any) = x == y 
isEqual: (x: Any， y: Any)Boolean

scala> isEqual(421， 421)
res11: Boolean = true
```
在scala里面，==被设计成be transparent with respect to the type's representation。对于value type，自然是equality。对于reference types other than Java's boxed numeric types，==被认为是继承于Object的equals的一个别名。这个方法原始是被定义为reference equality，但是在很多subclass里被重写，被用于他们自己的equality。这意味着，在scala里面永远不会遇到java的well- known trap concerning string comparisons，如：

```
scala> val x = "abcd".substring(2) 
x: String = cd

scala> val y = "abcd".substring(2) 
y: String = cd

scala> x == y 
res12: Boolean = true
```
在java里面，这样的比较是false，需要使用equals，但是经常被忘记。

有时候效率是最重要的，你会使用hash cons with一些类，比较他们的实例with reference equality。对于这种情况，AnyRef定义了一个额外的eq方法，它不能被重写，被实现作为reference equality。如：

```
scala> val x = new String("abc") 
x: String = abc

scala> val y = new String("abc") 
y: String = abc

scala> x == y 
res13: Boolean = true

scala> x eq y 
res14: Boolean = false

scala> x ne y 
res15: Boolean = true
```

### 11.3 bottom types
在层级关系的最底部，是Null和Nothing。类Null is the type of the null reference，是所有reference类的子类。Null和value type是不兼容的，如：

```
scala> val i: Int = null 
<console>:7: error: an expression of type Null is ineligible 
for implicit conversion
    val i: Int = null 
                 ^
```
类型Nothing是所有其他类型的subtype，无论如何这个类型是没有值的，因为它标志着不正常的终止。在scala的标准库里面，Predef对象有一个error方法，如：

```
def error(message: String): Nothing = 
    throw new RuntimeException(message)
```
error的返回类型是Nothing，告诉用户这个方法不会正常返回，它会抛出异常。由于Nothing是所有类型的subtype，所以你可以这么用：

```
def divide(x: Int， y: Int): Int =
    if (y != 0) x / y 
    else error("can't divide by zero")
```

### 11.4 defining your own value classes
如Section 11.1中提到的，我们可以定义自己的value class来扩展已经内置的。和内置的value class一样，你自己定义的会被编译成java字节码，且不会使用wrapper类，当一个wrapper需要时，像with generic code，value会get boxed and unboxed automatically。

只有certain类能被写成value class。对于要实现的value class，必须有一个参数，且除了def没有其它东西。而且，不能有类能继承，不能重新定义equals和hashcode。

定义value class时，实现成AnyVal的子类，参数是val的，如：

```
class Dollars(val amount: Int) extends AnyVal { 
    override def toString() = "$" + amount 
}
```
在Section 10.6中提到过，val允许参数作为field成员被访问。如：

```
scala> val money = new Dollars(1000000) 
money: Dollars = $1000000 
scala> money.amount 
res16: Int = 1000000
```
在这个例子中，money refers to这个value class的一个实例，在scala的源码里面是Dollars类型，但是编译成java字节码时会直接使用Int。

定义一些额外的类能在编译的时候帮你找错误：

```
def title(text: String， anchor: String， style: String): String = 
    s"<a id='$anchor'><h1 class='$style'>$text</h1></a>"
```
如果你写错了顺序，检测不出来

```
class Anchor(val value: String) extends AnyVal 
class Style(val value: String) extends AnyVal 
class Text(val value: String) extends AnyVal 
class Html(val value: String) extends AnyVal

def title(text: Text， anchor: Anchor， style: Style): Html = 
    new Html( 
        s"<a id='${anchor.value}'>" + 
            s"<h1 class='${style.value}'>" + 
            text.value + 
            "</h1></a>" )
```
指定了类型和顺序，编译的时候很有用

### 11.5 小结
这一章主要介绍了类的层级，下一章将介绍traits。


