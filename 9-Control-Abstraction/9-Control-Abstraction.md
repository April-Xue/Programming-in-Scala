# 9-Control-Abstraction.md
We'll show you how to apply function values to create new control abstractions.

You'll also learn about currying and by-name parameters.

### 9.1 REDUCING CODE DUPLICATION
Functions that take functions as parameters, are called higher-order functions.

One benefit of higher-order functions is they enable you to create control abstractions that allow you to reduce code duplication.

```
object FileMatcher { 
    private def filesHere = (new java.io.File(".")).listFiles
    
    def filesEnding(query: String) =
        for (file <- filesHere; if file.getName.endsWith(query)) 
            yield file
            
    def filesContaining(query: String) =
        for (file <- filesHere; if file.getName.contains(query)) 
            yield file    
}
```
You can use function values.

```
object FileMatcher { 
    private def filesHere = (new java.io.File(".")).listFiles
    
    def filesMatching(query: String, matcher: (String, String) => Boolean) = {
        for (file <- filesHere; if matcher(file.getName, query)) 
            yield file
    }
    
    def filesEnding(query: String) = 
        filesMatching(query, _.endsWith(_))
        
    def filesContaining(query: String) = 
        filesMatching(query, _.contains(_))
}

procedure：
(fileName: String, query: String) => fileName.endsWith(query)

same type String, leave off String
(fileName, query) => fileName.endsWith(query)

fileName and query are each used only once in the body of the function,
(fileName, query) => _.endsWith(_).

_.endsWith(_) uses two bound variables, no free variables.
```
The query gets passed to filesMatching, but filesMatching does nothing with it,   
just pass it to matcher, this is unnecessary,   
because the caller already knew the query to begin with!
you can leave off remove the query parameter.

```
object FileMatcher { 
    private def filesHere = (new java.io.File(".")).listFiles
    
    private def filesMatching(matcher: String => Boolean) = 
        for (file <- filesHere; if matcher(file.getName)) 
            yield file
    
    def filesEnding(query: String) = 
        filesMatching(_.endsWith(query))
    
    def filesContaining(query: String) = 
        filesMatching(_.contains(query))
}
_.endsWith(query),
contains one bound variable, the argument represented by the underscore, 
and one free variable named query.
```
At last, the cool thing is file.getName.endsWith(query).

### 9.2 SIMPLIFYING CLIENT CODE

```
def containsNeg(nums: List[Int]): Boolean = { 
    var exists = false 
    for (num <- nums) 
        if (num < 0) 
            exists = true 
    exists 
}

you can just call the higher-order function exists

def containsNeg(nums: List[Int]) = nums.exists(_ < 0)
```


### 9.3 CURRYING
A curried function is applied to multiple argument lists, instead of just one.

```
scala> def plainOldSum(x: Int, y: Int) = x + y 
plainOldSum: (x: Int, y: Int)Int

scala> plainOldSum(1, 2) 
res4: Int = 3

use currying

scala> def curriedSum(x: Int)(y: Int) = x + y 
curriedSum: (x: Int)(y: Int)Int

scala> curriedSum(1)(2) 
res5: Int = 3

you actually get two traditional function invocations back to back.
```

```
You can use the placeholder notation to usecurriedSum in a partially applied function expression.

scala> val onePlus = curriedSum(1)_ 
onePlus: Int => Int = <function1>

scala> onePlus(2) 
res7: Int = 3

The underscore in curriedSum(1)_ is a placeholder for the second parameter list.
```

### 9.4 WRITING NEW CONTROL STRUCTURES
You can make new control structures, by create methods that take functions as arguments.

```
scala> def twice(op: Double => Double, x: Double) = op(op(x)) 
twice: (op: Double => Double, x: Double)Double

scala> twice(_ + 1, 5) 
res9: Double = 7.0

Double => Double means takes one Double as an argument and returns another Double.
```
If a control pattern repeated in multiple parts of your code, 
you should implement it as a new control structure.

```
def withPrintWriter(file: File, op: PrintWriter => Unit) = { 
    val writer = new PrintWriter(file) 
    try { 
        op(writer) 
    } finally { 
        writer.close() 
    } 
}

withPrintWriter( 
    new File("date.txt"), 
    writer => writer.println(new java.util.Date) 
)
```

```
You can make the client code look a bit more like a built-in control structure.

In any method invocation in Scala in which you're passing in exactly one argument, 
you can use curly braces instead of parentheses.

def withPrintWriter(file: File)(op: PrintWriter => Unit) = { 
    val writer = new PrintWriter(file) 
    try { 
        op(writer) 
    } finally { 
        writer.close() 
    } 
}

val file = new File("date.txt")

withPrintWriter(file) { writer => 
    writer.println(new java.util.Date) 
}

This curly braces technique will work, only if you're passing in one argument.

This can make a method call feel more like a control abstraction.
```
Trick: currying + curly braces

### 9.5 BY-NAME PARAMETERS
If you want to implement something more like if or while, where there is no value to pass into the curly braces? Using by-name parameters.

```
def addByValue(a: Int, b: Int) = a + b

To make a by-name parameter

def addByName(a: Int, b: => Int) = a + b

The by-name type, Int is left out, is only allowed for parameters.
  
  addByValue(2, 2 + 2)  
->addByValue(2, 4)  
->2 + 4  
->6

  addByName(2, 2 + 2)  
->2 + (2 + 2)  
->2 + 4  
->6  

difference between two approaches:
2 + 2 is evaluated before the call to addByValue.
2 + 2 is not evaluated before the call to addByName,
Instead a function value will be created whose apply method will evaluate 2 + 2,
and this function value will be passed to addByName.
```

### 9.6 CONCLUSION
You can use functions within your code to factor out common control patterns.

You can take advantage of higher-order functions in the Scala library.

We discussed how to use currying and by-name parameters so that your own higher-order functions can be used with a concise syntax.

