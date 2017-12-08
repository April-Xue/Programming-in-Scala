# 22-Implementing-Lists

## 22.1 THE LIST CLASS IN PRINCIPLE
Lists are defined by an abstract class List in the scala package, which comes with two subclasses for :: and Nil.

List is an abstract and covariant.
![20171206151252314783721.png](http://op2oo1z8b.bkt.clouddn.com/20171206151252314783721.png)
![20171206151252297610209.png](http://op2oo1z8b.bkt.clouddn.com/20171206151252297610209.png)

three basic methods:

```
def isEmpty: Boolean 
def head: T 
def tail: List[T]
```
These three methods are all abstract in class List.  
They are override in the sub object Nil and the subclass ::.

**The Nil object**  
The Nil object defines an empty list， inherits from type List[Nothing].

Because of covariance, this means that Nil is compatible with every instance of the List type.

```
case object Nil extends List[Nothing] {
  override def isEmpty = true
  override def head: Nothing =
    throw new NoSuchElementException("head of empty list")
  override def tail: List[Nothing] =
    throw new UnsupportedOperationException("tail of empty list")
  // Removal of equals method here might lead to an infinite recursion similar to IntMap.equals.
  override def equals(that: Any) = that match {
    case that1: scala.collection.GenSeq[_] => that1.isEmpty
    case _ => false
  }
}
```
**The :: class**  
It's named that way in order to support pattern matching with the infix ::.

```
final case class ::[B](override val head: B, private[scala] var tl: List[B]) extends List[B] {
  override def tail : List[B] = tl
  override def isEmpty: Boolean = false
}
every infix operation in a pattern 
is treated as a constructor application of the infix operator to its arguments. 
So the pattern x :: xs is treated as ::(x, xs) where :: is a case class
```
**List construction**  
The list construction methods :: and ::: are special。

They are end in a colon, the bound to their right operand.

```
def ::[B >: A] (x: B): List[B] =
    new scala.collection.immutable.::(x, this)
    
1 :: List(2, 3) = List(2, 3).::(1) = List(1, 2, 3)
    
def :::[B >: A](prefix: List[B]): List[B] =
    if (isEmpty) prefix
    else if (prefix.isEmpty) this
    else (new ListBuffer[B] ++= prefix).prependToList(this)
    
List(1, 2) ::: List(3, 4) = List(3, 4).:::(List(1, 2)) = List(1, 2, 3, 4)   
```

## 22.2 THE LISTBUFFER CLASS
You want List(1,2,3) -> List(2,3,4)

```
scala> val xs= List(1,2,3)
xs: List[Int] = List(1, 2, 3)

scala> for (x <- xs) result = result ::: List(x + 1)

scala> result
res2: List[Int] = List(2, 3, 4)

This is terribly inefficient.
Because ::: takes time proportional to the length of its first operand, 
the whole operation takes time proportional to the square of the length of the list.
```
The efficient way is to use ListBuffer.

```
scala> import scala.collection.mutable.ListBuffer
import scala.collection.mutable.ListBuffer

scala> val buf = new ListBuffer[Int]
buf: scala.collection.mutable.ListBuffer[Int] = ListBuffer()

scala> for (x <- xs) buf += x + 1

scala> buf.toList
res4: List[Int] = List(2, 3, 4)

both the append operation (+=) and the toList operation take (very short) constant time.
```

## 22.3 THE LIST CLASS IN PRACTICE
Skipped

## 22.4 FUNCTIONAL ON THE OUTSIDE
a typical strategy in Scala programming,  
trying to combine purity with efficiency by carefully delimiting the effects of impure operations.

## 22.5 CONCLUSION
Skipped


