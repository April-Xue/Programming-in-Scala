# 13-Packages-and-Imports

## 13.1 PUTTING CODE IN PACKAGES

You can place the contents of an entire file into a package   
by putting a package clause at the top of the file.

```
package bobsrockets.navigation 
class Navigator

Listing 13.1 - Placing the contents of an entire file into a package.
```

You can place code into packages.  
This syntax is called a packaging.

```
package bobsrockets.navigation { 
    class Navigator 
}

Listing 13.2 - Long form of a simple package declaration.
```

You can have different parts of a file in different packages.

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

The code below also defines class Fleet in two nested packages bobrockets and fleets

```
package bobsrockets 
package fleets 
class Fleet {
    // No need to say bobsrockets.Ship
    def addShip() = { new Ship } }
```

Scala provides a package named "_root_" that is outside any package.  
Every top-level package is treated as a member of package "_root_".  
For example, both launch and bobsrockets of Listing 13.6 are members of package "_root_".



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

imports in Scala can appear anywhere, they can also refer to arbitrary values.   

```
def showFruit(fruit: Fruit) = { 
    import fruit._ 
    println(name + "s are " + color) 
}
Listing 13.8 - Importing the members of a regular (not singleton) object.
```

**SCALA'S FLEXIBLE IMPORTS**  
import packages themselves  

```
import java.util.regex

class AStarB { 
    // Accesses java.util.regex.Pattern 
    val pat = regex.Pattern.compile("a*b") 
}

Listing 13.9 - Importing a package name.
```
import Apple and Orange  

```
import Fruits.{Apple, Orange}
从object Fruits导入Apple和Orange。
```
import Apple, Orange, rename Apple to McIntosh  

```
import Fruits.{Apple => McIntosh, Orange}
```
import sql.Date as SDate, can simultaneously import Java Date  

```
import java.sql.{Date => SDate}
```
import S, access list S.Date  

```
import java.{sql => S}
```
import all members  

```
import Fruits.{_}
```
import all, but renames Apple to McIntosh  

```
import Fruits.{Apple => McIntosh, _}
```
import all, except Pear  

```
import Fruits.{Pear => _, _}
```
import Notebook.Apple, not the Fruits  

```
import Notebooks._ 
import Fruits.{Apple => _, _}
```


## 13.4 IMPLICIT IMPORTS
the following three import clauses had been added to the top of every source file with extension ".scala"  

```
import java.lang._ // everything in the java.lang package 
import scala._ // everything in the scala package 
import Predef._ // everything in the Predef object
```
because of implicit import:  
you can use Thread instead of java.lang.Thread  
you can use List instead of scala.List  

because later import overshadow earlier ones, StringBuilder will be scala.StringBuilder, not java.lang.StringBuilder  

## 13.5 ACCESS MODIFIERS
Scala's treatment of access modifiers roughly follows Java's but there are some important differences  

**Private members**  

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
A member labeled private is visible only inside the class or object that contains the member definition.  
It also applies to inner class, but java lets an outer class access private members of its inner classes  

**Protected members**  

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
In Scala, a protected member is only accessible from subclasses of the class  
In Java, a protected member can be accessed from other classes in the same package

**Public members**  
Any member not labeled private orprotected is public  

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
......



## 13.6 PACKAGE OBJECTS
you can put classes, traits, and standalone objects into packages  
you can also put method into package by using package object  
Package objects are frequently used to hold package-wide type aliases and implicit conversions.  

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



## 13.7 CONCLUSION



