# 15-Case-Classes-and-Pattern-Matching
Case classes are Scala's way to allow pattern matching on objects without requiring a large amount of boilerplate.


## 15.1 A SIMPLE EXAMPLE

```
abstract class Expr 
case class Var(name: String) extends Expr 
case class Number(num: Double) extends Expr 
case class UnOp(operator: String, arg: Expr) extends Expr 
case class BinOp(operator: String, left: Expr, right: Expr) extends Expr
```
**Case classed**  


```
First, it adds a factory method with the name of the class.

//instead of new Var("x")
scala> val v = Var("x") 
v: Var = Var(x)

Second, all arguments in the parameter list implicitly get a val prefix,
so they are maintained as fields

scala> v.name 
res0: String = x

scala> op.left 
res1: Expr = Number(1.0)

Third, the compiler adds toString, hashCode, and equals to your class.

scala> println(op) 
BinOp(+,Number(1.0),Var(x))

scala> op.right == Var("x") 
res3: Boolean = true

Finally, the compiler adds a copy method for making modified copies.

scala> op.copy(operator = "-") 
res4: BinOp = BinOp(-,Number(1.0),Var(x))
```
**Pattern matching**  

```
def simplifyTop(expr: Expr): Expr = expr match { 
    case UnOp("-", UnOp("-", e)) => e // Double negation 
    case BinOp("+", e, Number(0)) => e // Adding zero 
    case BinOp("*", e, Number(1)) => e // Multiplying by one 
    case _ => //result in Unit()
}
```
A pattern match includes a sequence of alternatives,   
each starting with the keyword case.   
Each alternative includes a pattern and one or more expressions.

A match expression is evaluated by trying each of the patterns in the order they are written.

```
A constructor pattern looks like UnOp("-", e), matches all values of type UnOp.
//the arguments to the constructor are themselves patterns
UnOp("-", UnOp("-", e))

A constant pattern like "+" or 1 matches values that are equal to the constant with respect to ==.

A variable pattern like e matches every value.

The wildcard pattern (_) also matches every value.
```
**match compared to switch**

```
1.match is an expression, which always results in a value.
2.Second, alternative expressions never "fall through" into the next case.
3.if none of the patterns match, an exception named MatchError is thrown.
```

## 15.2 KINDS OF PATTERNS
**Wildcard patterns**  

```
Wildcards can also be used to ignore parts of an object that you do not care about.
expr match { 
    case BinOp(_, _, _) => println(expr + " is a binary operation") 
    case _ => println("It's something else") 
}
```
**Constant patterns**  

```
def describe(x: Any) = x match { 
    case 5 => "five" 
    case true => "truth" 
    case "hello" => "hi!"
    case Nil => "the empty list" 
    case _ => "something else" 
}
```
**Variable patterns**  

```
A variable pattern matches any object, just like a wildcard.

scala> def describe(x: Any) = x match {
     |     case 5 => "five"
     |     case test => test
     | }
describe: (x: Any)Any

scala> describe(6)
res1: Any = 6

scala> describe(8)
res2: Any = 8

Unlike a wildcard, Scala binds the variable to whatever the object is.

You can't define wildcard and variable pattern at the same time.

A simple name starting with a lowercase letter is taken to be a pattern variable,
all other references are taken to be constants.

You can still use a lowercase name for a pattern constant
1.if the constant is a field of some object, use xx.pi
2.local variable, use back-tick

scala> class setTest{
     |   val test = 2
     | }
defined class setTest

scala> val t1 = new setTest
t1: setTest = setTest@68702e03

scala> val t2 = 3
t2: Int = 3

scala> def describe(x: Any) = x match {
     |     case 1 => "five"
     |     case t1.test => 2
     |     case `t2` => 3
     | }
describe: (x: Any)Any

scala> describe(1)
res16: Any = five

scala> describe(2)
res17: Any = 2

scala> describe(3)
res18: Any = 3
```
**Constructor patterns**  
A constructor pattern first check that the object is a member of the named case class,  
and then to check that the constructor parameters of the object match the extra patterns supplied.


```
expr match { 
    case BinOp("+", e, Number(0)) => println("a deep match") 
    case _ => 
}
```
**Sequence patterns**  

```
expr match {
    case List(0, _, _) => println("found it") 
    case _ =>
}
expr match { 
    case List(0, _*) => println("found it") 
    case _ => 
}
```
**Tuple patterns**  

```
def tupleDemo(expr: Any) =
    expr match { 
        case (a, b, c) => println("matched " + a + b + c) 
        case _ => 
    }
```
**Typed patterns**

```
def generalSize(x: Any) = x match { 
    case s: String => s.length 
    case m: Map[_, _] => m.size 
    case _ => -1 
}

scala> generalSize("abc") 
res16: Int = 3

something about Type erasure:

scala> def isIntIntMap(x: Any) = x match { 
    case m: Map[Int, Int] => true 
    case _ => false 
}

scala> isIntIntMap(Map(1 -> 1)) 
res19: Boolean = true

scala> isIntIntMap(Map("abc" -> "abc")) 
res20: Boolean = true

no information about type arguments is maintained at runtime

but Array is special in scala.
```
**Variable binding**  

```
expr match { 
    case UnOp("abs", e @ UnOp("abs", _)) => e 
    case _ => 
}

e as the variable and UnOp("abs", _) as the pattern.
```

## 15.3 PATTERN GUARDS

```
// won't compile
def simplifyAdd(e: Expr) = e match { 
    case BinOp("+", x, x) => BinOp("*", x, Number(2)) 
    case _ => e 
}

a pattern variable may only appear once in a pattern.

def simplifyAdd(e: Expr) = e match { 
    case BinOp("+", x, y) if x == y => BinOp("*", x, Number(2)) 
    case _ => e 
}
```

## 15.4 PATTERN OVERLAPS

```
def simplifyBad(expr: Expr): Expr = expr match { 
    case UnOp(op, e) => UnOp(op, simplifyBad(e)) 
    case UnOp("-", UnOp("-", e)) => e 
}
<console>:21: warning: unreachable code 
        case UnOp("-", UnOp("-", e)) => e 
                                        ^
                                        
the first case will match anything that would be matched by the second case
```

## 15.5 SEALED CLASSES
A sealed class cannot have any new subclasses added except the ones in the same file.

If you write a hierarchy of classes intended to be pattern matched, you should consider sealing them.

```
sealed abstract class Expr 
case class Var(name: String) extends Expr 
case class Number(num: Double) extends Expr 
case class UnOp(operator: String, arg: Expr) extends Expr 

def describe(e: Expr): String = e match { 
    case Number(_) => "a number" 
    case Var(_) => "a variable" 
}

If you match against case classes that inherit from a sealed class, 
the compiler will flag missing combinations of patterns with a warning message.

scala> def describe(e: Expr): String = e match {
     |     case Number(_) => "a number"
     |     case Var(_) => "a variable"
     | }
<console>:12: warning: match may not be exhaustive.
It would fail on the following input: UnOp(_, _)
       def describe(e: Expr): String = e match {
                                       ^
                     
def describe(e: Expr): String = (e: @unchecked) match { 
    case Number(_) => "a number" 
    case Var(_) => "a variable" 
}

exhaustivity checking for the patterns that follow will be suppressed
```

## 15.6 THE OPTION TYPE
Scala has a standard type named Option for optional values.  
Some(x), where x is the actual value,  
the None object, which represents a missing value.

```
scala> val capitals = Map(1 -> "Paris", 2 -> "Tokyo")
capitals: scala.collection.immutable.Map[Int,String] = Map(1 -> Paris, 2 -> Tokyo)

scala> capitals.get(1)
res0: Option[String] = Some(Paris)

scala> capitals.get(3)
res1: Option[String] = None
```

If a variable is allowed to be null, then you must remember to check it for null every time you use it.

By contrast, Scala encourages the use of Option to indicate an optional value.

When encounter a null error, it will b a type error in Scala rather than NullPointerException.

## 15.7 PATTERNS EVERYWHERE
**Patterns in variable definitions**  

```
scala> val myTuple = (123, "abc") 
myTuple: (Int, String) = (123,abc)

scala> val (number, string) = myTuple 
number: Int = 123 
string: String = abc

This construct is quite useful when working with case classes.

scala> val exp = new BinOp("*", Number(5), Number(1)) 
exp: BinOp = BinOp(*,Number(5.0),Number(1.0))

scala> val BinOp(op, left, right) = exp 
op: String = * 
left: Expr = Number(5.0) 
right: Expr = Number(1.0)
```
**Case sequences as partial functions**    

```
a case sequence is a function literal

val withDefault: Option[Int] => Int = { 
    case Some(x) => x 
    case None => 0 
}

scala> withDefault(Some(10)) 
res28: Int = 10

scala> withDefault(None) 
res29: Int = 0

a case sequence has multiple entry points, each with their own list of parameters.
```
a sequence of cases gives you a partial function.

```
scala> val second: List[Int] => Int = {
     |     case x :: y :: _ => y
     | }
<console>:7: warning: match may not be exhaustive.
It would fail on the following inputs: List(_), Nil
       val second: List[Int] => Int = {
                                      ^
you must first tell the compiler that you know you are working with partial functions.
                                      
val second: PartialFunction[List[Int],Int] = { 
    case x :: y :: _ => y 
}
```
you should try to work with complete functions whenever possible, 
because using partial functions allows for runtime errors.

**Patterns in for expressions**  

```
scala> for ((country, city) <- capitals) 
        println("The capital of " + country + " is " + city)    

capitals yields a sequence of pairs, 
so you can be sure that every generated pair can be matched against a pair pattern.

scala> val results = List(Some("apple"), None, Some("orange"))
results: List[Option[String]] = List(Some(apple), None, Some(orange))

scala> for (Some(fruit) <- results) println(fruit)
apple
orange

generated values that do not match the pattern are discarded.
```

## 15.8 A LARGER EXAMPLE
Skipped
## 15.9 CONCLUSION
Skipped




