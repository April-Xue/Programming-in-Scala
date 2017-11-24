# 16-Working-with-Lists

## 16.1 LIST LITERALS
Two differences between list and array:   
First, lists are immutable. That is, elements of a list cannot be changed by assignment.

Second, lists have a recursive structure (i.e., a linked list), whereas arrays are flat.

## 16.2 THE LIST TYPE
Lists are homogeneous: the elements of a list all have the same type.


```
val fruit: List[String] = List("apples", "oranges", "pears") 
val nums: List[Int] = List(1, 2, 3, 4) 
val diag3: List[List[Int]] = 
    List( 
        List(1, 0, 0), 
        List(0, 1, 0), 
        List(0, 0, 1) 
    ) 
val empty: List[Nothing] = List()
```

The list type in Scala is covariant.

The empty list has type List[Nothing].

The empty list = Nil = List().

## 16.3 CONSTRUCTING LISTS
All lists are built from two fundamental building blocks, Nil and :: (pronounced "cons"). 

```
val fruit = "apples" :: ("oranges" :: ("pears" :: Nil)) 
val nums = 1 :: (2 :: (3 :: (4 :: Nil))) 
val diag3 = (1 :: (0 :: (0 :: Nil))) ::
            (0 :: (1 :: (0 :: Nil))) ::
            (0 :: (0 :: (1 :: Nil))) :: Nil 
val empty = Nil
```

List(1, 2, 3) equals to 1 :: (2 :: (3 :: Nil)) equals to 1 :: 2 :: 3 :: Nil

Note: you can's omit Nil like 1 :: 2 :: 3!

## 16.4 BASIC OPERATIONS ON LISTS


```
scala> val nums = 1 :: 2 :: 3 :: 4 :: Nil
nums: List[Int] = List(1, 2, 3, 4)

scala> nums.head
res2: Int = 1

scala> nums.tail
res3: List[Int] = List(2, 3, 4)

scala> nums.isEmpty
res4: Boolean = false
```

Empty lists can't use head and tail.

## 16.5 LIST PATTERNS

```
scala> val nums = 1 :: 2 :: 3 :: Nil
nums: List[Int] = List(1, 2, 3)

scala> val List(a,b,c) = nums
a: Int = 1
b: Int = 2
c: Int = 3

scala> val a :: b :: c :: Nil = nums
a: Int = 1
b: Int = 2
c: Int = 3
```

So :: exists twice in Scala:    
a name of a class in package scala    
a method in class List.

## 16.6 FIRST-ORDER METHODS ON CLASS LIST
**Concatenating two lists**   

```
scala> List(1, 2) ::: List(3, 4, 5) res0: List[Int] = List(1, 2, 3, 4, 5)

scala> List() ::: List(1, 2, 3) res1: List[Int] = List(1, 2, 3)

scala> List(1, 2, 3) ::: List(4) res2: List[Int] = List(1, 2, 3, 4)
```

**Taking the length of a list: length**   

```
scala> List(1, 2, 3).length 
res3: Int = 3
```
length is a expensive operation. When xs.Length == 0, use xs.isEmpty instead.

**Accessing the end of a list: init and last**  

```
scala> val abcde = List('a', 'b', 'c', 'd', 'e') 
abcde: List[Char] = List(a, b, c, d, e)

scala> abcde.last 
res4: Char = e

scala> abcde.init 
res5: List[Char] = List(a, b, c, d)
```
head and tail, which both run in constant time  
init and last need to traverse the whole list to compute their result.   
Empty lists can't use init and last.

**Reversing lists: reverse**    

```
val abcde = List('a', 'b', 'c', 'd', 'e') 
abcde: List[Char] = List(a, b, c, d, e)

scala> abcde.reverse 
res6: List[Char] = List(e, d, c, b, a)
```

**Prefixes and suffixes: drop, take, and splitAt**   

```
val abcde = List('a', 'b', 'c', 'd', 'e') 
abcde: List[Char] = List(a, b, c, d, e)

scala> abcde take 2 
res8: List[Char] = List(a, b)
if n > length, return entire list
if n < 0, return empty list

scala> abcde drop 2 
res9: List[Char] = List(c, d, e)
if n > length, return empty list
if n < 0, return entire list

scala> abcde splitAt 2 
res10: (List[Char], List[Char]) = (List(a, b),List(c, d, e))
```

**Element selection: apply and indices**   

```
scala> abcde apply 2 // rare in Scala 
res11: Char = c

scala> abcde(2) // rare in Scala
res12: Char = c

scala> abcde.indices 
res13: scala.collection.immutable.Range = Range(0, 1, 2, 3, 4)
```
**Flattening a list of lists: flatten**   

```
scala> List(List(1, 2), List(3), List(), List(4, 5)).flatten 
res14: List[Int] = List(1, 2, 3, 4, 5) 

scala> val fruit: List[String] = List("apples", "oranges", "pears")
fruit: List[String] = List(apples, oranges, pears)

scala> fruit.map(_.toCharArray).flatten 
res15: List[Char] = List(a, p, p, l, e, s, o, r, a, n, g, e, s, p, e, a, r, s)
```
It can only be applied to lists whose elements are all lists.

**Zipping lists: zip and unzip**    

```
The zip operation takes two lists and forms a list of pairs:
scala> abcde.indices zip abcde 
res17: scala.collection.immutable.IndexedSeq[(Int, Char)] = Vector((0,a), (1,b), (2,c), (3,d), (4,e))

If the two lists are of different length, any unmatched elements are dropped:
scala> val zipped = abcde zip List(1, 2, 3) 
zipped: List[(Char, Int)] = List((a,1), (b,2), (c,3))

more efficient way
scala> abcde.zipWithIndex 
res18: List[(Char, Int)] = List((a,0), (b,1), (c,2), (d,3), (e,4))

scala> zipped.unzip 
res19: (List[Char], List[Int]) = (List(a, b, c),List(1, 2, 3))
```
**Displaying lists: toString and mkString**    

```
scala> abcde.toString 
res20: String = List(a, b, c, d, e)

mkString
scala> abcde mkString ("[", ",", "]") 
res21: String = [a,b,c,d,e]

scala> abcde mkString "" 
res22: String = abcde

scala> abcde.mkString 
res23: String = abcde

scala> abcde mkString ("List(", ", ", ")") 
res24: String = List(a, b, c, d, e)
```
The mkString and addString methods are inherited from List's super trait Traversable, so they are applicable to all other collections as well.

**Converting lists: iterator, toArray, copyToArray**     

```
toArray
scala> val arr = abcde.toArray 
arr: Array[Char] = Array(a, b, c, d, e) 

scala> arr.toList 
res26: List[Char] = List(a, b, c, d, e)

copyToArray
xs copyToArray (arr, start)
scala> val arr2 = new Array[Int](10) 
arr2: Array[Int] = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0)
scala> List(1, 2, 3) copyToArray (arr2, 3)
scala> arr2 res28: Array[Int] = Array(0, 0, 0, 1, 2, 3, 0, 0, 0, 0)

iterator
scala> val it = abcde.iterator 
it: Iterator[Char] = non-empty iterator

scala> it.next res29: Char = a

scala> it.next res30: Char = b
```

## 16.7 HIGHER-ORDER METHODS ON CLASS LIST
**Mapping over lists: map, flatMap and foreach**   

```
map
scala> List(1, 2, 3) map (_ + 1) 
res32: List[Int] = List(2, 3, 4)

scala> val words = List("the", "quick", "brown", "fox") 
words: List[String] = List(the, quick, brown, fox)

scala> words map (_.length) 
res33: List[Int] = List(3, 5, 5, 3)

scala> words map (_.toList.reverse.mkString) 
res34: List[String] = List(eht, kciuq, nworb, xof)

flatmap
It applies the function to each list element and returns the concatenation of all function results.
scala> words map (_.toList) 
res35: List[List[Char]] = List(List(t, h, e), List(q, u, i, c, k), List(b, r, o, w, n), List(f, o, x)) 
scala> words flatMap (_.toList) 
res36: List[Char] = List(t, h, e, q, u, i, c, k, b, r, o, w, n, f, o, x)

foreach
scala> var sum = 0 
sum: Int = 0

scala> List(1, 2, 3, 4, 5) foreach (sum += _)

scala> sum 
res39: Int = 15
```

**Filtering lists: filter, partition, find, takeWhile, dropWhile, and span**    

```
filter
It yields the list of all elements x in xs for which p(x) is true
scala> List(1, 2, 3, 4, 5) filter (_ % 2 == 0) 
res40: List[Int] = List(2, 4)

partition
scala> List(1, 2, 3, 4, 5) partition (_ % 2 == 0) 
res42: (List[Int], List[Int]) = (List(2, 4),List(1, 3, 5))

find
scala> List(1, 2, 3, 4, 5) find (_ % 2 == 0) 
res43: Option[Int] = Some(2)

takeWhile and dropWhile and span
scala> List(1, 2, 3, -4, 5) takeWhile (_ > 0) 
res45: List[Int] = List(1, 2, 3)

scala> List(1, 2, 3, -4, 5) dropWhile (_ > 0)
res3: List[Int] = List(-4, 5)

scala> List(1, 2, 3, -4, 5) span (_ > 0) 
res47: (List[Int], List[Int]) = (List(1, 2, 3),List(-4, 5))
```

**Predicates over lists: forall and exists**   

```
scala> List(0, 0, 0).forall(_ == 0)
res14: Boolean = true

scala> List(0, 0, 0).exists( _ == 1)
res13: Boolean = false
```

**Folding lists: /: and :\**   

```
???
```

**Sorting lists: sortWith**   

```
scala> List(1, -3, 4, 2, 6) sortWith (_ < _) 
res51: List[Int] = List(-3, 1, 2, 4, 6)

scala> words sortWith (_.length > _.length) 
res52: List[String] = List(quick, brown, the, fox)
```
## 16.8 METHODS OF THE LIST OBJECT
All operations you have seen in early are implemented as methods of class List

**Creating lists from their elements: List.apply**    

```
scala> List.apply(1, 2, 3) 
res53: List[Int] = List(1, 2, 3)
```

**Creating a range of numbers: List.range**   

```
scala> List.range(1, 5) 
res54: List[Int] = List(1, 2, 3, 4)

scala> List.range(1, 9, 2) 
res55: List[Int] = List(1, 3, 5, 7)

scala> List.range(9, 1, -3) 
res56: List[Int] = List(9, 6, 3)
```

**Creating uniform lists: List.fill**    

```
scala> List.fill(5)('a') 
res57: List[Char] = List(a, a, a, a, a)

scala> List.fill(3)("hello") 
res58: List[String] = List(hello, hello, hello)

scala> List.fill(2, 3)('b') 
res59: List[List[Char]] = List(List(b, b, b), List(b, b, b))
```

**Tabulating a function: List.tabulate**    

```
scala> val squares = List.tabulate(5)(n => n * n) 
squares: List[Int] = List(0, 1, 4, 9, 16) 
scala> val multiplication = List.tabulate(5,5)(_ * _) 
multiplication: List[List[Int]] = List(List(0, 0, 0, 0, 0),
                                        List(0, 1, 2, 3, 4), 
                                        List(0, 2, 4, 6, 8),
                                        List(0, 3, 6, 9, 12), 
                                        List(0, 4, 8, 12, 16))
it's like cartesian product
0, 1, 2, 3, 4
0, 1, 2, 3, 4
```

**Concatenating multiple lists: List.concat**   

```
scala> List.concat(List('a', 'b'), List('c')) 
res60: List[Char] = List(a, b, c)

scala> List.concat(List(), List('b'), List('c')) 
res61: List[Char] = List(b, c)

scala> List.concat() 
res62: List[Nothing] = List()
```

## 16.9 PROCESSING MULTIPLE LISTS TOGETHER

```
scala> (List(10, 20), List(3, 4, 5)).zipped.map(_ * _) 
res63: List[Int] = List(30, 80)

scala> (List(10, 20, 50), List(3, 4)).zipped.map(_ * _)
res27: List[Int] = List(30, 80)

scala> (List("abc", "de"), List(3, 2)).zipped.exists(_.length != _) 
res65: Boolean = false
```
So, you can zip two lists and then, use map, flatMap...

## 16.10 UNDERSTANDING SCALA'S TYPE INFERENCE ALGORITHM
Too complicated, not interested in.


## 16.11 CONCLUSION
Lists are a real work horse in Scala.

