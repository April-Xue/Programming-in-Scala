# 12-Traits.md
Traits are a fundamental unit of code reuse in Scala.

### 12.1 how trait work

```
trait Philosophical { 
    def philosophize() = { 
        println("I consume memory, therefore I am!") 
    } 
}

it has the default superclass of AnyRef.

it can be mixed in to a class using either the extends or with keywords.

Methods inherited from a trait can be used just like methods inherited from a superclass.

scala> val frog = new Frog 
frog: Frog = green

scala> frog.philosophize()
I consume memory, therefore I am!

A trait also defines a type.

scala> val phil: Philosophical = frog 
phil: Philosophical = green

scala> phil.philosophize() 
I consume memory, therefore I am!
```
Scala's syntax looks exactly the same, with only two exceptions:  
1.a trait cannot have any "class" parameters.  
2.in classes, super calls are statically bound, in traits, they are dynamically bound.

```
class Point(x: Int, y: Int)
trait NoPoint(x: Int, y: Int) // Does not compile
```
The method implementation to invoke for the super call is undefined when you define the trait.

### 12.2 thin versus rich interfaces
One major use of traits is to automatically add methods to a class in terms of methods the class already has.

Thin versus rich interfaces represents trade- off between the implementers and the clients of an interface.

### 12.3 example: rectangular objects
Skipped

### 12.4 the ordered trait
You replace all of the individual comparison methods with a single compare method, which defines <, >, <=, and >= for you in terms of this one method.

```
class Rational(n: Int, d: Int) extends Ordered[Rational] { 
    // ...
    def compare(that: Rational) = 
        (this.numer * that.denom) - (that.numer * this.denom) 
}

Beware that the Ordered trait does not define equals for you.
```

### 12.5 traits as stackable modifications

```
abstract class IntQueue { 
    def get(): Int 
    def put(x: Int) 
}

trait Doubling extends IntQueue { 
    abstract override def put(x: Int) = { super.put(2 * x) } 
}

The abstract override, implement stackable modifications.
two funny things going on:
1.it declares a superclass, means that 
the trait can only be mixed into a class that also extends IntQueue.
2.the super call will work so long as 
the trait is mixed in after another trait or class that gives a concrete definition to the method.
```

### 12.6 why not multiple inheritance
With multiple inheritance, the method called by a supercall can be determined right where the call appears.

With traits, the method called is determined by a linearization of the classes and traits that are mixed into a class.

```
class Animal 
trait Furry extends Animal 
trait HasLegs extends Animal 
trait FourLegged extends HasLegs 
class Cat extends Animal with Furry with FourLegged
```
![20171205151244299277959.png](http://op2oo1z8b.bkt.clouddn.com/20171205151244299277959.png)


```
Type        Linearization 
Animal      Animal -> AnyRef -> Any
Furry       Furry -> Animal -> AnyRef -> Any
FourLegged  FourLegged -> HasLegs -> Animal -> AnyRef -> Any
HasLegs     HasLegs -> Animal -> AnyRef -> Any
Cat         Cat -> FourLegged -> HasLegs -> Furry -> Animal -> AnyRef -> Any
```

### 12.7 to trait or not to trait
Sometimes, you will have to decide whether you want to use a trait or an abstract class.

```
If the behavior will not be reused, then make it a concrete class.

If it might be reused in multiple, unrelated classes, make it a trait.

If you want to inherit from it in Java code, use an abstract class.

If you plan to distribute it in compiled form, 
and you expect outside groups to write classes inheriting from it, 
you might lean towards using an abstract class.

If you still do not know, after considering the above, then start by making it as a trait.
```

### 12.8 CONCLUSION
Skipped

