# 24-Collections-in-Depth

## 24.1 MUTABLE AND IMMUTABLE COLLECTIONS
A mutable collection can be changed,  
immutable collection can't,  
but still have operations that simulation add, remove or update,   
and in each case return a new collection.

```
All collection classes are found in the package scala.collection   
or one of its sub packages: mutable, immutable, and generic.

A collection in package scala.collection.immutable   
is guaranteed to be immutable.

A collection in package scala.collection.mutable   
have some operations that change the collection in place.

A collection in package scala.collection   
can be either mutable or immutable. 

for example:
scala.collection.IndexedSeq[T] is a supertrait of
scala.collection.immutable.IndexedSeq[T] 
and 
scala.collection.mutable.IndexedSeq[T]

Generally, the root collections in package scala.collection   
define the same interface as the immutable collections.   
Typically, the mutable collections in package scala.collection.mutable   
add some side-effecting modification operations to this immutable interface.

scala.collection.generic
contains building blocks for implementing collections.
```
Collection hierarchy

```
Traversable
    Iterable 
        Seq
            IndexedSeq 
                Vector 
                ResizableArray 
                GenericArray 
            LinearSeq 
                MutableList 
                List 
                Stream 
            Buffer 
                ListBuffer 
                ArrayBuffer

        Set
            SortedSet 
                TreeSet 
            HashSet (mutable) 
            LinkedHashSet 
            HashSet (immutable) 
            BitSet 
            EmptySet, Set1, Set2, Set3, Set4

        Map
        SortedMap 
            TreeMap 
        HashMap (mutable)
        LinkedHashMap (mutable) 
        HashMap (immutable) 
        EmptyMap, Map1, Map2, Map3, Map4
```

## 24.2 COLLECTIONS CONSISTENCY
every kind of collection can be created by the same uniform syntax

```
Traversable(1, 2, 3) 
Iterable("x", "y", "z") 
Map("x" -> 24, "y" -> 25, "z" -> 26) 
Set(Color.Red, Color.Green, Color.Blue) 
SortedSet("hello", "world") 
Buffer(x, y, z) 
IndexedSeq(1.0, 2.0) 
LinearSeq(a, b, c)
```

## 24.3 TRAIT TRAVERSABLE


## 24.4 TRAIT ITERABLE


## 24.5 THE SEQUENCE TRAITS SEQ, INDEXEDSEQ, AND LINEARSEQ


## 24.6 SETS







## 24.7 MAPS







## 24.8 CONCRETE IMMUTABLE COLLECTION CLASSES







## 24.9 CONCRETE MUTABLE COLLECTION CLASSES







## 24.10 ARRAYS







## 24.11 STRINGS







## 24.12 PERFORMANCE CHARACTERISTICS







## 24.13 EQUALITY







## 24.14 VIEWS







## 24.15 ITERATORS







## 24.16 CREATING COLLECTIONS FROM SCRATCH







## 24.17 CONVERSIONS BETWEEN JAVA AND SCALA COLLECTIONS







## 24.18 CONCLUSION









