# 13-Packages-and-Imports

## 13.1 PUTTING CODE IN PACKAGES

**You can place the contents of an entire file into a package   
by putting a package clause at the top of the file.**

```
package bobsrockets.navigation 
class Navigator

Listing 13.1 - Placing the contents of an entire file into a package.
```

**You can place code into packages.  
This syntax is called a packaging.**

```
package bobsrockets.navigation { 
    class Navigator 
}

Listing 13.2 - Long form of a simple package declaration.
```

**You can have different parts of a file in different packages.**

```
package bobsrockets { 
    package navigation {
        // In package bobsrockets.navigation 
        class Navigator
        package tests {
            // In package bobsrockets.navigation.tests 
            class NavigatorSuite
        }
    }
}

Listing 13.3 - Multiple packages in the same file.
```

```
package bobsrockets { 
    package navigation { 
        class Navigator { 
                // No need to say bobsrockets.navigation.StarMap 
                val map = new StarMap 
            } 
            class StarMap 
        } 
        class Ship { 
            // No need to say bobsrockets.navigation.Navigator 
            val nav = new navigation.Navigator 
        } 
        package fleets { 
            class Fleet { 
                // No need to say bobsrockets.Ship 
                def addShip() = { new Ship } 
        } 
    }
}

Listing 13.4 - Concise access to classes and packages.
```

```
package bobsrockets { 
   class Ship 
}
    
package bobsrockets.fleets { 
   class Fleet { 
       // Doesn't compile! Ship is not in scope. 
       def addShip() = { new Ship } 
    }
}

Listing 13.5 - Symbols in enclosing packages not automatically available.
```

```
// In file launch.scala 
package launch { 
    class Booster3 
} 
// In file bobsrockets.scala 
package bobsrockets { 
    package navigation { 
        package launch { 
            class Booster1 
        } 
        class MissionControl { 
            val booster1 = new launch.Booster1 
            val booster2 = new bobsrockets.launch.Booster2 
            val booster3 = new _root_.launch.Booster3 
        }
    } 
    package launch { 
        class Booster2 
    }
}

Listing 13.6 - Accessing hidden package names.
```
## 13.2 CONCISE ACCESS TO RELATED CODE

**The code below also defines class Fleet in two nested packages bobrockets and fleets**

```
package bobsrockets 
package fleets 
class Fleet {
    // No need to say bobsrockets.Ship
    def addShip() = { new Ship } }
```

**Scala provides a package named "_root_" that is outside any package.  
Every top-level package is treated as a member of package "_root_".  
For example, both launch and bobsrockets of Listing 13.6 are members of package "_root_".**



## 13.3 IMPORTS

```
package bobsdelights

abstract class Fruit( 
    val name: String, val color: String 
)

object Fruits { 
    object Apple extends Fruit("apple", "red") 
    object Orange extends Fruit("orange", "orange") 
    object Pear extends Fruit("pear", "yellowish") 
    val menu = List(Apple, Orange, Pear) 
}
Listing 13.7 - Bob's delightful fruits, ready for import.
```

```
// easy access to Fruit 
import bobsdelights.Fruit

// easy access to all members of bobsdelights 
import bobsdelights._

// easy access to all members of Fruits 
import bobsdelights.Fruits._

```

**imports in Scala can appear anywhere, they can also refer to arbitrary values.**   

```
def showFruit(fruit: Fruit) = { 
    import fruit._ 
    println(name + "s are " + color) 
}
Listing 13.8 - Importing the members of a regular (not singleton) object.
```

**SCALA'S FLEXIBLE IMPORTS**  
**import packages themselves**  

```
import java.util.regex

class AStarB { 
    // Accesses java.util.regex.Pattern 
    val pat = regex.Pattern.compile("a*b") 
}

Listing 13.9 - Importing a package name.
```
**import Apple and Orange**  

```
import Fruits.{Apple, Orange}
从object Fruits导入Apple和Orange。
```
**import Apple, Orange, rename Apple to McIntosh**  

```
import Fruits.{Apple => McIntosh, Orange}
```
**import sql.Date as SDate, can simultaneously import Java Date**  

```
import java.sql.{Date => SDate}
```
**import S, access list S.Date**  

```
import java.{sql => S}
```
**import all members**  

```
import Fruits.{_}
```
**import all, but renames Apple to McIntosh**  

```
import Fruits.{Apple => McIntosh, _}
```
**import all, except Pear**  

```
import Fruits.{Pear => _, _}
```
**import Notebook.Apple, not the Fruits**  

```
import Notebooks._ 
import Fruits.{Apple => _, _}
```


## 13.4 IMPLICIT IMPORTS
**the following three import clauses had been added to the top of every source file with extension ".scala"**  

```
import java.lang._ // everything in the java.lang package 
import scala._ // everything in the scala package 
import Predef._ // everything in the Predef object
```
**because of implicit import:**  
**you can use Thread instead of java.lang.Thread**  
**you can use List instead of scala.List**  

**because later import overshadow earlier ones, StringBuilder will be scala.StringBuilder, not java.lang.StringBuilder**  

## 13.5 ACCESS MODIFIERS
package，class或object的member能使用private和protected进行修饰，这些access modifier限制了其在特定代码区域的访问。scala的access modifier与java相似，但略有不同。

**Private members**
private members在scala中与java相似，只能在含有member定义的class或object中被访问。这个规则同样使用于inner class。这样子更一致，但是与java不同。如：

```
class Outer { 
    class Inner { 
        private def f() = { println("f") } 
        class InnerMost { 
            f() // OK 
        } 
    } 
    (new Inner).f() // error: f is not accessible 
}
Listing 13.10 - How private access differs in Scala and Java.
```
在scala里，(new Inner).f()是不被允许的，因为在Inner class定义之外，但是InnerMost class中可以，因为其定义在Inner class内部。在java中，两种情况都可以。

**Protected members**
scala中访问protected members比java更严格。

```
package p {
    class Super { 
        protected def f() = { println("f") } 
    } 
    class Sub extends Super {
        f() 
    } 
    class Other {
        (new Super).f() // error: f is not accessible 
    }
}

Listing 13.11 - How protected access differs in Scala and Java.
```
在scala中，只有是一个class的子类，才能访问该class的protected member。但是在java中是被允许，因为Other class 和Super class被定义在同一个package中。

**Public members**
在scala里，不带private或protected修饰符就是public，能被在任何地方访问。

```
package bobsrockets

package navigation { 
    private[bobsrockets] class Navigator { 
        protected[navigation] def useStarChart() = {} 
        class LegOfJourney { 
            private[Navigator] val distance = 100 
        } 
        private[this] var speed = 200 
    } 
} 
package launch { 
    import navigation._ 
    object Vehicle { 
        private[launch] val guide = new Navigator 
    } 
}

Listing 13.12 - Flexible scope of protection with access qualifiers.
```
**Scope of protection**
在scala中，使用qualifiers，能使access modifiers被放大。private[X]指的是protected[X]，访问时"up to" X，X通常指的是一些封闭的package，class或object。

Listing 13.12展现了很多被使用的access qualifiers。class Navigator被修饰为private[bobsrockets]，所以能被package bobsrockets里所有的class和object访问。但是在object Vehicle中无法被访问，因为Vehicle包含在package launch，其被包含在bobsrockets。另一方面，包含在package bobsrockets中的所有代码不能访问class Navigator。

这种方法在跨package的大型项目中相当有用。能够使得在你项目的子package中互相被访问，但是在你项目的client中不能被访问。java中没有这种方法。

当然，private也能be the directly enclosing package。如Listing 13.12中object Vehicle里所展现的。与Java的package-private意思相同。

```
Table 13.1 - Effects of private qualifiers on LegOfJourney.distance
no access modifier public access 
private[bobsrockets] access within outer 
package private[navigation] same as package visibility in Java 
private[Navigator] same as private in Java 
private[LegOfJourney] same as private in Scala 
private[this] access only from same object
```
所有qualifiers能被作用于protected，同样适合于private。class C里的protected[X]，能被class C的所有子类和enclosing package, class, or object X里的所有代码访问。如useStarChart方法能被Navigator的所有subclass和enclosing package navigation里的所有代码访问。与java中protected意思相同。

private能被指向一个enclosing class或object。如class LegOfJourney中的distance被修饰为private[Navigator]，所以能在class Navigator中任意地方被访问。与java中inner class的private member意思相同。

最后，scala有比private更严格的access modifier，称为object-private。如class Navigator中的speed，访问不仅要在class Navigator里，而且必须是同一个的instance。

以下这种情况不被允许，即使出现在class Navigator内：

```
val other = new Navigator 
other.speed // this line would not compile
```




一个member为private[this]时，在其他相同class的object将不能被访问。这对documentation很有用，有时候在写variance annotations (see Section 19.7 for details)时会更普通。

总结，Table 13.1列举了private qualifiers的作用，每一条在Listing 13.12可以找到具体的作用。

**Visibility and companion objects**
在java里，static member和instance member属于相同的class，所以access modifiers的作用相同。在scala里面，没有static member，但是你可以有一个companion object，包含了只存在一次的member。如：

```
class Rocket { 
    import Rocket.fuel 
    private def canGoHomeAgain = fuel > 20 
}

object Rocket {
    private def fuel = 10 
    def chooseStrategy(rocket: Rocket) = {
        if (rocket.canGoHomeAgain)
            goHome()
        else
            pickAStar() 
    } 
    def goHome() = {} 
    def pickAStar() = {}
}
Listing 13.13 - Accessing private members of companion classes and objects.
```
class和其companion object对其member有共享的访问权限。即一个object能访问companion class的所有private的member，反之亦然。

需要注意的一点是，java中所有的subclass能访问class的protected static member，但是scala中，companion object的protected member没有意义，因为singleton object没有subclass。

## 13.6 PACKAGE OBJECTS
到目前为止，你看到的所有代码加入到package里的都是class，trait，和object。但是scala不会限制于此，Any kind of definition that you can put inside a class can also be at the top level of a package。

这种情况下，定义在package object里，每一个package只能有一个package object。定义在package object里的都是package的member。如Listing 13.14，package.scala文件里定义了一个包含showFruit方法的package object，，它不是一个package，之后可以在PrintMenu.scala中直接import showFruit方法。

```
// In file bobsdelights/package.scala 
package object bobsdelights { 
    def showFruit(fruit: Fruit) = { 
        import fruit._ 
        println(name + "s are " + color) 
    } 
}

// In file PrintMenu.scala 
package printmenu 
import bobsdelights.Fruits 
import bobsdelights.showFruit

object PrintMenu { 
    def main(args: Array[String]) = { 
        for (fruit <- Fruits.menu) { 
            showFruit(fruit) 
        } 
    } 
}

Listing 13.14 - A package object.
```
package将会被频繁使用，为了hold package-wide type aliases (Chapter 20) and implicit conversions (Chapter 21)。The top-level的scala package包含一个package object，里面的定义能被所有scala代码import。

## 13.7 CONCLUSION



