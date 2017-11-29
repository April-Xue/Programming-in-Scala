# 7-Built-in-Control-Structures.md
The only control structures are if,while, for, try, match, and function calls.



### 7.1 IF EXPRESSIONS

```
var filename = "default.txt" 
if (!args.isEmpty) 
    filename = args(0)
```

```
Scala's if is an expression that results in a value.
val filename =
    if (!args.isEmpty) args(0) 
    else "default.txt"
```
Using a val is the functional style.


### 7.2 WHILE LOOPS
while

```
def gcdLoop(x: Long, y: Long): Long = { 
    var a = x 
    var b = y 
    while (a != 0) { 
        val temp = a 
        a = b % a 
        b = temp 
    } 
    b
}
```

do-while

```
var line = "" 
do {
    line = readLine()
    println("Read: " + line) 
} while (line != "")
```

The while and do-while constructs are called "loops," not expressions,  
because they don't result in an interesting value. The type of the result is Unit.

Unit is called the unit value and is written ().

```
scala> def greet() = { println("hi") } 
greet: ()Unit

scala> () == greet() 
hi 
res0: Boolean = true
```
In Scala assignment always results in the unit value, ().

```
var line = "" 
while ((line = readLine()) != "") // This doesn't work!
    println("Read: " + line)
```

### 7.3 FOR EXPRESSIONS
**Iteration through collections**

```
val filesHere = (new java.io.File(".")).listFiles
for (file <- filesHere) 
    println(file)

With the "file <- filesHere" syntax, which is called a generator.
In each iteration, a new val named file is initialized with an element value.
```

```
// Not common in Scala...
for (i <- 0 to filesHere.length - 1) 
    println(filesHere(i))
    
//include upper bound
for (i <- 1 to 4) 
    println("Iteration " + i) 
Iteration 1 
Iteration 2 
Iteration 3 
Iteration 4

//not include upper bound
for (i <- 1 until 4) 
    println("Iteration " + i) 
Iteration 1 
Iteration 2 
Iteration 3
```

**Filtering**

```
for (file <- filesHere)
    if (file.getName.endsWith(".scala")) 
        println(file)     

for (file <- filesHere if file.getName.endsWith(".scala")) 
    println(file)
    
for ( 
    file <- filesHere 
    if file.isFile 
    if file.getName.endsWith(".scala") 
) println(file)
```

**Nested iteration**

```
def grep(pattern: String) =
    for ( 
        file <- filesHere 
        if file.getName.endsWith(".scala"); 
        line <- fileLines(file) 
        if line.trim.matches(pattern) 
    ) println(file + ": " + line.trim)

you can use curly braces instead of parentheses,
by doing so, you can leave off some of the semicolons.
def grep(pattern: String) =
    for {
      file <- filesHere
      if file.getName.endsWith(".scala")
      line <- fileLines(file)
      if line.trim.matches(pattern)
    } println(file + ": " + line.trim)

```
**Mid-stream variable bindings**

```
The bound variable trimmed just like a val, only with the val keyword left out.
def grep(pattern: String) =
for { 
    file <- filesHere 
    if file.getName.endsWith(".scala") 
    line <- fileLines(file) 
    trimmed = line.trim 
    if trimmed.matches(pattern) 
} println(file + ": " + trimmed)
```
**Producing a new collection**

```
The syntax of a for-yield expression is like this:
for clauses yield body

for (file <- filesHere if file.getName.endsWith(".scala")) { 
    yield file // Syntax error!
}
Even if the body is a block surrounded by curly braces, 
put the yield before the first curly brace
```

### 7.4 EXCEPTION HANDLING WITH TRY EXPRESSIONS
**Throwing exceptions**

```
val half =
    if (n % 2 == 0) 
        n / 2 
    else
        throw new RuntimeException("n must be even")
```
An exception throw has type Nothing.

**Catching exceptions**

```
import java.io.FileReader 
import java.io.FileNotFoundException 
import java.io.IOException

try { 
    val f = new FileReader("input.txt") 
    // Use and close file 
    } 
catch { 
    case ex: FileNotFoundException => // Handle missing file 
    case ex: IOException => // Handle other I/O error 
}
each catch clause is tried in turn.
```

**The finally clause**  
With a finally clause if you want to cause some code to execute no matter how the expression terminates.

```
val file = new FileReader("input.txt") 
    try {
        // Use the file 
    } finally {
        file.close() // Be sure to close the file 
}
```

**Yielding a value**  
try-catch-finally results in a value.

```
def urlFor(path: String) =
try { 
    new URL(path) 
} catch {
    case e: MalformedURLException =>
        new URL("http://www.scala-lang.org") 
}
no exception, result = try clause
throw exception, result = catch clause
```
If an exception is thrown but not caught, the expression has no result.

```
if a finally clause includes an explicit return statement, or throws an exception, 
that return value or exception will "overrule" any previous one 
that originated in the try block or one of its catch clauses.

def f(): Int = try return 1 finally return 2
results in 2
def g(): Int = try 1 finally 2
results in 1

it's usually best to avoid returning values from finally clauses
```

### 7.5 MATCH EXPRESSIONS

```
val firstArg = if (args.length > 0) args(0) else ""
firstArg match { 
    case "salt" => println("pepper") 
    case "chips" => println("salsa") 
    case "eggs" => println("bacon") 
    case _ => println("huh?") 
}
The default case is specified with an underscore (_), 
a wildcard symbol frequently used in Scala as a placeholder for a completely unknown value.
```
There are no breaks at the end of each alternative, Instead the break is implicit.

```
val firstArg = if (!args.isEmpty) args(0) else ""
val friend =
    firstArg match { 
        case "salt" => "pepper" 
        case "chips" => "salsa" 
        case "eggs" => "bacon" 
        case _ => "huh?" 
    }
println(friend)
```
Match expressions result in a value.

### 7.6 LIVING WITHOUT BREAK AND CONTINUE

```
int i = 0; 
boolean foundIt = false; 
while (i < args.length) {
    // This is Java
    if (args[i].startsWith("-")) { 
        i = i + 1; 
     continue; 
    } 
    if (args[i].endsWith(".scala")) {
     foundIt = true;
        break; 
    } 
    i = i + 1;
}
```
Better to replace every continue by an if and every break by a boolean variable.

```
var i = 0 
var foundIt = false
while (i < args.length && !foundIt) {
    if (!args(i).startsWith("-")) {
        if (args(i).endsWith(".scala")) 
            foundIt = true 
    } 
    i = i + 1 
}
```
Another way is rewriting the loop as a recursive function.

```
def searchFrom(i: Int): Int =
    if (i >= args.length) -1 
    else if (args(i).startsWith("-")) searchFrom(i + 1) 
    else if (args(i).endsWith(".scala")) i 
    else searchFrom(i + 1)

val i = searchFrom(0)
```
Class Breaks in package scala.util.control offers a break method, which can be used to exit an enclosing block that's marked with breakable.

```
import scala.util.control.Breaks._ 
import java.io._

val in = new BufferedReader(new InputStreamReader(System.in))

breakable { 
    while (true) { 
        println("? ") 
        if (in.readLine() == "") 
            break 
    } 
}
```
The Breaks class implements break by throwing an exception that is caught by an enclosing application of the breakable method.

### 7.7 VARIABLE SCOPE
Curly braces generally introduce a newscope.

```
val a = 1 
val a = 2 // Does not compile 
println(a)

val a = 1; 
{
    val a = 2 // Compiles just fine
    println(a) 
} 
println(a)
```

### 7.8 refactoring imperative-style code

```
def printMultiTable() = {
    var i = 1 // only i in scope here
    while (i <= 10) {
        var j = 1 // both i and j in scope here
        while (j <= 10) {
            val prod = (i * j).toString // i, j, and prod in scope here
            var k = prod.length // i, j, prod, and k in scope here
            while (k < 4) {
                print(" ")
                k += 1
            }
            print(prod)
            j += 1
        }
        // i and j still in scope; prod and k out of scope
        println()
        i += 1
    }
    // i still in scope; j, prod, and k out of scope
}
```
rewrite：

```
// Returns a row as a sequence 
def makeRowSeq(row: Int) =
    for (col <- 1 to 10) yield { 
        val prod = (row * col).toString 
        val padding = " " * (4 - prod.length) 
        padding + prod 
    }

// Returns a row as a string 
def makeRow(row: Int) = makeRowSeq(row).mkString

// Returns table as a string with one row per line 
def multiTable() = {
    val tableSeq = // a sequence of row strings 
       for (row <- 1 to 10) 
       yield makeRow(row)
    tableSeq.mkString("\n")
}
```
side-effect-free functions is easier to unit test.

imperative style: while and vars
function style: vals, for, helper functions and mkString.


### 7.9 小结
Skipped


