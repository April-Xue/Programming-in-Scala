# 16-Working-with-Lists

## 16.1 LIST LITERALS
Two differences between list and array:   
First, lists are immutable. That is, elements of a list cannot be changed by assignment.

Second, lists have a recursive structure (i.e., a linked list), whereas arrays are flat.

## 16.2 THE LIST TYPE
lists are homogeneous: the elements of a list all have the same type.


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
## 16.8 METHODS OF THE LIST OBJECT
## 16.9 PROCESSING MULTIPLE LISTS TOGETHER
## 16.10 UNDERSTANDING SCALA'S TYPE INFERENCE ALGORITHM
## 16.11 CONCLUSION


