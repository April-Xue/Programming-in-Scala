# 11-Scala's-Hierarchy.md
### 11.1 scala's class hierarchy
![20171204151236883039474.png](http://op2oo1z8b.bkt.clouddn.com/20171204151236883039474.png)

At the top of the hierarchy is class Any, which defines methods that include the following:

```
final def ==(that: Any): Boolean 
final def !=(that: Any): Boolean 
def equals(that: Any): Boolean 
def ##: Int 
def hashCode: Int def toString: String
```

Every class inherits from Any.

The root class Any has two subclasses: AnyVal and AnyRef.

**AnyVal**

AnyVal is the parent class of value classes, there are nine value classes:  
Byte, Short, Char, Int, Long, Float, Double, Boolean, and Unit.

The instances of these classes are all written as literals in Scala.  
You can't create instances of these classes using new.

```
So if you were to write:
scala> new Int

you would get:
<console>:5: error: class Int is abstract; cannot be instantiated
            new Int 
            ^
```

But, unit has a single instance value, which is written ().

The value classes support the usual arithmetic and boolean operators as methods.

```
scala> 42.toString 
res1: String = 42

scala> 42.hashCode 
res2: Int = 42

scala> 42 equals 42 
res3: Boolean = true
```

All value classes are subtypes of scala.AnyVal, but they do not subclass each other.

But, there are implicit conversions between different value class types.

Implicit conversions are also used to add more functionality to value types.  
Like implicit conversion from class Int to RichInt.

```
scala> 42 max 43 
res4: Int = 43

scala> 42 min 43 
res5: Int = 42

scala> 3.abs 
res8: Int = 3

scala> (-3).abs 
res9: Int = 3
```
**AnyRef**

AnyRef is the base class of all reference classes in Scala.

On the Java platform, AnyRef is in fact just an alias for class java.lang.Object.


### 11.2 how primitives are implemented
Integers of type Int are converted transparently to "boxed integers" of type java.lang.Integer whenever necessary.

Standard operations like addition or multiplication are implemented as primitive operations.

```
// This is Java 
boolean isEqual(int x， int y) { 
    return x == y; 
} 
System.out.println(isEqual(421， 421));
true

// This is Java 
boolean isEqual(Integer x， Integer y) { 
    return x == y; 
} 
System.out.println(isEqual(421， 421));
false

In Java, 
The number 421 gets boxed twice, so that the arguments for x and y are two different objects.

Because == means reference equality on reference types, and Integer is a reference type, the result is false.

There is a difference between primitive types and reference types.
```

```
scala> def isEqual(x: Int， y: Int) = x == y 
isEqual: (x: Int， y: Int)Boolean

scala> isEqual(421， 421) 
res10: Boolean = true

scala> def isEqual(x: Any， y: Any) = x == y 
isEqual: (x: Any， y: Any)Boolean

scala> isEqual(421， 421)
res11: Boolean = true

The equality operation == in Scala is designed to be transparent with respect to the type's representation.

For value types, it is the natural (numeric or boolean) equality.

For reference types other than Java's boxed numeric types, == is treated as an alias of the equals method inherited from Object.

scala> val x = "abcd".substring(2) 
x: String = cd

scala> val y = "abcd".substring(2) 
y: String = cd

scala> x == y 
res12: Boolean = true
```
You would like to hash cons with some classes and compare their instances with reference equality.

```
scala> val x = new String("abc") 
x: String = abc

scala> val y = new String("abc") 
y: String = abc

scala> x == y 
res13: Boolean = true

//eq is in class AnyRef
scala> x eq y 
res14: Boolean = false

scala> x ne y 
res15: Boolean = true
```

### 11.3 bottom types
At the bottom of the type hierarchy classes scala.Null and scala.Nothing.

Class Null is the type of the null reference.

Type Nothing is a subtype of every other type.

```
scala> val i: Int = null 
<console>:7: error: an expression of type Null is ineligible 
for implicit conversion
    val i: Int = null 
                 ^
```
Usage of Nothing:

```
def error(message: String): Nothing = 
    throw new RuntimeException(message)
    
The return type of error is Nothing, which tells users that the method will not return normally (it throws an exception instead).

def divide(x: Int， y: Int): Int =
    if (y != 0) x / y 
    else error("can't divide by zero")
    
Because Nothing is a subtype of Int, the type of the whole conditional is Int, as required.
```

### 11.4 defining your own value classes
You can define your own value classes to augment the ones that are built in.

For a class to be a value class, it must have exactly one parameter and it must have nothing inside it except defs.

No other class can extend a value class, and a value class can't redefine equals or hashCode.

To define a value class, make it a subclass of AnyVal, and put val before the one parameter.

```
class Dollars(val amount: Int) extends AnyVal { 
    override def toString() = "$" + amount 
}

scala> val money = new Dollars(1000000) 
money: Dollars = $1000000 

scala> money.amount 
res16: Int = 1000000
```

### 11.5 小结
Skipped.



