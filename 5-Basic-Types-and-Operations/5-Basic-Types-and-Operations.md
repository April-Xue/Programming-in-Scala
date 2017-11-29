# 5-Basic-Types-and-Operations.md
Now that you've seen classes and objects in action,  
it's a good time to look at Scala's basic types and operations in more depth.

Scala's basic types, including Strings and the value types Int, Long, Short, Byte, Float, Double, Char, and Boolean.

You'll learn how implicit conversions can "enrich" variants of these basic types.

### 5.1 SOME BASIC TYPES
types Byte, Short, Int, Long, and Char are called integral types.  
The integral types plus Float and Double are called numeric types.

```
Byte 	 8-bit signed two's complement integer (-27 to 27 - 1, inclusive) 
Short 	 16-bit signed two's complement integer (-215 to 215 - 1, inclusive) 
Int 	 32-bit signed two's complement integer (-231 to 231 - 1, inclusive) 
Long 	 64-bit signed two's complement integer (-263 to 263 - 1, inclusive) 
Char 	 16-bit unsigned Unicode character (0 to 216 - 1, inclusive) 
String 	 a sequence of Chars 
Float 	 32-bit IEEE 754 single-precision float 
Double 	 64-bit IEEE 754 double-precision float 
Boolean  true or false
```
String resides in package java.lang, others in package scala.

All the members of package scala and java.lang are automatically imported into every Scala source file, just use them with simple name.

Scala's basic types have the exact same ranges as the corresponding types in Java.

### 5.2 LITERALS
All of the basic types can be written with literals.
A literal is a way to write a constant value directly in code.

**FAST TRACK FOR JAVA PROGRAMMERS**  
Scala does not support octal literals, integer literals that start with a 0, such as 031, do not compile.

**Integer literals**  
Integer literals for the types Int, Long, Short, and Byte come in two forms: decimal and hexadecimal.

```
Number begins with a 0x or 0X, is hexadecimal (base 16).
scala> val hex = 0x5 
hex: Int = 5

scala> val hex2 = 0x00FF 
hex2: Int = 255

scala> val magic = 0xcafebabe 
magic: Int = -889275714

Number begins with a non-zero digit, is decimal.
scala> val dec1 = 31 
dec1: Int = 31
```
Scala shell always prints integer values in base 10.


```
If an integer literal ends in an L or l, it is a Long; otherwise it is an Int.
scala> val prog = 0XCAFEBABEL 
prog: Long = 3405691582

scala> val tower = 35L 
tower: Long = 35

If an Int literal is assigned to a variable of type Short or Byte, it's just Short or Byte.
scala> val little: Short = 367 
little: Short = 367

scala> val littler: Byte = 38 
littler: Byte = 38
```

**Floating point literals**   

```
If a floating-point literal ends in an F or f, it is a Float; otherwise it is a Double
scala> val big = 1.2345 
big: Double = 1.2345

scala> val bigger = 1.2345e1 
bigger: Double = 12.345

scala> val biggerStill = 123E45 
biggerStill: Double = 1.23E47
```

**Character literals**  

```
Character literals are composed of any Unicode character between single quotes.
scala> val a = 'A' 
a: Char = A

you can identify a character using its Unicode code point
write \u followed by four hex digits
scala> val d = '\u0041' 
d: Char = A

such Unicode characters can appear anywhere
scala> val B\u0041\u0044 = 1 
BAD: Int = 1

escape sequences:
scala> val backslash = '\\' 
backslash: Char = \

other escape sequences:
\n 	line feed (\u000A) 
\b 	backspace (\u0008) 
\t 	ab (\u0009) 
\f 	form feed (\u000C) 
\r 	carriage return (\u000D) 
\" 	double quote (\u0022) 
\' 	single quote (\u0027) 
\\ 	backslash (\u005C) 
```
**String literals**  
A string literal is composed of characters surrounded by double quotes

```
scala> val hello = "hello" 
hello: String = hello

Scala includes a special syntax for raw strings.
println("""|Welcome to Ultamix 3000.
           |Type "HELP" for help.""".stripMargin)
```
**Symbol literals**  

```
A symbol literal is written 'ident, where ident can be any alphanumeric identifier.
scala> val s = 'aSymbol 
s: Symbol = 'aSymbol

scala> val nm = s.name 
nm: String = aSymbol
```
**Boolean literals**  

```
scala> val bool = true 
bool: Boolean = true

scala> val fool = false 
fool: Boolean = false
```

### 5.3 STRING INTERPOLATION
Allows you to embed expressions within string literals.

```
val name = "reader" 
println(s"Hello, $name!")

scala> s"The answer is ${6 * 7}." 
res0: String = The answer is 42.
```

Scala provides two other string interpolators by default: raw and f.

```
The raw string interpolator behaves like s, 
except it does not recognize character literal escape sequences.
println(raw"No\\\\escape!") // prints: No\\\\escape!

The f string interpolator allows you to attach printf-style formatting instructions
scala> f"${math.Pi}%.5f" 
res1: String = 3.14159
default is %s
```

### 5.4 operators are methods
Scala provides a rich set of operators for its basic types.

```
scala> val sum = 1 + 2 // Scala invokes 1.+(2)
sum: Int = 3

scala> val sumMore = 1.+(2) 
sumMore: Int = 3
```
You can use any method in operator notation.

```
scala> val s = "Hello, world!" 
s: String = Hello, world!

scala> s indexOf 'o' // Scala invokes s.indexOf('o')
res0: Int = 4

scala> s indexOf ('o', 5) // Scala invokes 
s.indexOf('o', 5) res1: Int = 8
```
Above is infix operator notation, takes two operands.
scala also has prefix and postfix, they take just one operand.   
prefix notation-the `-' in -7,  
postfix-the "toLong" in "7 toLong.  


### 5.5 arithmetic operations
You can invoke arithmetic methods via infix operator notation for addition (+), subtraction (-), multiplication (*), division (/), and remainder (%) on any numeric type.

```
scala> 1.2 + 2.3 
res6: Double = 3.5
```

### 5.6 relational and logical operations
You can compare numeric types with relational methods greater than (>), less than (<), greater than or equal to (>=), and less than or equal to (<=), which yield a Boolean result.

```
scala> 1 > 2 
res16: Boolean = false
```

Logical methods, logical-and (&& and &) and logical-or (|| and |), take Boolean operands in infix notation and yield a Boolean result.

```
scala> val toBe = true 
toBe: Boolean = true

scala> val question = toBe || !toBe 
question: Boolean = true

scala> val paradox = toBe && !toBe 
paradox: Boolean = false
```

### 5.7 bitwise operations
Scala enables you to perform operations on individual bits of integer types with several bitwise methods.

```
bitwise-ands each bit in 1 (0001) and 2 (0010), which yields 0 (0000)
scala> 1 & 2 
res24: Int = 0

bitwise-ors each bit in the same operands, yielding 3 (0011)
scala> 1 | 2 
res25: Int = 3

bitwise-xors each bit in 1 (0001) and 3 (0011), yielding 2 (0010)
scala> 1 ^ 3 
res26: Int = 2

inverts each bit in 1 (0001), yielding -2
scala> ~1 
res27: Int = -2
```
Scala integer types also offer three shift methods.

```
-1 in binary is 11111111111111111111111111111111

-1 is shifted to the right 31 bit positions, fills with ones as it shifts right
scala> -1 >> 31 
res28: Int = -1

-1 is shifted to the right 31 bit positions, filling with zeroes along the way.
scala> -1 >>> 31 
res29: Int = 1

1, is shifted left two positions (filling in with zeroes)
scala> 1 << 2 
res30: Int = 4
```

### 5.8 object equality
You can use either == or its inverse !=, to compare any two objects.

```
scala> 1 == 2 
res31: Boolean = false

scala> 1 != 2 
res32: Boolean = true

scala> 2 == 2 
res33: Boolean = true
```

### 5.9 operator precedence and associativity
Shit

### 5.10 rich wrappers
For each basic type described in this chapter, there is also a "rich wrapper" that provides several additional methods.

```
Basic 	type Rich wrapper 
Byte 	scala.runtime.RichByte
Short 	scala.runtime.RichShort 
Int 	scala.runtime.RichInt 
Long 	scala.runtime.RichLong 
Char 	scala.runtime.RichChar 
Float 	scala.runtime.RichFloat 
Double 	scala.runtime.RichDouble 
Boolean scala.runtime.RichBoolean 
String	scala.collection.immutable.StringOps
```
### 5.11 小结
 

