# 10-Composition-and-Inheritance.md
We'll compare two fundamental relationships between classes: composition and inheritance.

we'll discuss  
abstract classes,  
parameterless methods,  
extending classes,  
overriding methods and fields,  
parametric fields,  
invoking superclass constructors,  
polymorphism and dynamic binding,  
final members and classes,  
and factory objects and methods.

### 10.1 A TWO-DIMENSIONAL LAYOUT LIBRARY
As a running example in this chapter,  
we'll create a library for building and rendering two-dimensional layout elements.

Each element will represent a rectangle filled with text.

Factory methods named "elem" that construct new elements from passed data.

```
elem(s: String): Element

val column1 = elem("hello") above elem("***") 
val column2 = elem("***") above elem("world") 
column1 beside column2

Printing the result of this expression would give you:

hello *** 
*** world
```
In this chapter, we'll define classes that enable element objects to be constructed from arrays, lines, and rectangles.

### 10.2 ABSTRACT CLASSES
The contents can be represented as an array of strings, where each string represents a line.

```
abstract class Element { 
    def contents: Array[String] 
}

The abstract modifier signifies that the class may have abstract members that do not have an implementation.

A class with abstract members must itself be declared abstract.

A method is abstract if it does not have an implementation.

You cannot instantiate an abstract class.
```

### 10.3 DEFINING PARAMETERLESS METHODS
We'll add methods to Element that reveal its width and height.

```
abstract class Element { 
    def contents: Array[String] 
    def height = contents.length 
    def width = if (height == 0) 0 else contents(0).length 
}

def width(): Int //empty-paren methods.

def width: Int //parameterless methods

The recommended convention is to use a parameterless method whenever there are no parameters 
and the method accesses mutable state only by reading fields of the containing object
```
we could implement width and height as fields

```
abstract class Element { 
    def contents: Array[String] 
    val height = contents.length 
    val width = if (height == 0) 0 else contents(0).length 
}

Field accesses might be slightly faster than method invocations, 
because field values are pre-computed when the class is initialized.
```
You can also leave off the empty parentheses on an invocation of any function that takes no arguments.

```
Array(1, 2, 3).toString 
"abc".length
```
However, it's still recommended to write the empty parentheses when the invoked method represents more than a property of its receiver object.

```
"hello".length // no () because no side-effect 
println() // better to not drop the ()

empty parentheses are appropriate if the method performs I/O, 
writes reassignable variables (vars), 
or reads vars other than the receiver's fields, 
either directly or indirectly by using mutable objects.

That way, the parameter list acts as a visual clue 
that some interesting computation is triggered by the call.
```
To summarize

Methods that take no parameters and have no side effects, write parameterless methods.

If you invoke a function that has side effects, be sure to include the empty parentheses when you write the invocation.

If the function you're calling performs an operation, use the parentheses.

### 10.4 EXTENDING CLASSES
We still need to be able to create new element objects.

We will need to create a subclass that extends Element and implements the abstract contents method.

```
abstract class Element { 
    def contents: Array[String] 
    def height = contents.length 
    def width = if (height == 0) 0 else contents(0).length 
}

class ArrayElement(conts: Array[String]) extends Element { 
    def contents: Array[String] = conts 
}

Class ArrayElement is called a subclass.
Element is called a superclass.

It makes the type ArrayElement a subtype of the type Element.

//In fact, the initializing value's type is ArrayElement.
val e: Element = new ArrayElement(Array("hello"))

It makes class ArrayElement inherit all non-private members from class Element.

scala> val ae = new ArrayElement(Array("hello", "world")) 
ae: ArrayElement = ArrayElement@39274bf7

scala> ae.width 
res0: Int = 5

Inheritance means that all members of the superclass are also members of the subclass.
two exceptions:
1.private members of the superclass 
2.a member with the same name and parameters, which is already implemented in the subclass, 
means overrides(implements) the member of the superclass.
```
If you leave out an extends clause, class Element implicitly extends class AnyRef.
![20170530149613169044577.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613169044577.png)

The relationship between ArrayElement andArray[String] is called composition.

Scala compiler will place into the binary class it generates for ArrayElement a field that holds a reference to the passed conts array.


### 10.5 OVERRIDING METHODS AND FIELDS
In Scala, fields and methods belong to the same namespace.

It's possible for a field to override a parameterless method.

```
abstract class Element { 
    def contents: Array[String] 
    def height = contents.length 
    def width = if (height == 0) 0 else contents(0).length 
}

class ArrayElement(conts: Array[String]) extends Element { 
    val contents: Array[String] = conts 
}
```
You can't define a field and method with the same name in the same class.

```
// This is Java 
class CompilesFine { 
    private int f = 0; 
    public int f() { 
        return 1; 
    } 
}

//This is Scala
class WontCompile { 
    private var f = 0 // Won't compile, because a field 
    def f = 1 // and method have the same name 
}
```
Java's four namespaces: 
fields, methods, types, and packages.

Scala's two namespaces:  
1.values (fields, methods, packages, and singleton objects)  
2.types (class and trait names)  

### 10.6 DEFINING PARAMETRIC FIELDS

```
class ArrayElement(conts: Array[String]) extends Element { 
    def contents: Array[String] = conts 
}

The parameter conts whose sole purpose is to be copied into the contents field.

This is a "code smell," 
a sign that there may be some unnecessary redundancy and repetition in your code.

You can avoid the code smell 
by combining the parameter and the field in a single parametric field definition.

class ArrayElement( 
    val contents: Array[String] 
) extends Element

Now, class ArrayElement defines a parameter and field with the same name, at the same time, which can be accessed from outside the class.
The field is initialized with the value of the parameter.
```

```
class Cat {
    val dangerous = false
}
class Tiger(
    override val dangerous: Boolean,
    private var age: Int 
) extends Cat

Tiger's definition is a shorthand as following:

class Tiger(param1: Boolean, param2: Int) extends Cat {
  override val dangerous = param1
  private var age = param2
}
```

### 10.7 INVOKING SUPERCLASS CONSTRUCTORS

```
abstract class Element { 
    def contents: Array[String] 
    def height = contents.length 
    def width = if (height == 0) 0 else contents(0).length 
}
class ArrayElement(conts: Array[String]) extends Element { 
    def contents: Array[String] = conts 
}
//class LineElement passes Array(s) to ArrayElement's primary constructor.
class LineElement(s: String) extends ArrayElement(Array(s)) {
  override val width = s.length
  override val height = 1
}

LineElement extends ArrayElement, 
and ArrayElement's constructor takes a parameter (an Array[String]), 
LineElement needs to pass an argument to the primary constructor of its superclass.

```
Now, the inheritance hierarchy for layout elements looks following:
![20170530149613536769621.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613536769621.png)

### 10.8 USING OVERRIDE MODIFIERS
The modifier override is optional if a member implements an abstract member with the same name.

The modifier override is required if a member override concrete definitions in parent class.

The modifier override is forbidden if a member does not override or implement some other member in parent class.

### 10.9 POLYMORPHISM AND DYNAMIC BINDING
A variable of type Element could refer to an object of type ArrayElement.  
This is called polymorphism, means "many shapes" or "many forms".

You now see two such forms of Element: ArrayElement and LineElement.

You can create more forms of Element by defining new Element subclasses.

```
class UniformElement( 
    ch: Char, 
    override val width: Int, 
    override val height: Int 
) extends Element { 
    private val line = ch.toString * width 
    def contents = Array.fill(height)(line) 
}

val e1: Element = new ArrayElement(Array("hello", "world")) 
val ae: ArrayElement = new LineElement("hello") 
val e2: Element = ae 
val e3: Element = new UniformElement('x', 2, 3)
```
The inheritance hierarchy for class Element now looks like:
![20171203151228838818448.png](http://op2oo1z8b.bkt.clouddn.com/20171203151228838818448.png)

Method invocations on variables and expressions are dynamically bound.

```
abstract class Element { 
    def demo() = { 
        println("Element's implementation invoked") 
    } 
}

class ArrayElement extends Element { 
    override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}

class LineElement extends ArrayElement { 
    override def demo() = { 
        println("LineElement's implementation invoked") 
    } 
}
class UniformElement extends Element

def invokeDemo(e: Element) = { 
    e.demo() 
}

scala> invokeDemo(new ArrayElement) 
ArrayElement's implementation invoked

scala> invokeDemo(new LineElement) 
LineElement's implementation invoked

scala> invokeDemo(new UniformElement) 
Element's implementation invoked

UniformElement does not override demo, it inherits the implementation of demo from its superclass.
```

### 10.10 DECLARING FINAL MEMBERS
You want to ensure that a member cannot be overridden by subclasses.

```
class ArrayElement extends Element { 
    final override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}
```
You may also at times want to ensure that an entire class not be subclassed.

```
final class ArrayElement extends Element { 
    override def demo() = { 
        println("ArrayElement's implementation invoked") 
    } 
}
```



### 10.11 USING COMPOSITION AND INHERITANCE
Composition,  
If what you're after is primarily code reuse.

Inheritance,  
1.whether it models an is-a relationship.  
2.whether clients will want to use the subclass type as a superclass type.

It's better to define LineElement as a direct subclass of Element.

```
class LineElement(s: String) extends Element { 
    val contents = Array(s) 
    override def width = s.length 
    override def height = 1 
}
```

The inheritance hierarchy for class Element now looks like:
![20170530149613841950053.png](http://op2oo1z8b.bkt.clouddn.com/20170530149613841950053.png)

### 10.12 IMPLEMENTING ABOVE, BESIDE, AND TOSTRING
With the addition of these three methods, class Element now looks like:

```
abstract class Element { 
    def contents: Array[String]
    
    def width: Int = 
        if (height == 0) 0 else contents(0).length
    
    def height: Int = contents.length
    
    def above(that: Element): Element = 
        new ArrayElement(this.contents ++ that.contents)
    
    def beside(that: Element): Element =
        new ArrayElement( 
            for ( 
                (line1, line2) <- this.contents zip that.contents 
            ) yield line1 + line2 
        )
    
    override def toString = contents mkString "\n"
}
```

### 10.13 DEFINING A FACTORY OBJECT
A factory object contains methods that construct other objects.

Clients would then use these factory methods to construct objects, rather than constructing the objects directly with new.

A straight-forward solution is to create a companion object of class Element and make this the factory object for layout elements.

```
object Element {
    def elem(contents: Array[String]): Element = 
        new ArrayElement(contents) 
    
    def elem(chr: Char, width: Int, height: Int): Element = 
        new UniformElement(chr, width, height) 
    
    def elem(line: String): Element =
        new LineElement(line)
}
```
We'll import Element.elem, so we can use the elem factory methods rather than creating newArrayElement instances explicitly.

```
import Element.elem 
abstract class Element {
    def contents: Array[String] 
    
    def width: Int = 
        if (height == 0) 0 else contents(0).length 
    
    def height: Int = contents.length 
    
    def above(that: Element): Element = 
        elem(this.contents ++ that.contents) 
        
    def beside(that: Element): Element = 
        elem( 
            for ( 
            (line1, line2) <- this.contents zip that.contents 
            ) yield line1 + line2 
        ) 
    override def toString = contents mkString "\n"
}
```
### 10.14 HEIGHTEN AND WIDEN
Now, we add widen and heighten.

These subclasses, ArrayElement, LineElement, andUniformElement, 
could now be private because they no longer need to be accessed directly by clients.

In Scala, you can define classes and singleton objects inside other classes and singleton objects.

```
object Element {
    private class ArrayElement( 
        val contents: Array[String] 
    ) extends Element
    
    private class LineElement(s: String) extends Element { 
        val contents = Array(s) 
        override def width = s.length 
        override def height = 1 
    }
    
    private class UniformElement( 
        ch: Char, 
        override val width: Int, 
        override val height: Int 
    ) extends Element { 
        private val line = ch.toString * width 
        def contents = Array.fill(height)(line) 
    }
    
    def elem(contents: Array[String]): Element = 
        new ArrayElement(contents)
    
    def elem(chr: Char, width: Int, height: Int): Element = 
        new UniformElement(chr, width, height)
    
    def elem(line: String): Element = 
        new LineElement(line)
}
```

```
import Element.elem 
abstract class Element { 
    def contents: Array[String]
    
    def width: Int = contents(0).length 
    def height: Int = contents.length
    
    def above(that: Element): Element = { 
        val this1 = this widen that.width 
        val that1 = that widen this.width 
        elem(this1.contents ++ that1.contents) 
    }
    
    def beside(that: Element): Element = { 
        val this1 = this heighten that.height 
        val that1 = that heighten this.height 
        elem( 
            for ((line1, line2) <- this1.contents zip that1.contents) 
                yield line1 + line2
        ) 
    }
    
    def widen(w: Int): Element =
        if (w <= width) this 
        else {
            val left = elem(' ', (w - width) / 2, height)
            val right = elem(' ', w - width - left.width, height)
            left beside this beside right 
        }
    
    def heighten(h: Int): Element =
        if (h <= height) this 
        else {
            val top = elem(' ', width, (h - height) / 2)
            val bot = elem(' ', width, h - height - top.height)
            top above this above bot 
        }

    override def toString = contents mkString "\n"
}
```
### 10.15 PUTTING IT ALL TOGETHER
Skipped
### 10.16 CONCLUSION
Skipped

