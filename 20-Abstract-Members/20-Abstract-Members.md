# 20-Abstract-Members
You can declare abstract methods, fields and even abstract types of classes and traits. Theses are all called the members of a class or trait.

## 20.1 A QUICK TOUR OF ABSTRACT MEMBERS
An abstract type (T), method (transform), val (initial), and var (current)
```
trait Abstract { 
    type T 
    def transform(x: T): T 
    val initial: T 
    var current: T 
}
```

A concrete implementation

```
class Concrete extends Abstract { 
    type T = String 
    def transform(x: String) = x + x 
    val initial = "hi" 
    var current = initial 
}
```

## 20.2 TYPE MEMBERS
An abstract type in Scala is always a member of some class or trait.

One reason to use a type member is to define a short, descriptive alias for a type whose real name is more verbose, or less obvious in meaning, than the alias.

The other main use of type members is to declare abstract types that must be defined in subclasses.

## 20.3 ABSTRACT VALS
An abstract val constrains its legal implementation: Any implementation must be a val definition; it may not be a var or a def.

## 20.4 ABSTRACT VARS
If you declare an abstract var named hour, you implicitly declare an abstract getter method, hour, and an abstract setter method, hour_=.

## 20.5 INITIALIZING ABSTRACT VALS
Abstract vals sometimes like superclass parameters.

Traits don't have a constructor to which you could pass parameters. So the usual notion of parameterizing a trait works via abstract vals that are implemented in subclasses.

```
trait RationalTrait { 
    val numerArg: Int 
    val denomArg: Int 
}
To instantiate a concrete instance of that trait:
val test = new RationalTrait {
    val numerArg = 1
    val denomArg = 2
}
This expression yields an instance of an anonymous class 
that mixes in the trait and is defined by the body.
```
This particular anonymous class instantiation has an effect analogous to the instance creation new Rational(1, 2).But not perfect:

```
new Rational(expr1, expr2)
In Class, expr1 and expr2, are evaluated before class Rational is initialized,
so the values of expr1 and expr2 are available for the initialization of class Rational.

new RationalTrait { 
    val numerArg = expr1 
    val denomArg = expr2 
}
But in Trait, expr1 and expr2, are evaluated as part of the initialization of the anonymous class,
but the anonymous class is initialized after the RationalTrait.

so a selection of either value would yield the default value for type Int, 0.
trait RationalTrait {
    val numerArg: Int 
    val denomArg: Int 
    require(denomArg != 0)
    ...
}
scala> val x = 2 
x: Int = 2

scala> new RationalTrait { 
        val numerArg = 1 * x 
        val denomArg = 2 * x
        } 
java.lang.IllegalArgumentException: requirement failed
    at scala.Predef$.require(Predef.scala:207)
    at RationalTrait$class.$init$(<console>:10)
    ... 28 elided
```
Now you know abstract vals behave differently from parameters.

But Scala offers two alternative solutions: pre-initialized fields and lazy vals. So, a RationalTrait that can be initialized robustly, without fearing errors due to uninitialized fields.

**Pre-initialized fields**  
Usage:

```
anonymous classes
scala> new { 
        val numerArg = 1 * x 
        val denomArg = 2 * x 
       } with RationalTrait 
res1: RationalTrait = 1/2

objects
object twoThirds extends { 
    val numerArg = 2 
    val denomArg = 3 
} with RationalTrait

named subclasses
class RationalClass(n: Int, d: Int) extends { 
    val numerArg = n 
    val denomArg = d 
} with RationalTrait { 
    def + (that: RationalClass) = new RationalClass( 
        numer * that.denom + that.numer * denom, 
        denom * that.denom ) 
}
```
Because pre-initialized fields are initialized before the superclass constructor is called, their initializers cannot refer to the object that's being constructed.

```
scala> new { 
        val numerArg = 1 
        val denomArg = this.numerArg * 2 
       } with RationalTrait 
<console>:11: error: value numerArg is not a member of object $iw 
        val denomArg = this.numerArg * 2 
                            ^
```
**Lazy vals**  
The initializing expression on the right-hand side will only be evaluated the first time the val is used. (ps: lazy value can't be absract)

```
scala> object Demo { 
         lazy val x = { println("initializing x"); "done" } 
       } 
defined object Demo

scala> Demo 
res5: Demo.type = Demo$@5b1769c

scala> Demo.x 
initializing x 
res6: String = done
```

```
scala> trait LazyRationalTrait {
     |   val numerArg: Int
     |   lazy val numer = numerArg + g
     |   lazy val g = numerArg * 2
     |   override def toString = "numer: " + numer
     | }
defined trait LazyRationalTrait

scala> val x = 1
x: Int = 1

scala> new LazyRationalTrait {
     |    val numerArg = 1 * x
     | }
res3: LazyRationalTrait = numer: 3
```
The textual order of their definitions does not matter because values get initialized on demand.

This is cool as long as the initialization of lazy vals neither produces side effects nor depends on them. So lazy vals are an ideal complement to functional objects, where the order of initializations does not matter.

## 20.6 ABSTRACT TYPES
This chapter discusses what such an abstract type declaration means and what it's good for.

```
class Food
abstract class Animal {
  def eat(food: Food)
}
class Grass extends Food
class Cow extends Animal {
  override def eat(food: Food) = {} // This won't compile }
}
error: method eat overrides nothing
class Cow did not override the eat method in class Animal 
because its parameter type is different

class Food
abstract class Animal {
  type SuitableFood <: Food
  def eat(food: SuitableFood)
}
class Grass extends Food
class Cow extends Animal {
  type SuitableFood = Grass
  override def eat(food: Grass) = {} 
}
Now, any concrete instantiation of SuitableFood (in a subclass of Animal) 
must be a subclass of Food.

scala> class Fish extends Food 
defined class Fish

scala> val bessy: Animal = new Cow 
bessy: Animal = Cow@1515d8a6

scala> bessy eat (new Fish) 
<console>:14: error: type mismatch; 
    found : Fish 
    required: bessy.SuitableFood 
                bessy eat (new Fish) 
                            ^
```

## 20.7 PATH-DEPENDENT TYPES
See the error above, bessy.SuitableFood, this type consists of an object reference, bessy, followed by a type field, SuitableFood, of the object. So this shows that objects in Scala can have types as members.

A type like bessy.SuitableFood is called a path-dependent type.

The word "path" here means a reference an object.

## 20.8 REFINEMENT TYPES
Shit.
## 20.9 ENUMERATIONS
To create a new enumeration, define an object that extends scala.Enumeration.

```
object Color extends Enumeration { 
    val Red = Value 
    val Green = Value 
    val Blue = Value 
}
you can make it short
object Color extends Enumeration { 
    val Red, Green, Blue = Value 
}
```
Enumeration defines an inner class named Value,   
and the same-named parameterless Value method returns a fresh instance of that class.

A value such as Color.Red is of type Color.Value;   
Color.Value is the type of all enumeration values defined in object Color.

It's a path- dependent type,   
with Color being the path and Value being the dependent type.

```
object Direction extends Enumeration { 
    val North, East, South, West = Value 
}
```
Direction.Value would be different from Color.Value  
because the path parts of the two types differ.

Scala's Enumeration class also offers many other features:

```
object Direction extends Enumeration { 
    val North = Value("North") 
    val East = Value("East") 
    val South = Value("South") 
    val West = Value("West") 
}
scala> for (d <- Direction.values) print(d + " ") 
North East South West

scala> Direction.East.id 
res14: Int = 1

scala> Direction(1) 
res15: Direction.Value = East
```

## 20.10 CASE STUDY: CURRENCIES
Skipped.
## 20.11 CONCLUSION
This chapter has shown how to take advantage of abstract members.

When designing a class, make everything that is not yet known into an abstract member.


