# 6-Functional-Objects.md
we'll show you more aspects of object-oriented programming in Scala: class parameters and constructors, methods and operators, private members, overriding, checking preconditions, overloading, and self references.

### 6.1 a specification for class Rational
我们要设计一个Rational类，能够实现加减乘除。
rational number不能有mutable状态。当你将两个Rational object相加，会得到一个新的Rational object。
### 6.2 constructing a rational
我们的设计基于客户端会如何创建Rational object。由于我们决定将Rational设计成immutable，我们要求客户端提供数据来构造实例，如：

```
class Rational(n: Int, d: Int)
```
如果class没有body，我们可以不写花括号。n和d被称为class参数。scala编译器会gather up这个两个class参数，创建使用这两个参数的主构造器。

使用immutable object有四个好处：
1、不会有复杂的状态，不会经常改变
2、传递起来自由，不用像mutable object那样传递起来有所担心
3、不用担心两个线程同时接触会破坏状态，因为它永远不能被改变
4、能创建安全的hash table key，而mutable object放入HashSet时，下次可能会找不到。

Rational的初始化体现了java和scala的一个重要的不同，scala能直接使用参数，class参数能在class里直接使用，there's no need to define fields and write assignments that copy constructor parameters into fields。

scala编译器会编译class里除了field和method的所有代码进主构造器，如：

```
class Rational(n: Int, d: Int) {
    println("Created " + n + "/" + d)
}
```
scala编译器会在主构造器里面调用println。

### 6.3 reimplementing the toString method
当我们创建一个Rational的实例时，如果直接打印出来，是Rational@90110a。默认情况下，类Rational继承类java.lang.Object的toString，会打印类名+@+十六进制数字。toString的结果主要是用于debug print statements, log messages, test failure reports, and interpreter and debugger output。我们能重写toString：

```
class Rational(n: Int, d: Int) { 
    override def toString = n + "/" + d 
}
```


### 6.4 checking preconditions
下一步，我们将关心主构造器的一个问题，需要检查0的情况，如：

```
scala> new Rational(5, 0) 
res1: Rational = 5/0
```
面向对象编程的一个好处是，你可以将数据的有效性校验放在object内部。最好的实现就是define as a precondition of主构造器，使得分子不能是0。precondition会作为值的约束传入方法或构造器，a requirement which callers must fulfill。一种使用的方法是require，如：

```
class Rational(n: Int, d: Int) { 
    require(d != 0) 
    override def toString = n + "/" + d 
}
```
如果传入的是true，require会正常返回，不然require会阻止object被构造，并抛出异常IllegalArgumentException。

### 6.5 adding fields
目前，主构造器能合适地执行precondition，我们将关注add。我们添加了一个add函数：

```
class Rational(n: Int, d: Int) { 
    // This won't compile 
    require(d != 0) 
    override def toString = n + "/" + d 
    def add(that: Rational): Rational = 
        new Rational(n * that.d + that.n * d, d * that.d) 
}
```
但是会报错：

```
<console>:11: error: value d is not a member of Rational 
            new Rational(n * that.d + that.n * d, d * that.d)
<console>:11: error: value d is not a member of Rational 
            new Rational(n * that.d + that.n * d, d * that.d)
```
我们需要添加两个field，如：

```
class Rational(n: Int, d: Int) { 
    require(d != 0) 
    val numer: Int = n
    val denom: Int = d 
    override def toString = numer + "/" + denom 
    def add(that: Rational): Rational =
        new Rational( 
            numer * that.denom + that.numer * denom, 
            denom * that.denom 
    )

}
```
同时可以在外部调用：

```
scala> val r = new Rational(1, 2) 
r: Rational = 1/2

scala> r.numer 
res3: Int = 1

scala> r.denom 
res4: Int = 2
```

### 6.6 self reference
关键词this refers to the object instance on which the currently executing method was invoked，或者当在构造器中使用时，the object instance being constructed.
如：

```
def lessThan(that: Rational) = 
    this.numer * that.denom < that.numer * this.denom
```
this.numer指向了object的分子，你也可以省略this。
以下这种情况，this不能省略：

```
def max(that: Rational) = 
    if (this.lessThan(that)) that else this
```
第一个this可以省略，第二个this不能省略

### 6.7 auxiliary constuctors
在一个类里面，有时候你需要不同的构造器。在scala里，不同于主构造器的构造器称作“辅助构造器”。举例，分母为1的有理数可以直接简写成分子，如5/1可以直接写成5。因此调用new Rational(5,1)可以简写成new Rational(5)。于是我们需要为Rational类添加一个辅助构造器，只包含一个参数，且分母被预定义为1。
如：

```
class Rational(n: Int， d: Int) {

    require(d != 0) 
    
    val numer: Int = n 
    val denom: Int = d 
    
    def this(n: Int) = this(n， 1) // auxiliary constructor 
    override def toString = numer + "/" + denom 
    
    def add(that: Rational): Rational =
        new Rational(
        numer * that.denom + that.numer * denom，
        denom * that.denom
    )
}
```

辅助构造器必须以def this(...)开头，其仅仅调用主构造器，传入自己的参数，即分子与1。通过解释器，可以直接看到辅助构造器的效果：

```
scala> val y = new Rational(3) 
y: Rational = 3/1
```

在scala里，每一个辅助构造器必须首先调用同一个类里的其他一个构造器。每一个scala类里的每一个辅助构造器必须首先= this(...)，被调用的构造器要么是主构造器，要么是另一个辅助构造器。这个规则的net effect是每一个声明在scala里的构造器最终调用的是主构造器。因此，主构造器是一个类的单一入口。

### 6.8 private fields and methods
在之前版本的Rational类里，我们通过n和d来初始化numer和denom。因此会出现66/42可以简写为11/7的情况，但是主构造器并不能处理：

```
scala> new Rational(66， 42) res5: Rational = 66/42
```
为了简写，我们需要将分子和分母除以它们最大的公因数。如66/42，分子和分母的最大公因数为6，都除以6就可以简写成11/7。
如：

```
class Rational(n: Int， d: Int) {

    require(d != 0) 
    
    private val g = gcd(n.abs， d.abs) 
    
    val numer = n / g 
    val denom = d / g 
    
    def this(n: Int) = this(n， 1) 
    
    def add(that: Rational): Rational =
        new Rational(
           numer * that.denom + that.numer * denom，
           denom * that.denom
        ) 
    override def toString = numer + "/" + denom 
    
    private def gcd(a: Int， b: Int): Int =
    if (b == 0) a else gcd(b， a % b)
}
```

在这个版本的Rational里，添加了一个私有成员:g，修改了分子和分母的初始化。由于g是私有的，它只能在类里面被调用。还添加了一个私有方法:gcd，计算时传入两个整数的最大公因数。Section4.1中提到过，要声明成员或方法为私有，只需在定义前加上private关键字。像gcd这种"helper method"私有方法是为了在类内部调用，如主构造器。为了让g绝对有效，我们调用abs函数，传入n和d的绝对值。

scala编译器会按出现顺序将Rational的三个成员放入主构造器。由于g最先出现，它的初始化:gcd(n.abs， d.abs)会在其他两个成员之前，通过除以最大公因数g，每个Rational的构造结果会被简写：

```
scala> new Rational(66， 42) 
res6: Rational = 11/7
```

### 6.9 defining operators
接下来我们讨论如何使Rational变得更方便使用。
第一步，用数学符号替代add，由于+是合法的标识符，理解起来更直接，因此我们直接定义名为+的方法，并实现一个名为*的方法来进行乘法。
如：

```
class Rational(n: Int， d: Int) {

  require(d != 0)

  private val g = gcd(n.abs， d.abs) 
  val numer = n / g 
  val denom = d / g 

  def this(n: Int) = this(n， 1)

  def + (that: Rational): Rational =
    new Rational( 
      numer * that.denom + that.numer * denom， 
      denom * that.denom 
    )

  def * (that: Rational): Rational = 
    new Rational(numer * that.numer， denom * that.denom)

  override def toString = numer + "/" + denom

  private def gcd(a: Int， b: Int): Int = 
    if (b == 0) a else gcd(b， a % b)
}
```

通过以上Rational类的定义，我们可以实现：

```
scala> val x = new Rational(1， 2) 
x: Rational = 1/2

scala> val y = new Rational(2， 3) 
y: Rational = 2/3

scala> x + y 
res7: Rational = 7/6
```

正如Section 5.9描述过的，*比+优先级更高：

```
scala> x + x * y 
res9: Rational = 5/6

scala> (x + x) * y 
res10: Rational = 2/3

scala> x + (x * y) 
res11: Rational = 5/6
```

### 6.10 identifiers in scala
至此我们已经见到scala里两种组成标识符的方法：字符与运算符。除此之外还有，全部的四种将会在本章节进行解释。
一个标识符以字母或下划线开头，后面可以跟字符、数字或下划线。$符号也是一个字母，但是在scala编译器里面被保留，标识符不能带有$符号，不然会导致命名冲突。

scala依照java的习惯，使用camel-case标识符，如toString和HashSet。虽然下划线是合法的标识符，但是并不经常使用，一方面是为了和java保持一致，一方面是下划线在scala里有太多非标识符的用处，所以避免使用以下标识符，如to_string。

按照camel-case定义规则：
成员，方法参数，变量和函数应该以小写字母开头，如length，flatMap；
同时class和trait应该以大写字母开头，如BigInt，List，UnbalanceTreeMap。

关于以下划线结尾的标识符，如果你是使用val name_: Int = 1，编译器会报错，它会认为你定义了一个name_:的变量。为了避免出错，你需要定义成这样：val name_ : Int = 1.

在scala里，constant并不意味着val。尽管val在初始化之后保持不变，但是它仍是一个变量。如方法的参数是val，但是每次方法被调用时，都会有不同的值。在java里，常量通常全由大写字母组成，使用下划线进行分割，如MAX_VALUE和PI;在scala里，通常首字母大写，如XOffset和Pi，同时java风格的常量也适用。

操作符标识符有一个或多个操作符组成，如可打印的ASCII码:+，:，?，~或#。以下是一些操作符标识符：

```
+ ++ ::: <?> :->
```

scala编译器内部使用内置的$，将操作符标识符分割成java标识符。如：->，会被分割$colon$minus$greater，

由于scala里操作符标识符可以被定义成任意长度，所以与java略有不同。在java里，x<-y会被解析成四个4个字符，即x < - y。在scala里，<-会被解析成单个字符，即x <- y。

混合标识符，由字母数字开头，跟着下划线和操作符标识符。如:名为unary_+的方法是一元+操作符，名为：myvar_=的方法=操作符。

字符串标识符，由'...'加任意字符组成，如：`x`，`<clinit>`，`yield`，即使是scala保留关键字，但是使用时不能是Thread.yield，应该是Thread.'yield'()。

### 6.11 method overloading
我们已为Rational类添加了有理数的乘法，但是仍缺少混合运算，如"r * 2"必须写成让"r * new Rational(2)"。接下来我们将进行简化：
**Listing 6.5 - Rational with overloaded methods.**

```
class Rational(n: Int, d: Int) {

    require(d != 0)
    
    private val g = gcd(n.abs, d.abs) 
    val numer = n / g 
    val denom = d / g
    
    def this(n: Int) = this(n, 1)
    
    def + (that: Rational): Rational =
        new Rational( 
            numer * that.denom + that.numer * denom, 
            denom * that.denom 
        )
    
    def + (i: Int): Rational = 
        new Rational(numer + i * denom, denom)
    
    def - (that: Rational): Rational =
        new Rational( 
            numer * that.denom - that.numer * denom, 
            denom * that.denom 
        )
    
    def - (i: Int): Rational = 
        new Rational(numer - i * denom, denom)
    
    def * (that: Rational): Rational = 
        new Rational(numer * that.numer, denom * that.denom)
    
    def * (i: Int): Rational = 
        new Rational(numer * i, denom)
    
    def / (that: Rational): Rational = 
        new Rational(numer * that.denom, denom * that.numer)
    
    def / (i: Int): Rational = 
        new Rational(numer, denom * i)
    
    override def toString = numer + "/" + denom
    
    private def gcd(a: Int, b: Int): Int = 
        if (b == 0) a else gcd(b, a % b)
｝
```

以上的每一个方法名字都有两种实现方法，这就实现了重载。调用方法时，编译器会自动选择符合参数类型的重载方法。例如：

```
scala> val x = new Rational(2, 3) 
x: Rational = 2/3

scala> x * x 
res12: Rational = 4/9

scala> x * 2 
res13: Rational = 4/3
```

当没有最佳匹配时，编译器会报"ambiguous reference"错误。

### 6.12 implicit conversions
目前我们已经实现了"r * 2"的写法，但是交换顺序："2 * r"会报错，如：

```
scala> 2 * r 
<console>:10: error: overloaded method value * with 
alternatives:
    (x: Double)Double <and> 
    (x: Float)Float <and> 
    (x: Long)Long <and> 
    (x: Int)Int <and> 
    (x: Char)Int <and> 
    (x: Short)Int <and> 
    (x: Byte)Int 
    cannot be applied to (Rational) 
            2 * r 
              ^
```

"2 * r"相当于"2.* (r)"，但是对于整形的*方法，并没有实现的Rational参数。针对这种情况，我们可以创建一个自动将整数转化为Rational类型的隐式转换。如：

```
scala> implicit def intToRational(x: Int) = new Rational(x)
```

方法前implicit修饰符告诉编译器对整数进行自动转换，如：

```
scala> val r = new Rational(2,3) 
r: Rational = 2/3

scala> 2 * r 
res15: Rational = 4/3
```

在这个例子里，你不能将隐式转换定义在类内部，只有定义在编译器里才能生效。

### 6.13 a word of caution
我们调用代码时，目标是更易读，而不是更简洁，所以请慎重使用操作符标识符和隐式转换。

### 6.14 小结
在Chapter 30，我们会对equals和hashcode进行重写，使Rational能更好地支持==和放入hash表。
在Chapter 21，我们将介绍如何在伴生对象里面使用隐式方法定义，使得Rational更易被调用。


