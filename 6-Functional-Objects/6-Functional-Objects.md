# 6-Functional-Objects.md
In this chapter, the emphasis is on classes that define functional objects, or objects that do not have any mutable state.

More aspects of object-oriented programming in Scala: class parameters and constructors, methods and operators, private members, overriding, checking preconditions, overloading, and self references.

### 6.1 A SPECIFICATION FOR CLASS RATIONAL
The Rational class, we'll design in this chapter must model the behavior of rational numbers, including allowing them to be added, subtracted, multiplied, and divided.

### 6.2 CONSTRUCTING A RATIONAL
We will start the design with this.

```
The identifiers n and d, are called class parameters.
class Rational(n: Int, d: Int)

The Scala compiler will gather up these two class parameters,
and create a primary constructor that takes the same two parameters.
```
If a class doesn't have a body, you don't need to specify empty curly braces.

**IMMUTABLE OBJECT TRADE-OFFS**  
Advantages    
1.immutable objects have less complex state spaces that change over time.  
2.pass immutable objects around quite freely.
3.no thread can change the state of an immutable.
4.immutable objects make safe hash table keys.

Disadvantages  
1.immutable objects require that a large object graph be copied, whereas an update could be done in its place. In some cases this can be awkward to express and might also cause a performance bottleneck.

In Java, classes have constructors, which can take parameters; whereas in Scala, classes can take parameters directly.  

Class parameters can be used directly in the body of the class.

```
Scala compiler would place the call to println into Rational's primary constructor.
class Rational(n: Int, d: Int) { 
    println("Created " + n + "/" + d)
}
scala> new Rational(1, 2) 
Created 1/2 
res0: Rational = Rational@2591e0c9
```

### 6.3 REIMPLEMENTING THE TOSTRING METHOD
By default, class Rational inherits the implementation oftoString defined in class java.lang.Object, which just prints the class name, an @ sign, and a hexadecimal number.

You can override the default implementation by adding a method toString to class Rational, like this:

```
class Rational(n: Int, d: Int) { 
    override def toString = n + "/" + d 
}
```

### 6.4 CHECKING PRECONDITIONS
A precondition is a constraint on values passed into a method or constructor, a requirement which callers must fulfill.

```
One way to do that is to use require
class Rational(n: Int, d: Int) { 
    require(d != 0) 
    override def toString = n + "/" + d 
}

If the passed value is true, require will return normally. 
Otherwise, require will prevent the object from being constructed by throwing an IllegalArgumentException.
```

### 6.5 ADDING FIELDS

```
// This won't compile 
class Rational(d: Int) { 
    require(d != 0) 
    override def toString = "" + d 
    def add(that: Rational): Rational = 
        new Rational(d * that.d) 
}
<console>:11: error: value d is not a member of Rational 
            new Rational(d * that.d) 
                                  ^
Won't let you say that.d because that does not refer to the Rational object on which add was invoked.  

you'll need to make d into fields.                          
class Rational(d: Int) { 
    require(d != 0) 
    val denom: Int = d
    override def toString = "/" + denom
    def add(that: Rational): Rational = 
        new Rational(denom * that.denom) 
}
besides, you can access d from outside
scala> val r = new Rational(2) 
r: Rational = 2

scala> r.denom res4: Int = 2
```
### 6.6 SELF REFERENCES
The keyword this refers to the object instance on which the currently executing method was invoked, or if used in a constructor, the object instance being constructed.

```
def lessThan(that: Rational) = 
    this.numer * that.denom < that.numer * this.denom
    
This.numer refers to the numer of the object on which lessThan was invoked.
this.numer equals to numer.
```

### 6.7 AUXILIARY CONSTRUCTORS
In Scala, constructors other than the primary constructor are called auxiliary constructors.

```
class Rational(n: Int, d: Int) {
    require(d != 0) 
    val numer: Int = n 
    val denom: Int = d 
    
    def this(n: Int) = this(n, 1) // auxiliary constructor 
    
    override def toString = numer + "/" + denom 
    
    def add(that: Rational): Rational =
        new Rational(
            numer * that.denom + that.numer * denom,
            denom * that.denom
    )
}
scala> val y = new Rational(3) 
y: Rational = 3/1
```
In Scala, every auxiliary constructor must invoke another constructor of the same class as its first action, So eventually will call the primary constructor of the class. 

The primary constructor is thus the single point of entry of a class.

In a Scala class, only the primary constructor can invoke a superclass constructor.

### 6.8 PRIVATE FIELDS AND METHODS

```
class Rational(n: Int, d: Int) {
    require(d != 0) 
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n 
    val denom: Int = d 
    
    def this(n: Int) = this(n, 1) // auxiliary constructor 
    
    override def toString = numer + "/" + denom 
    
    def add(that: Rational): Rational =
        new Rational(
            numer * that.denom + that.numer * denom,
            denom * that.denom
    )
    private def gcd(a: Int, b: Int): Int =
        if (b == 0) a else gcd(b, a % b)
}
g's initializer,gcd(n.abs, d.abs), will execute before the other two, 
because it appears first in the source.
```
The Scala compiler will place the code for the initializers of Rational's three fields into the primary constructor in the order in which they appear in the source code.

### 6.9 DEFINING OPERATORS

```
As + is a legal identifier in Scala, we can define a method with + as its name.
class Rational(n: Int, d: Int) {
    require(d != 0) 
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n 
    val denom: Int = d 
    
    def this(n: Int) = this(n, 1) // auxiliary constructor 
    
    def + (that: Rational): Rational =
        new Rational( 
            numer * that.denom + that.numer * denom, 
            denom * that.denom
    )
    
    override def toString = numer + "/" + denom 
    
    def add(that: Rational): Rational =
        new Rational(
            numer * that.denom + that.numer * denom,
            denom * that.denom
    )
    private def gcd(a: Int, b: Int): Int =
        if (b == 0) a else gcd(b, a % b)
}
scala> val x = new Rational(1, 2) 
x: Rational = 1/2

scala> val y = new Rational(2, 3) 
y: Rational = 2/3

scala> x + y 
res7: Rational = 7/6
```

### 6.10 IDENTIFIERS IN SCALA
Scala follows Java's convention of using camel-case identifiers, such as toString and HashSet.

Fields, method parameters, local variables, and functions should start with a lower case letter.

Classes and traits should start with an upper case letter.

In Java, give constants names that are all upper case, like PI,   
in scala, only the first character should be upper case, like Pi.

### 6.11 METHOD OVERLOADING

```
class Rational(n: Int, d: Int) {
    require(d != 0) 
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n 
    val denom: Int = d 
    
    def this(n: Int) = this(n, 1) // auxiliary constructor 
    
    def + (that: Rational): Rational =
        new Rational( 
            numer * that.denom + that.numer * denom, 
            denom * that.denom
    )
    def + (i: Int): Rational = 
        new Rational(numer + i * denom, denom)
        
    override def toString = numer + "/" + denom 
    
    def add(that: Rational): Rational =
        new Rational(
            numer * that.denom + that.numer * denom,
            denom * that.denom
    )
    private def gcd(a: Int, b: Int): Int =
        if (b == 0) a else gcd(b, a % b)
}
```
Each of + method names is overloaded because each name is now being used by multiple methods.

In a method call, the compiler picks the version of an overloaded method that correctly matches the types of the arguments.

### 6.12 IMPLICIT CONVERSIONS
If you place the implicit method definition inside class Rational, it won't be in scope in the interpreter. 

For now, you'll need to define it directly in the interpreter.

### 6.13 A WORD OF CAUTION
The goal you should keep in mind as you design libraries is not merely enabling concise client code, but readable, understandable client code.

### 6.14 CONCLUSION
Skipped


