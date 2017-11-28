# 2-First-Steps-in-Scala.md


### step 1.learn to use the scala interpreter

```
$ scala 
Welcome to Scala version 2.11.7 
Type in expressions to have them evaluated. 
Type :help for more information.

scala>

After you type an expression, such as 1 + 2, and hit enter:

scala> 1 + 2
res0: Int = 3

scala> res0 * 3 
res1: Int = 9
```

### step 2.define some variables
A val is similar to a final variable in Java.  
Once initialized, a val can never be reassigned. 

A var, by contrast, is similar to a non-final variable in Java.  
Can be reassigned throughout its lifetime.

```
scala> val msg = "Hello, world!" 
msg: String = Hello, world!

Scala strings are implemented by Java's String class.
scala> val msg2: java.lang.String = "Hello again, world!"
msg2: String = Hello again, world!
```
You can see, String doesn't appear anywhere in the val definition.  
This is type inference, Scala's ability to figure out types you leave off. I

### step 3.define some functions

```
scala> def max(x: Int, y: Int): Int = { 
            if (x > y) x 
            else y 
        } 
max: (x: Int, y: Int)Int
```
A type annotation must follow every function parameter, preceded by a colon,
because the Scala compiler does not infer function parameter types.

![20171127151177978581610.png](http://op2oo1z8b.bkt.clouddn.com/20171127151177978581610.png)

If the function is recursive, you must explicitly specify the function's result type.

In the case of max, you can leave the result type off and the compiler will infer it.Also, if a function consists of just one statement, you can optionally leave off the curly braces.

```
scala> def max(x: Int, y: Int) = if (x > y) x else y 
max: (x: Int, y: Int)Int
```

You can call it by name

```
scala> max(3, 5) 
res4: Int = 5
```

```
scala> def greet() = println("Hello, world!") 
greet: ()Unit
```
Unit is greet's result type.  
A result type of Unit indicates the function returns no interesting value. Scala's Unit type is similar to Java's void type

If you wish to exit the interpreter, use :quit or :q.

```
scala> :quit 
$
```

### step 4.write some scala scripts
Put this into a file named hello.scala:

```
println("Hello, world, from a script!")
```
run:

```
$ scala hello.scala
```

You get:

```
Hello, world, from a script!
```



### step 5.loop with while; decide with if

```
var i = 0 
while (i < args.length) {
    println(args(i))
    i += 1 
}
```
Java's + +i and i++ don't work in Scala.  
In Scala, you need to say either i = i + 1 or i += 1.

Note that in Scala, as in Java, you must put the boolean expression for a while or an if in parentheses.

If a block has only one statement, you can optionally leave off the curly braces.

### step 6.iterate with foreach and for
For now, you were programming in an imperative style:   
You give one imperative command at a time, iterate with loops, and often mutate state shared between different functions.

As you get to know Scala better, you'll likely often find yourself programming in a more functional style.

One of the main characteristics of a functional language is that functions are first class constructs.

```
args.foreach(arg => println(arg))
```
If a function literal consists of one statement that takes a single argument, you need not explicitly name and specify the argument.

```
args.foreach(println)
```
![20171127151178095124039.png](http://op2oo1z8b.bkt.clouddn.com/20171127151178095124039.png)

```
for (arg <- args)
    println(arg)
```
To the left of <- is "arg", the name of a val, not a var.  
For each element of the args array, a new arg val will be created and initialized to the element value, and the body of the for will be executed

### 小结
In this chapter, you learned some Scala basics.


