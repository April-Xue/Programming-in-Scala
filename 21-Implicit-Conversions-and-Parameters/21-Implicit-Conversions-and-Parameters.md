# 21-Implicit-Conversions-and-Parameters
You can change or extend your own code as you wish, but if you want to use someone else's libraries, you usually have to take them as they are.

Scala's answer is implicit conversions and parameters.

## 21.1 IMPLICIT CONVERSIONS
Implicit conversions are often helpful for working with two bodies of software that were developed without each other in mind.

Each library has its own way to encode a concept that is essentially the same thing.

Implicit conversions help by reducing the number of explicit conversions that are needed from one type to another.

```
val button = new JButton 
button.addActionListener(
    new ActionListener { 
        def actionPerformed(event: ActionEvent) = { 
            println("pressed!")
        }
    }
)
The only new information here is the code to be performed, namely the call to println. 
This new information is drowned out by the boilerplate。

implicit def function2ActionListener(f: ActionEvent => Unit) = 
    new ActionListener { 
        def actionPerformed(event: ActionEvent) = f(event) 
}
button.addActionListener( 
    (_: ActionEvent) => println("pressed!") 
)
Because function2ActionListener is marked as implicit, 
it can be left out and the compiler will insert it automatically.
```
Now, you know use implicit conversions can dress up existing libraries.

## 21.2 RULES FOR IMPLICITS
**Marking rule: Only definitions marked implicit are available**  
You can use implicit keyword to mark any variable, function, or object definition. The compiler will only select among the definitions you have explicitly marked as implicit.

**Scope rule: An inserted implicit conversion must be in scope as a single identifier, or be associated with the source or target type of the conversion**  
The Scala compiler will only consider implicit conversions that are in scope.

The implicit conversion must be in scope as a single identifier.

If you want to make someVariable.convert available as an implicit, you would need to import it, which would make it available as a single identifier.

The compiler will also look for implicit definitions in the companion object of the source or expected target types of the conversion.

**One-at-a-time rule: Only one implicit is inserted.**  
The compiler will never rewrite x + y to convert1(convert2(x)) + y.

**Explicits-first rule: Whenever code type checks as it is written, no implicits are attempted**  
The compiler will not change code that already works.

You can always replace implicit identifiers by explicit ones.

**Naming an implicit conversion**  
Implicit conversions can have arbitrary names. But sometimes name matters.

```
object MyConversions { 
    implicit def stringWrapper(s: String):
        IndexedSeq[Char] = ...
    implicit def intToString(x: Int): String = ... 
}

If you want stringWrapper, not intToString
import MyConversions.stringWrapper 
... // code making use of stringWrapper
```
**Where implicits are tried**  
There are three places implicits are used in the language:   
1.conversions to an expected type  
let you use one type in a context where a different type is expected. 
 
2.conversions of the receiver of a selection  
"abc".exists, which is converted to stringWrapper("abc").exists  
because the existsmethod is not available on Strings but is available on IndexedSeqs.
  
3.implicit parameters.  
especially useful with generic functions, which may know nothing at all about the type of one or more arguments.

## 21.3 IMPLICIT CONVERSION TO AN EXPECTED TYPE

```
scala> val i: Int = 3.5
<console>:7: error: type mismatch;
 found   : Double(3.5)
 required: Int
       val i: Int = 3.5
                    ^
scala> implicit def doubleToInt(x: Double) = x.toInt 
doubleToInt: (x: Double)Int

scala> val i: Int = 3.5 
i: Int = 3
before it failed, it finds doubleToInt,
because doubleToInt is in scope as a single identifier.
```
But, converting Doubles to Ints causes a loss in precision.

The scala.Predef object, which is implicitly imported into every Scala program, defines implicit conversions that convert "smaller" numeric types to "larger" ones.

```
scala> val i: Double = 3
i: Double = 3.0
```

## 21.4 CONVERTING THE RECEIVER
This kind of implicit conversion has two main uses:  
First, receiver conversions allow smoother integration of a new class into an existing class hierarchy.  
And second, they support writing domain-specific languages (DSLs) within the language.

**Interoperating with new types**  

```
class Rational(n: Int, d: Int) { 
    ...
    def + (that: Rational): Rational = ... 
    def + (that: Int): Rational = ...
}

scala> val oneHalf = new Rational(1, 2) 
oneHalf: Rational = 1/2

scala> oneHalf + oneHalf 
res0: Rational = 1/1

scala> oneHalf + 1 
res1: Rational = 3/2

scala> 1 + oneHalf 
<console>:6: error: overloaded method value + with 
alternatives (Double)Double <and> ... cannot be applied 
to (Rational)
    1 + oneHalf 
      ^
      
scala> implicit def intToRational(x: Int) = 
        new Rational(x, 1) 
intToRational: (x: Int)Rational

scala> 1 + oneHalf 
res2: Rational = 3/2
```
**Simulating new syntax**  

```
Map(1 -> "one", 2 -> "two", 3 -> "three")
-> is a method of the class ArrowAssoc, this class is defined in scala.Predef
convert from Any to ArrowAssoc.

//start 2.11.0
implicit final class ArrowAssoc[A](private val self: A) extends AnyVal {
    @inline def -> [B](y: B): Tuple2[A, B] = Tuple2(self, y)
    def →[B](y: B): Tuple2[A, B] = ->(y)
}
When you write 1 -> "one", the compiler inserts a conversion from 1 to ArrowAssoc, 
and invoke -> method.

note: @deprecated("Use `ArrowAssoc`", "2.11.0") def any2ArrowAssoc[A](x: A): ArrowAssoc[A]
```
**Implicit classes**  
Implicit classes were added in Scala 2.10 to make it easier to write rich wrapper classes.

The compiler generates an implicit conversion from the class's constructor parameter to the class itself.

```
case class Rectangle(width: Int, height: Int)

If you use this class very frequently, 
you might want to use the rich wrappers pattern so you can more easily construct it.

implicit class RectangleMaker(width: Int) { 
    def x(height: Int) = Rectangle(width, height) 
}

// Automatically generated 
implicit def RectangleMaker(width: Int) = 
    new RectangleMaker(width)
    
scala> val myRectangle = 3 x 4 
myRectangle: Rectangle = Rectangle(3,4)
```
An implicit class cannot be a case class, and its constructor must have exactly one parameter.

An implicit class must be located within some other object, class, or trait. 

## 21.5 IMPLICIT PARAMETERS

```
scala> class PreferredPrompt(val preference: String)
defined class PreferredPrompt

scala> object Greeter {
     |   def greet(name: String)(implicit prompt: PreferredPrompt) = {
     |     println("Welcome, " + name + ". The system is ready.")
     |     println(prompt.preference) }
     | }
defined object Greeter

scala> object JoesPrefs {
     |   implicit val prompt = new PreferredPrompt("Yes, master> ")
     | }
defined object JoesPrefs

scala> val bobsPrompt = new PreferredPrompt("relax> ")
bobsPrompt: PreferredPrompt = PreferredPrompt@5ec0a365

scala>Greeter.greet("Joe")
<console>:10: error: could not find implicit value for parameter prompt: PreferredPrompt
                  Greeter.greet("Joe")
                               ^
you bring it into scope via an import
scala> import JoesPrefs._
import JoesPrefs._

scala> Greeter.greet("Joe")
Welcome, Joe. The system is ready.
Yes, master>

Note that the implicit keyword applies to an entire parameter list, not to individual parameters.
object Greeter {
def greet(name: String)(implicit prompt: PreferredPrompt, 
                        drink: PreferredDrink) = {
    ...                        
｝
Now, both PreferredPrompt and PreferredDrink are implicit.
```
Another thing to know about implicit parameters is that they are perhaps most often used to provide information about a type mentioned explicitly in an earlier parameter list,

```
def maxListImpParm[T](elements: List[T])(implicit ordering: Ordering[T]): T =
    elements match {
        case List() => 
            throw new IllegalArgumentException("empty list!") 
        case List(x) => x 
        case x :: rest => 
            val maxRest = maxListImpParm(rest)(ordering) 
            if (ordering.gt(x, maxRest)) x 
            else maxRest
}

scala> maxListImpParm(List(1,5,10,3)) 
res9: Int = 10

scala> maxListImpParm(List(1.5, 5.2, 10.7, 3.14159)) 
res10: Double = 10.7

scala> maxListImpParm(List("one", "two", "three")) 
res11: String = two
```
**A style rule for implicit parameters**  
Use at least one role-determining name within the type of an implicit parameter.

## 21.6 CONTEXT BOUNDS

```
def maxList[T](elements: List[T]) (implicit ordering: Ordering[T]): T =
  elements match {
    case List() =>
      throw new IllegalArgumentException("empty list!")
    case List(x) => x
    case x :: rest =>
      val maxRest = maxList(rest)    // (ordering) is implicit
      if (ordering.gt(x, maxRest)) x // this ordering is
      else maxRest                   // still explicit
}

by using 
def implicitly[T](implicit t: T) = t // defined in scala.Predef
and using context bound

def maxList[T : Ordering](elements: List[T]): T =
    elements match { 
        case List() => 
            throw new IllegalArgumentException("empty list!")
        case List(x) => x 
        case x :: rest => 
            val maxRest = maxList(rest) 
            if (implicitly[Ordering[T]].gt(x, maxRest)) x 
            else maxRest
}
The syntax [T : Ordering] is a context bound, and it does two things. 
First, it introduces a type parameter T as normal. 
Second, it adds an implicit parameter of type Ordering[T].
```

## 21.7 WHEN MULTIPLE CONVERSIONS APPLY

```
scala> implicit def intToDouble1(x: Double):Int = x.toInt
warning: there was one feature warning; re-run with -feature for details
intToDouble1: (x: Double)Int

scala> implicit def intToDouble2(x: Double):Int = x.toInt + 1
warning: there was one feature warning; re-run with -feature for details
intToDouble2: (x: Double)Int

scala> val a: Int = 2.5
<console>:29: error: type mismatch;
 found   : Double(2.5)
 required: Int
Note that implicit conversions are not applicable because they are ambiguous:
 both method intToDouble1 of type (x: Double)Int
 and method intToDouble2 of type (x: Double)Int
 are possible conversion functions from Double(2.5) to Int
       val a: Int = 2.5
                    ^
```
After Scala 2.8, If one of the available conversions is strictly more specific than the others, then the compiler will choose the more specific one.

Ane implicit conversion is more specific than another if one of the following applies:  
• The argument type of the former is a subtype of the latter's.  
• Both conversions are methods, and the enclosing class of the former extends the enclosing class of the latter.

## 21.8 DEBUGGING IMPLICITS
Implicits are powerful, but sometimes difficult to get right.

First, you can just write the implicit conversion out explicitly.

Or, the -Xprint:typer can help to see what implicit conversions the compiler is inserting. But if do so, you'll see an enormous amount of boilerplate surrounding the meat of your code.
## 21.9 CONCLUSION
Before adding a new implicit conversion,   
first ask whether you can achieve a similar effect through other means,   
such as inheritance, mixin composition, or method overloading.  


