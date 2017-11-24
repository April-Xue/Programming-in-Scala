# 17-Working-with-Other-Collections

## 17.1 SEQUENCES
Sequence types let you work with groups of data lined up in order.

The elements are ordered.

**Lists**
Lists support fast addition and removal of items to the beginning of the list, but they do not provide fast access to arbitrary indexes.

```
scala> val colors = List("red", "blue", "green") 
colors: List[String] = List(red, blue, green)

scala> colors.head 
res0: String = red

scala> colors.tail 
res1: List[String] = List(blue, green)
```

**Arrays**
Arrays allow you to hold a sequence of elements and efficiently access an element at an arbitrary position, either to get or update the element, with a zero-based index.

```
scala> val fiveInts = new Array[Int](5) 
fiveInts: Array[Int] = Array(0, 0, 0, 0, 0)

scala> val fiveToOne = Array(5, 4, 3, 2, 1) 
fiveToOne: Array[Int] = Array(5, 4, 3, 2, 1)

scala> fiveInts(0) = fiveToOne(4)

scala> fiveInts 
res3: Array[Int] = Array(1, 0, 0, 0, 0)
```

**List buffers**
ListBuffer provides constant time append and prepend operations.

```
scala> import scala.collection.mutable.ListBuffer 
import scala.collection.mutable.ListBuffer

scala> val buf = new ListBuffer[Int] 
buf: scala.collection.mutable.ListBuffer[Int] = ListBuffer()

scala> buf += 1 
res4: buf.type = ListBuffer(1)

scala> buf += 2 
res5: buf.type = ListBuffer(1, 2)

scala> buf 
res6: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2)

scala> 3 +=: buf 
res7: buf.type = ListBuffer(3, 1, 2)

scala> buf.toList 
res8: List[Int] = List(3, 1, 2)
```

**Array buffers**
An ArrayBuffer is like an array, you can additionally add and remove elements from the beginning and end of the sequence.

All Array operations are available, but a little bit slow.

```
scala> import scala.collection.mutable.ArrayBuffer 
import scala.collection.mutable.ArrayBuffer

must specify a type parameter, don't need a length
scala> val buf = new ArrayBuffer[Int]() 
buf: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()

scala> buf += 12 
res9: buf.type = ArrayBuffer(12)

scala> buf += 15 
res10: buf.type = ArrayBuffer(12, 15)

scala> buf 
res11: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(12, 15)
```

**Strings (via StringOps)**

```
Predef has an implicit conversion from String to StringOps
So you can use exist, String don't have it, StringOps have it.
scala> def hasUpperCase(s: String) = s.exists(_.isUpper) 
hasUpperCase: (s: String)Boolean

scala> hasUpperCase("Robert Frost") 
res14: Boolean = true

scala> hasUpperCase("e e cummings") 
res15: Boolean = false
```

## 17.2 SETS AND MAPS
By default when you write "Set" or "Map" you get an immutable object.

If you want to use both mutable and immutable sets or maps in the same source file: 

```
use immutable set as Set

use mutable Set
scala> import scala.collection.mutable 
import scala.collection.mutable

scala> val mutaSet = mutable.Set(1, 2, 3) mutaSet: scala.collection.mutable.Set[Int] = Set(1, 2, 3)
```
**Using sets**
Sets  ensure that at most one of each object.

```
scala> val wordsArray = Array("See", "run", "Run", "Run")
wordsArray: Array[String] = Array(See, run, Run, Run)

scala> import scala.collection.mutable
import scala.collection.mutable

scala> val words = mutable.Set.empty[String]
words: scala.collection.mutable.Set[String] = Set()

scala> for (word <- wordsArray) words += word.toLowerCase

scala> words
res5: scala.collection.mutable.Set[String] = Set(see, run)
```

```
Differences between mutable and immutable Set
immutable
val nums = Set(1, 2, 3)
nums + 5
nums - 3
nums ++ List(5, 6)
nums -- List(1, 2)
nums & Set(1, 3, 5, 7)
nums.size
nums.contains(3)

mutable
import scala.collection.mutable
val words = mutable.Set.empty[String]
words += "the"
words -= "the"
words ++= List("do", "re", "mi")
words --= List("do", "re")
words.clear
```

**Using maps**
Maps let you associate a value with each element of a set.

```
Differences between mutable and immutable Map
immutable
val nums = Map("i" -> 1, "ii" ->)
nums + ("vi" -> 6)
nums - "ii"
nums ++ List("iii" -> 3, "v" -> 5)
nums -- List("i", "ii")
nums.size
nums.contains("ii")
nums("ii")
nums.keys
nums.keySet
nums.values
nums.isEmpty

mutable
import scala.collection.mutable
val words = mutable.Map.empty[String, Int]
words += ("one" -> 1)
words -= "one"
words ++= List("one" -> 1, "two" -> 2, "three" -> 3)
words --= List("one", "two")
```
**Sorted sets and maps**
TreeSet and TreeMap implement traits SortedSet and SortedSet, internally use red-black tree.


```
TreeSet
scala> import scala.collection.immutable.TreeSet 
import scala.collection.immutable.TreeSet

scala> val ts = TreeSet(9, 3, 1, 8, 0, 2, 7, 4, 6, 5) 
ts: scala.collection.immutable.TreeSet[Int] = TreeSet(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val cs = TreeSet('f', 'u', 'n') 
cs: scala.collection.immutable.TreeSet[Char] = TreeSet(f, n, u)

TreeMap
scala> import scala.collection.immutable.TreeMap 
import scala.collection.immutable.TreeMap

scala> var tm = TreeMap(3 -> 'x', 1 -> 'x', 4 -> 'x') 
tm: scala.collection.immutable.TreeMap[Int,Char] = Map(1 -> x, 3 -> x, 4 -> x)

scala> tm += (2 -> 'x')

scala> tm 
res30: scala.collection.immutable.TreeMap[Int,Char] = Map(1 -> x, 2 -> x, 3 -> x, 4 -> x)
```

## 17.3 SELECTING MUTABLE VERSUS IMMUTABLE COLLECTIONS
It's better to start with an immutable collection and change it later.

Immutable can bring important space savings and performance advantages.

Scala provides some syntactic sugar to switch from immutable to mutable collections.

```
val
scala> val people = Set("Nancy", "Jane") people: scala.collection.immutable.Set[String] = Set(Nancy, Jane)

scala> people += "Bob" 
<console>:14: error: value += is not a member of    scala.collection.immutable.Set[String]
            people += "Bob" ^
     
var 
Whenever you write a += b, and a does not support a method named +=, Scala will try interpreting it as a = a + b.      
scala> var people = Set("Nancy", "Jane") 
people: scala.collection.immutable.Set[String] = Set(Nancy, Jane)

scala> people += "Bob"

scala> people 
res34: scala.collection.immutable.Set[String] = Set(Nancy, Jane, Bob)
First, a new collection will be created, and then people will be reassigned to refer to the new collection
```

## 17.4 INITIALIZING COLLECTIONS
You just place the elements in parentheses after the companion object name, and the Scala compiler make an invocation of an apply method on that companion object.

```
scala> List(1, 2, 3) 
res41: List[Int] = List(1, 2, 3)
```
Scala compiler infer the element type of a collection, if you want different types in a collection, use Any.


```
scala> val stuff = mutable.Set[Any](42) 
stuff: scala.collection.mutable.Set[Any] = Set(42)
```

**Converting to array or list**

```
scala> treeSet.toList 
res50: List[String] = List(blue, green, red, yellow)

scala> treeSet.toArray 
res51: Array[String] = Array(blue, green, red, yellow)
```

**Converting between mutable and immutable sets and maps**

```
Create a collection of the new type using the empty method and then add the new elements using either ++ or ++=。
scala> import scala.collection.mutable 
import scala.collection.mutable

scala> treeSet 
res52: scala.collection.immutable.TreeSet[String] = TreeSet(blue, green, red, yellow)

scala> val mutaSet = mutable.Set.empty ++= treeSet 
mutaSet: scala.collection.mutable.Set[String] = Set(red, blue, green, yellow)

scala> val immutaSet = Set.empty ++ mutaSet 
immutaSet: scala.collection.immutable.Set[String] = Set(red, blue, green, yellow)
```

## 17.5 TUPLES

```
def longestWord(words: Array[String]) = { 
    var word = words(0) 
    var idx = 0 
    for (i <- 1 until words.length) 
        if (words(i).length > word.length) { 
            word = words(i) 
            idx = i 
        } 
    (word, idx)
}

scala> val longest = longestWord("The quick brown fox".split(" ")) 
longest: (String, Int) = (quick,1)

scala> longest._1 
res53: String = quick

scala> longest._2 
res54: Int = 1

scala> val (word, idx) = longest 
word: String = quick idx: Int = 1

scala> word 
res55: String = quick

scala> val word, idx = longest 
word: (String, Int) = (quick,1) 
idx: (String, Int) = (quick,1)
```

## 17.6 CONCLUSION


