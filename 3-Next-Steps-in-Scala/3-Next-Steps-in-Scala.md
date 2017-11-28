# 3-Next-Steps-in-Scala.md


### 7.parameterize arrays with types
In Scala, you can instantiate objects, or class instances, using new.
 
You parameterize an instance with values by passing objects to a constructor in parentheses.

```
val big = new java.math.BigInteger("12345")
```
When you parameterize an instance with both a type and a value, the type comes first in its square brackets, followed by the value in parentheses.

```
val greetStrings = new Array[String](3)

greetStrings(0) = "Hello" 
greetStrings(1) = ", " 
greetStrings(2) = "world!\n"
When you define a variable with val, the variable can't be reassigned,  
but the object to which it refers could potentially still be changed.

for (i <- 0 to 2) 
    print(greetStrings(i))
If a method takes only one parameter, you can call it without a dot or parentheses.
The code 0 to 2 is transformed into the method call (0).to(2).
```
Scala doesn't technically have operator overloading, because it doesn't actually have operators in the traditional sense. Instead, characters such as +, -, *, and / can be used in method names.

![20171127151178234531287.png](http://op2oo1z8b.bkt.clouddn.com/20171127151178234531287.png)

When you apply parentheses surrounding one or more values to a variable, Scala will transform the code into an invocation of a method named apply on that variable. So greetStrings(i) gets transformed into greetStrings.apply(i).

Any application of an object to some arguments in parentheses will be transformed to an apply method call, only if that type of object actually defines an apply method.

Similarly, when an assignment is made to a variable to which parentheses and one or more arguments have been applied, the compiler will transform that into an invocation of an update method.

```
greetStrings(0) = "Hello"
greetStrings.update(0, "Hello")
```

```
val numNames = Array("zero", "one", "two")
val numNames2 = Array.apply("zero", "one", "two")
```
What you're actually doing is calling a factory method, named apply, which creates and returns the new array.

This apply method takes a variable number of arguments and is defined on the Array companion object.

### 8.use lists
One of the big ideas of the functional style of programming is that methods should not have side effects.

A method's only act should be to compute and return a value, so that they become less entangled, and therefore more reliable and reusable.

Applying this functional philosophy to the world of objects means making objects immutable.

after array is instantiated, you can change its element values, so they are mutable objects.

Lists are always immutable.

```
val oneTwoThree = List(1, 2, 3)
```
List has a method named `:::' for list concatenation.

```
val oneTwo = List(1, 2) 
val threeFour = List(3, 4)
val oneTwoThreeFour = oneTwo ::: threeFour
println(oneTwoThreeFour + " is a new list.")

you get
List(1, 2, 3, 4) is a new list.
```
`::', which is pronounced "cons.", it prepends a new element to the beginning of an existing list and returns the resulting list.

```
val twoThree = List(2, 3) 
val oneTwoThree = 1 :: twoThree 
println(oneTwoThree)

You'll see:
List(1, 2, 3)
```
If a method is used in operator notation, such as a * b, the method is invoked on the left operand, like a.*(b).

If the method name ends in a colon, the method is invoked on the right operand. 1 :: twoThree, like twoThree.::(1).

An empty list is Nil.


```
val oneTwoThree = 1 :: 2 :: 3 :: Nil
```
Class List does offer an "append" operation—it's written :+.  
The time it takes to append to a list grows linearly with the size of the list,   whereas prepending with :: takes constant time.

If you want to build a list efficiently by appending elements, you can prepend them and when you're done call reverse. Or you can use a ListBuffer, a mutable list that does offer an append operation.

### 9.use tuples
Like lists, tuples are immutable,  
but unlike lists, tuples can contain different types of elements.

```
val pair = (99, "Luftballons") 
println(pair._1) 
println(pair._2)
```
These _N numbers are one-based, instead of zero-based,  
because starting with 1 is a tradition set by other languages with statically typed tuples, such as Haskell and ML.

### 10.use sets and maps
Scala also provides mutable and immutable alternatives for sets and maps.

Scala then provides two subtraits, one for mutable sets and another for immutable sets.

```
var jetSet = Set("Boeing", "Airbus") 
You invoke apply on the companion object for scala.collection.immutable.Set, 
which returns an instance of a default, immutable Set.

jetSet += "Lear" 
On both mutable and immutable sets, 
the + method will create and return a new set with the element added.

println(jetSet.contains("Cessna"))
```
![20171127151178375756297.png](http://op2oo1z8b.bkt.clouddn.com/20171127151178375756297.png)

As with sets, Scala provides mutable and immutable versions of Map, using a class hierarchy.

![20171127151178421424813.png](http://op2oo1z8b.bkt.clouddn.com/20171127151178421424813.png)

```
import scala.collection.mutable

val treasureMap = mutable.Map[Int, String]() 
treasureMap += (1 -> "Go to island.") 
treasureMap += (2 -> "Find big X on ground.") 
treasureMap += (3 -> "Dig.") 
println(treasureMap(2))
```
This -> method, which you can invoke on any object in a Scala program,  
returns a two-element tuple containing the key and value.

If you prefer an immutable map, no import is necessary, as immutable is the default map.

### 11.learn to recognize the functional style
Scala allows you to program in an imperative style, but encourages you to adopt a more functional style.

If code contains any vars, it is probably in an imperative style.

If the code contains no vars at all, like it contains only vals, it is probably in a functional style.

```
//imperative style
def printArgs(args: Array[String]): Unit = { 
    var i = 0 
    while (i < args.length) { 
        println(args(i)) i += 1 
    }
}
//functional style
def printArgs(args: Array[String]): Unit = { 
    args.foreach(println) 
}
still not purely functional because it has side effects,
which is printing to the standard output stream,
and its result type is Unit.

//more functional approach
def formatArgs(args: Array[String]) = args.mkString("\n")
//you still need a print
println(formatArgs(args))

Every useful program is likely to have side effects of some form; otherwise, 
it wouldn't be able to provide value to the outside world.
```
To test early verswion printArgs methods, you'd need to redefine println, capture the output passed to it, and make sure it is what you expect.  
By contrast, you could test the formatArgs function simply by checking its result:

```
val res = formatArgs(Array("zero", "one", "two")) 
assert(res == "zero\none\ntwo")
```


Scala is not a pure functional language that forces you to program everything in the functional style. Scala is a hybrid imperative/functional language.

### 12.read lines from a file

```
import scala.io.Source
for (line <- Source.fromFile(args(0)).getLines()) 
    println(line.length + " " + line)
    
Source.fromFile(args(0)) attempts to returns a Source object
The getLines method returns an Iterator[String]
```

### 小结
Nothing


