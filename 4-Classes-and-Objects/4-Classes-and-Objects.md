# 4-Classes-and-Objects.md


### 4.1 classes, fields, and methods
A class is a blueprint for objects.  
You can create objects from the class blueprint with the keyword new.

```
class ChecksumAccumulator { 
    // class definition goes here 
}

new ChecksumAccumulator
```
You place fields and methods, which are collectively called members.  

Fields are variables that refer to objects.

When you instantiate a class, the runtime sets aside some memory to hold the image of that object's state (i.e., the content of its variables).

```
class ChecksumAccumulator { 
    var sum = 0 
}
val acc = new ChecksumAccumulator 
val csa = new ChecksumAccumulator
```
The image of the objects in memory might look like this:  
![20171128151183796247262.png](http://op2oo1z8b.bkt.clouddn.com/20171128151183796247262.png)

```
acc.sum = 3
```
![20171128151183794238685.png](http://op2oo1z8b.bkt.clouddn.com/20171128151183794238685.png)


```
// Won't compile, because acc is a val 
acc = new ChecksumAccumulator
```
acc will always refer to the same ChecksumAccumulatorobject with which you initialize it,   
but the fields contained inside that object might change over time.

private fields can only be accessed by methods defined in the same class  
and method parameters in Scala is that they are vals, not vars.

```
class ChecksumAccumulator { 
    private var sum = 0 
    def add(b: Byte): Unit = { 
        b = 1 // This won't compile, because b is a val
        sum += b
    }
}
val acc = new ChecksumAccumulator 
acc.sum = 5 // Won't compile, because sum is private
```
Public is Scala's default access level, don't have to write public.

In the absence of any explicit return statement, a Scala method returns the last value computed by the method.

The recommended style for methods is in fact to avoid having explicit, and especially multiple, return statements.

You can leave off the curly braces if a method computes only a single result expression.

```
class ChecksumAccumulator { 
    private var sum = 0 
    def add(b: Byte): Unit = sum += b 
}
```
Readers of the code may need to mentally infer the result types by studying the bodies of the methods, so better to explicitly provide the result types.

Methods with a result type of Unit are executed for their side effects.  
A side effect is generally defined as mutating state somewhere external to the method or performing an I/O action.

In add's case, the side effect is that sum is reassigned.  
A method that is executed only for its side effects is known as a procedure.

### 4.2 semicolon inference

```
x 
+ y
This parses as two statements x and +y.
you should wrap it in parentheses
(x 
+ y)

that is ok
x + 
y + 
z
it is a common Scala style to put the operators at the end of the line 
instead of the beginning
```

**THE RULES OF SEMICOLON INFERENCE**

```
A line ending is treated as a semicolon 
unless one of the following conditions is true:
1. The line in question ends in a word that would not be legal 
    as the end of a statement, such as a period or an infix operator.

2. The next line begins with a word that cannot start a statement.

3. The line ends while inside parentheses (...) or brackets [...], 
    because these cannot contain multiple statements anyway.
```

### 4.3 singleton objects
Scala cannot have static members. Instead, Scala has singleton objects.

When a singleton object shares the same name with a class, it is called that class's companion object.

You must define both the class and its companion object in the same source file.

The class is called the companion class of the singleton object.

A class and its companion object can access each other's private members.

```
object ChecksumAccumulator {
    private val cache = ...
    def calculate(s: String): Int = ...
}
invoke method just like that:
ChecksumAccumulator.calculate("Every value is an object.")
```
Given just a definition of object ChecksumAccumulator, you can't make a variable of type ChecksumAccumulator.

Singleton objects extend a superclass and can mix in traits.

Singleton objects cannot take parameters, whereas classes can.

You can't instantiate a singleton object with the new keyword.

Each singleton object is implemented as an instance of a synthetic class referenced from a static variable.

A singleton object that does not share the same name with a companion class is called a standalone object.
### 4.4 a scala application
To run a Scala program,  
you must supply the name of a standalone singleton object,  
with a main method that takes one parameter,  
an Array[String], and has a result type of Unit.

```
// In file Summer.scala 
import ChecksumAccumulator.calculate

object Summer { 
    def main(args: Array[String]) = { 
        for (arg <- args) 
            println(arg + ": " + calculate(arg)) 
    } 
}
```
Scala implicitly imports members of packages java.lang and scala,  
as well as the members of a singleton object named Predef, into every Scala source file.

Java requires you to put a public class in a file named after the class.  
put class SpeedRacer in file SpeedRacer.java.  
Scala can name .scala files anything you want.

Summer.scala are not scripts, because they end in a definition.
A script must end in a result expression.

You'll need to actually compile these files with the Scala compiler,  
then run the resulting class files.

```
$ scalac ChecksumAccumulator.scala Summer.scala
scalac is late,
every time the compiler starts up, 
it spends time scanning the contents of jar files and doing other initial work.

$ fsc ChecksumAccumulator.scala Summer.scala
The first time you run fsc, it will create a local server daemon
in next time,
fsc will simply send the file list to the daemon, 
which will immediately compile the files.

fsc -shutdown.

Running either of these scalac or fsc commands will produce Java class files.
Now, you can run it just by giving it the name of a standalone object.
$ scala Summer args0 args1
```

### 4.5 the app trait

```
import ChecksumAccumulator.calculate 

object FallWinterSpringSummer extends App {
    for (season <- List("fall", "winter", "spring"))
        println(season + ": " + calculate(season)) 
}
```
You may don't have to write main method, by extend a trait scala.App.

### 4.6 小结
Skipped

