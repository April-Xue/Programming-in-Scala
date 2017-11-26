# 19-Type-Parameterization
The first part develops a data structure for purely functional queues. 

The second part develops techniques to hide internal representation details of this structure. 

The final part explains variance of type parameters and how it interacts with information hiding.
## 19.1 FUNCTIONAL QUEUES
A functional queue works like that:

```
scala> val q = Queue(1, 2, 3) 
q: Queue[Int] = Queue(1, 2, 3)

scala> val q1 = q enqueue 4 
q1: Queue[Int] = Queue(1, 2, 3, 4)

scala> q 
res0: Queue[Int] = Queue(1, 2, 3)
```
where a list is usually extended at the front, using a :: operation, a queue is extended at the end, using enqueue.


```
class SlowAppendQueue[T](elems: List[T]) { // Not efficient 
    def head = elems.head 
    def tail = new SlowAppendQueue(elems.tail) 
    def enqueue(x: T) = new SlowAppendQueue(elems ::: List(x)) 
}
Head and tail are constant, enqueue is not.

class SlowHeadQueue[T](smele: List[T]) { // Not efficient 
    // smele is elems reversed 
    def head = smele.last 
    def tail = new SlowHeadQueue(smele.init) 
    def enqueue(x: T) = new SlowHeadQueue(x :: smele) 
}
Now enqueue is constant time, but head and tail are not.

class Queue[T]( 
    private val leading: List[T], 
    private val trailing: List[T] 
) { 
    private def mirror = 
    if (leading.isEmpty) 
        new Queue(trailing.reverse, Nil) 
    else 
        this
    
    def head = mirror.leading.head
    def tail = { 
        val q = mirror 
        new Queue(q.leading.tail, q.trailing) 
    }
    def enqueue(x: T) = 
        new Queue(leading, x :: trailing)
}
That is cool
```

## 19.2 INFORMATION HIDING
For now, the Queue constructor, which is globally accessible, takes two lists as parameters, we need to hide this constructor from client code.

**Private constructors and factory methods**  
In Scala, the primary constructor does not have an explicit definition; it is defined implicitly by the class parameters and body. 

We can add a private modifier in front of the class parameter

```
class Queue[T] private ( 
    private val leading: List[T], 
    private val trailing: List[T] 
)
```
Now it can be accessed only from within the class itself and its companion object.

```
scala> new Queue(List(1, 2), List(3)) 
<console>:9: error: constructor Queue in class Queue cannot 
be accessed in object $iw
            new Queue(List(1, 2), List(3)) ^
```            
There needs to be some other way to create new queues from a sequence of initial elements.

```
1. One possibility is to add an auxiliary constructor.
def this(elems: T*) = this(elems.toList, Nil)  

Now you can:
val test = new Queue(1,2,3)  


2. Another possibility is to add a factory method.
A neat way to do this:
object Queue { 
    // constructs a queue with initial elements `xs' 
    def apply[T](xs: T*) = new Queue[T](xs.toList, Nil) 
}
Because the factory method is called apply, 
clients can just create queues like Queue(1, 2, 3), equals to Queue.apply(1, 2, 3), 
seems like a globally defined factory method.

Now you can:
val test = Queue(1,2,3)  

But you should know that Scala has no globally visible methods, 
every method must be contained in an object or a class. 
However, using methods named apply inside global objects, 
that looks like invocations of global methods.
```

**An alternative: private classes**

```
trait Queue[T] { 
    def head: T 
    def tail: Queue[T] 
    def enqueue(x: T): Queue[T] 
}

object Queue {

    def apply[T](xs: T*): Queue[T] = 
        new QueueImpl[T](xs.toList, Nil)
    
    private class QueueImpl[T]( 
        private val leading: List[T], 
        private val trailing: List[T] 
    ) extends Queue[T] {
    
    def mirror =
        if (leading.isEmpty) 
            new QueueImpl(trailing.reverse, Nil) 
        else
            this
    
    def head: T = mirror.leading.head
    def tail: QueueImpl[T] = { 
        val q = mirror 
        new QueueImpl(q.leading.tail, q.trailing) 
    }
    def enqueue(x: T) = 
        new QueueImpl(leading, x :: trailing)
    }
}
```
Instead of hiding individual constructors and methods, this version hides the whole implementation class.

## 19.3 VARIANCE ANNOTATIONS
For now, Queue is a trait and Queue[String] is a type.

You can also say that Queue is a generic trait, means that you are defining many specific types with one generically written class or trait.

The combination of type parameters and subtyping poses some interesting questions.

Is Queue covariant?

In Scala, however, generic types have by default nonvariant. A Queue[String] would not be usable as a Queue[AnyRef].

So, what is covariant, contravariant?

```
covariant:
trait Queue[+T] { ... }

you are telling Scala that you wantQueue[String], 
for example, to be considered a subtype of Queue[AnyRef].
```

```
contravariant:
trait Queue[-T] { ... }

if T is a subtype of type S, 
then Queue[S] is a subtype of Queue[T]
```

**Variance and arrays**

```
// this is Java 
String[] a1 = { "abc" }; 
Object[] a2 = a1; 
a2[0] = new Integer(17); 
String s = a1[0];

compiles ok, executing failed.
Exception in thread "main" java.lang.ArrayStoreException: 
java.lang.Integer 
        at JavaArrays.main(JavaArrays.java:8)
        
Java stores the element type of the array at runtime.
It's unsafe and expensive.
but the principal inventor of the Java said they want treat arrays generically.
they wanted to be able to write a method to sort all elements of an array.
void sort(Object[] a, Comparator cmp) { ... }
```

Scala tries to be purer than Java in not treating arrays as covariant, which treats arrays as nonvariant

```
scala> val a1 = Array("abc") 
a1: Array[String] = Array(abc)

scala> val a2: Array[Any] = a1 
<console>:8: error: type mismatch; 
found : Array[String] 
required: Array[Any] 
    val a2: Array[Any] = a1 
                         ^
But sometimes it is necessary to interact with legacy methods.
scala> val a2: Array[Object] = a1.asInstanceOf[Array[Object]] 
a2: Array[Object] = Array(abc)
The cast is always legal at compile-time
but then you will get ArrayStore exceptions just like java.
```

## 19.4 CHECKING VARIANCE ANNOTATIONS
Useless
## 19.5 LOWER BOUNDS

```
class Queue[+T] (private val leading: List[T], 
        private val trailing: List[T] ) { 
    def enqueue[U >: T](x: U) = 
        new Queue[U](leading, x :: trailing) // ... 
}
"U >: T", defines T as the lower bound for U.
U is required to be a supertype of T.
```

## 19.6 CONTRAVARIANCE
Useless.
## 19.7 OBJECT PRIVATE DATA
Useless.
## 19.8 UPPER BOUNDS
Shit.
## 19.9 CONCLUSION
Shit.


