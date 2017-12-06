# 14-Assertions-and-Tests

## 14.1 ASSERTIONS
assert and ensuring are defined in Predef.
**assert**  

```
def above(that: Element): Element = { 
    val this1 = this widen that.width 
    val that1 = that widen this.width 
    assert(this1.width == that1.width) 
    elem(this1.contents ++ that1.contents) 
}

assert(condition)
assert(condition, explanation)
```
**ensuring**

```
private def widen(w: Int): Element =
    if (w <= width) 
        this 
    else { 
        val left = elem(' ', (w - width) / 2, height) 
        var right = elem(' ', w - width - left.width, height) 
        left beside this beside right 
    } ensuring (w <= _.width)

The ensuring method takes one argument, 
a predicate function that takes a result type and returns Boolean, 
and passes the result to the predicate.
```

## 14.2 TESTING IN SCALA
You have many options for testing in Scala,   
from established Java tools, such as JUnit and TestNG,   
to tools written in Scala, such as ScalaTest, specs2, and ScalaCheck.

ScalaTest is the most flexible Scala test framework: it can be easily customized to solve different problems.

```
Int maven pom.xml:
<dependency>
  <groupId>org.scalatest</groupId>
  <artifactId>scalatest_2.11</artifactId>
  <version>3.0.4</version>
  <scope>test</scope>
</dependency>

import org.scalatest.FunSuite 
import Element.elem

class ElementSuite extends FunSuite {
    test("elem result should have passed width") { 
        val ele = elem('x', 2, 3) 
        assert(ele.width == 2) 
    }
}

Trait Suite is the central unit of composition in ScalaTest.

Suite declares "lifecycle" methods defining a default way to run tests, 
which can be overridden to customize how tests are written and run.
```
ScalaTest is integrated into common build tools (such as sbt and Maven) and IDEs (such as IntelliJ IDEA and Eclipse).

```
scala> (new ElementSuite).execute() 
ElementSuite:
- elem result should have passed width
```

## 14.3 INFORMATIVE FAILURE REPORTS
An informative error message

```
scala -cp scalatest_2.11-2.2.6.jar 
    
scala> import org.scalatest.Assertions._
import org.scalatest.Assertions._

scala> assert(width == 2)
org.scalatest.exceptions.TestFailedException: 3 did not equal 2
  at org.scalatest.Assertions$class.newAssertionFailedException(Assertions.scala:500)
  at org.scalatest.Assertions$.newAssertionFailedException(Assertions.scala:1538)
  at org.scalatest.Assertions$AssertionsHelper.macroAssert(Assertions.scala:466)
  ... 33 elided
    
scala> import org.scalatest.DiagrammedAssertions._
import org.scalatest.DiagrammedAssertions._

scala> assert(1 == 2)
org.scalatest.exceptions.TestFailedException:

assert(1 == 2)
         |
         false

  at org.scalatest.Assertions$class.newAssertionFailedException(Assertions.scala:500)
  at org.scalatest.DiagrammedAssertions$.newAssertionFailedException(DiagrammedAssertions.scala:381)
  at org.scalatest.DiagrammedAssertions$DiagrammedAssertionsHelper.macroAssert(DiagrammedAssertions.scala:238)
  ... 33 elided
```

## 14.4 TESTS AS SPECIFICATIONS
In the behavior-driven development (BDD) testing style,   
the emphasis is on writing human-readable specifications of the expected behavior of code   
and accompanying tests that verify the code has the specified behavior.

```
import org.scalatest.FlatSpec 
import org.scalatest.Matchers 
import Element.elem

class ElementSpec extends FlatSpec with Matchers {
    "A UniformElement" should 
        "have a width equal to the passed value" in { 
        val ele = elem('x', 2, 3) 
        ele.width should be (2) 
    }
    it should "have a height equal to the passed value" in { 
        val ele = elem('x', 2, 3) 
        ele.height should be (3) 
    }
}

By mixing in trait Matchers, you can write assertions that read more like natural language.
like "should be"
```
The specs2 testing framework,   
an open source tool written in Scala by Eric Torreborre,   
also supports the BDD style of testing but with a different syntax.

```
import org.specs2._ 
import Element.elem

object ElementSpecification extends Specification { 
    "A UniformElement" should { 
        "have a width equal to the passed value" in { 
            val ele = elem('x', 2, 3) 
            ele.width must be_==(2) 
        } 
        "have a height equal to the passed value" in { 
            val ele = elem('x', 2, 3) 
            ele.height must be_==(3) 
        } 
    }
}
```
One of the big ideas of BDD is that tests can be used to facilitate communication between the people, like users, designers.  
Another example:

```
import org.scalatest._

class TVSetSpec extends FeatureSpec with GivenWhenThen {
    feature("TV power button") { 
        scenario("User presses power button when TV is off") { 
            Given("a TV set that is switched off") 
            When("the power button is pressed") 
            Then("the TV should switch on") 
            pending 
        } 
    }
}
```

## 14.5 PROPERTY-BASED TESTING
Another useful testing tool for Scala is ScalaCheck, an open source framework written by Rickard Nilsson,   
enables you to specify properties that the code under test must obey,  
for each property, will generate data and execute assertions that check whether the property.

```
import org.scalatest.WordSpec 
import org.scalatest.prop.PropertyChecks 
import org.scalatest.MustMatchers._ 
import Element.elem

class ElementSpec extends WordSpec with PropertyChecks { 
    "elem result" must { 
        "have passed width" in { 
            forAll { (w: Int) => 
                whenever (w > 0) { 
                    elem('x', w, 3).width must equal (w) 
                } 
            } 
        }
    }
}
```

## 14.6 ORGANIZING AND RUNNING TESTS
Skipped

## 14.7 CONCLUSION
Skipped

