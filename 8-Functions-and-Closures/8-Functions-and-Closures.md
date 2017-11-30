# 8-Functions-and-Closures.md
Besides methods, which are functions that are members of some object, there are also functions nested within functions, function literals, and function values.

### 8.1 METHODS
Define a function as a member of some object, is called a method.

### 8.2 LOCAL FUNCTIONS
Programs should be decomposed into many small functions that each do a well-defined task.

You can define functions inside other functions, called local functions, whose are visible only in their enclosing block. 

```
def processFile(filename: String, width: Int) = {
    def processLine(filename: String, width: Int, line: String) = {
    if (line.length > width) 
        println(filename + ": " + line.trim)
    }
    ...
}
```
Local functions can access the parameters of their enclosing function.

```
def processFile(filename: String, width: Int) = {
    def processLine(line: String) = { 
        if (line.length > width) 
            println(filename + ": " + line.trim) 
    }
    ...
}
```

### 8.3 FIRST-CLASS FUNCTIONS
Scala has first-class functions.

You define functions and call them, and write down functions as unnamed literals and then pass them around as values.

A function literal is compiled into a class that 
when instantiated at runtime is a function value.

Function literals exist in the source code, 
whereas function values exist as objects at runtime.

```
The => designates that this function converts the thing on the left (x) 
to the thing on the right (x + 1).

scala> (x: Int) => x + 1
res0: Int => Int = <function1>

Function values are objects, so you can store them in variables.

scala> var increase = (x: Int) => x + 1 
increase: Int => Int = <function1>

scala> increase(10) 
res0: Int = 11

have more than one statement in the function literal

scala> increase = (x: Int) => { 
        println("We") 
        println("are") 
        println("here!") 
        x + 1 
        } 
increase: Int => Int = <function1>

the value returned from the function is whatever results from evaluating the last expression.

scala> increase(10) 
We 
are 
here!
res2: Int = 11
```

```
Foreach method takes a function as an argument 
and invokes that function on each of its elements.

scala> val someNumbers = List(-11, -10, -5, 0, 5, 10) 
someNumbers: List[Int] = List(-11, -10, -5, 0, 5, 10)

scala> someNumbers.foreach((x: Int) => println(x)) 
-11 
-10 
-5 
0 
5
10

scala> someNumbers.filter((x: Int) => x > 0)
res4: List[Int] = List(5, 10)
```

### 8.4 SHORT FORMS OF FUNCTION LITERALS
Scala provides a number of ways to leave out redundant information and write function literals more briefly.

```
The Scala compiler knows that x must be an integer, 
because it sees that you are immediately using the function to filter a list of integers.
This is called target typing,
the targeted usage of an expression is allowed to influence the typing of that expression.

scala> someNumbers.filter((x) => x > 0) 
res5: List[Int] = List(5, 10)

you can also leave out parentheses around a parameter whose type is inferred.

scala> someNumbers.filter(x => x > 0) 
res6: List[Int] = List(5, 10)
```

### 8.5 PLACEHOLDER SYNTAX

```
If each parameter appears only one time within the function literal,
you can use underscores as placeholders for one or more parameters.

scala> someNumbers.filter(_ > 0) 
res7: List[Int] = List(5, 10)

This blank will be filled in with an argument to the function each time the function is invoked.

scala> val f = _ + _ 
<console>:7: error: missing parameter type for expanded 
function ((x$1, x$2) => x$1.$plus(x$2))
    val f = _ + _ 
              ^

scala> val f = (_: Int) + (_: Int) 
f: (Int, Int) => Int = <function2>

scala> f(5, 10) 
res9: Int = 15

Note that _ + _ expands into a literal for a function that takes two parameters.
Multiple underscores mean multiple parameters, not reuse of a single parameter repeatedly.
The first underscore represents the first parameter, the second underscore the second parameter, and so on.
```


### 8.6 PARTIALLY APPLIED FUNCTIONS
You can also replace an entire parameter list with an underscore.

```
When you use an underscore in this way, you are writing a partially applied function.

someNumbers.foreach(println _)
equals to
someNumbers.foreach(x => println(x))

You need to leave a space between the function name and the underscore
```

```
In Scala, when you invoke a function, passing in any needed arguments, 
you apply that function to the arguments.

scala> def sum(a: Int, b: Int, c: Int) = a + b + c 
sum: (a: Int, b: Int, c: Int)Int

You could apply the function sum to the arguments 1, 2, and 3

scala> sum(1, 2, 3) 
res10: Int = 6

A partially applied function is an expression in which you don't supply all of the arguments needed by the function.
Instead, you supply some, or none, of the needed arguments.

scala> val a = sum _ 
a: (Int, Int, Int) => Int = <function3>

scala> a(1, 2, 3) 
res11: Int = 6

scala> val b = sum(1, _: Int, 3) 
b: Int => Int = <function1>

scala> b(2) 
res13: Int = 6

Scala compiler instantiates a function value 
that takes the three integer parameters missing from the partially applied function expression, sum _, 
and assigns a reference to that new function value to the variable a.

The variable named a refers to a function value object, 
which is an instance of a class generated automatically by the Scala compiler from sum _.

The class generated by the compiler has an apply method that takes three arguments.

scala> a.apply(1, 2, 3) 
res12: Int = 6

Another way to think about this kind of expression, 
in which an underscore is used to represent an entire parameter list, 
is as a way to transform a def into a function value.
```

```
If you are writing a partially applied function expression in which you leave off all parameters,
you can express it more concisely by leaving off the underscore.

someNumbers.foreach(println _)

someNumbers.foreach(println)

This last form is allowed only in places where a function is required.

In situations where a function is not required, attempting to use this form will cause a compilation error.

scala> val c = sum 
<console>:8: error: missing arguments for method sum; 
follow this method with `_' if you want to treat it 
as a partially applied function 
    val c = sum 
            ^
```

### 8.7 CLOSURES

```
The resulting function value, 
which will contain a reference to the captured morevariable, is called a closure, 
means that the act of "closing" the function literal by "capturing" the bindings of its free variables.

scala> var more = 1 
more: Int = 1

scala> val addMore = (x: Int) => x + more 
addMore: Int => Int = <function1>

more is a free variable,
The x variable, is a bound variable.

scala> addMore(10) 
res16: Int = 11

(x: Int) => x + 1, is called a closed term
(x: Int) => x + more, is an open term, 
which requirs that a binding for its free variable, more, be captured.
```

```
Scala's closures capture variables themselves, not the value to which variables refer.

scala> more = 9999 
more: Int = 9999

scala> addMore(10) 
res17: Int = 10009

The same is true in the opposite direction.

scala> val someNumbers = List(-11, -10, -5, 0, 5, 10) 
someNumbers: List[Int] = List(-11, -10, -5, 0, 5, 10)

scala> var sum = 0 
sum: Int = 0

scala> someNumbers.foreach(sum += _)

scala> sum 
res19: Int = -11
```

### 8.8 SPECIAL FUNCTION CALL FORMS
Scala supports repeated parameters, named arguments, and default arguments.

**Repeated parameters**

```
Scala allows you to indicate that the last parameter to a function may be repeated.

scala> def echo(args: String*) = for (arg <- args) println(arg) 
echo: (args: String*)Unit

Defined this way, echo can be called with zero to many String arguments.

scala> echo()

scala> echo("one") 
one

scala> echo("hello", "world!") 
hello 
world!

The type of the repeated parameter is an Array of the declared type of the parameter.
"String*" is actually Array[String].

But you can't pass an array as a repeated parameter.

scala> val arr = Array("What's", "up", "doc?") 
arr: Array[String] = Array(What's, up, doc?)

scala> echo(arr) 
<console>:10: error: type mismatch; 
    found : Array[String] 
    required: String 
            echo(arr) 
                 ^
you can do that, this notation tells the compiler to pass each element of arr as its own argument to echo.

scala> echo(arr: _*) 
What's 
up 
doc?
```
**Named arguments**

```
Named arguments allow you to pass arguments to a function in a different order.

scala> def speed(distance: Float, time: Float): Float = distance / time 
speed: (distance: Float, time: Float)Float

scala> speed(time = 10, distance = 100) 
res29: Float = 10.0
```
**Default parameter values**

```
scala> def aTest(d1: Int = 10, d2: Int = 20) = d1 + d2
aTest: (d1: Int, d2: Int)Int

scala> aTest()
res4: Int = 30

scala> aTest(5)
res5: Int = 25

scala> aTest(d1 = 20)
res6: Int = 40

scala> aTest(d2 = 50)
res7: Int = 60

scala> aTest(d2 = 50, d1 = 50)
res8: Int = 100
```

### 8.9 TAIL RECURSION

```
def approximateLoop(initialGuess: Double): Double = { 
    var guess = initialGuess 
    while (!isGoodEnough(guess)) 
        guess = improve(guess) 
    guess 
}

def approximate(guess: Double): Double = 
    if (isGoodEnough(guess)) guess 
    else approximate(improve(guess))

A recursive solution is more elegant and concise than a loop-based one. 
If the solution is tail recursive, there won't be any runtime overhead to be paid.

Functions like approximate, which call themselves as their last action, are called tail recursive.
```
**Tracing tail-recursive functions**

A tail-recursive function will not build a new stack frame for each call; all calls will execute in a single frame.

```
def bang(x: Int): Int = {
 if (x == 0) 
        throw new Exception("bang!") 
    else 
        bang(x - 1)
}
   
scala> bang(5) 
java.lang.Exception: bang!
    at .bang(<console>:5) 
    at .<init>(<console>:6) ...

Sometimes, you'll be confused about tail recursive exception.
scala -g:notailcalls

scala> bang(5) 
java.lang.Exception: bang!
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .bang(<console>:5) 
    at .<init>(<console>:6) ...
```

**TAIL CALL OPTIMIZATION**

```
Both approximateLoop and approximate, 
are compiled down to the same thirteen instructions of Java bytecodes.

But Scala compiler does some optimization for tail recursive.

public double approximate(double);
    Code:
        0: aload_0 
        1: astore_3 
        2: aload_0 
        3: dload_1 
        4: invokevirtual #24; //Method isGoodEnough:(D)Z 
        7: ifeq 12 
        10: dload_1 
        11: dreturn 
        12: aload_0 
        13: dload_1 
        14: invokevirtual #27; //Method improve:(D)D 
        17: dstore_1 
        18: goto 2
```

**Limits of tail recursion**  
The use of tail recursion in Scala is fairly limited because the JVM instruction set makes implementing more advanced forms of tail recursion very difficult.

```
You also won't get a tail-call optimization if the final call goes to a function value.

val funValue = nestedFun _ 
def nestedFun(x: Int) : Unit = { 
    if (x != 0) { println(x); funValue(x - 1) } 
}

Tail-call optimization is limited to situations 
where a method or nested function calls itself directly as its last operation, 
without going through a function value or some other intermediary.
```

### 8.10 CONCLUSION
Skipped



