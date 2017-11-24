# 18-Mutable-Objects

## 18.1 WHAT MAKES AN OBJECT MUTABLE?
If arbitrary operation on the list,   
return same result,   
it's immutable,   
otherwise, it's mutable.  

In Scala, every var that is a non-private member in a class implicitly defines a getter and a setter method with it. 

```
scala> class Time{ var hour = 1 }
defined class Time

scala> val time = new Time
time: Time = Time@a38d7a3

scala> time.hour
res7: Int = 1

scala> time.hour_=(2) 

scala> time.hour
res9: Int = 2
```

Some Class notices

```
class Thermometer {
    var celsius: Float = _
}
an initializer "= _" of a field assigns a zero value to that field. 
The zero value depends on the field's type. 
It is 0 for numeric types, 
false for booleans, 
and null for reference types.

Note that you cannot simply leave off the "= _" initializer in Scala. 
If you had written:
    var celsius: Float
this would declare an abstract variable, not an uninitialized one.
```

## 18.2 REASSIGNABLE VARIABLES AND PROPERTIES
The rest of this chapter shows by way of an extended example how mutable objects can be combined with first-class function values in interesting ways.

If you feel you want to get on with learning more Scala instead, it's safe to skip ahead to the next chapter.

## 18.3 CASE STUDY: DISCRETE EVENT SIMULATION
skipped
## 18.4 A LANGUAGE FOR DIGITAL CIRCUITS
skipped
## 18.5 THE SIMULATION API
skipped
## 18.6 CIRCUIT SIMULATION
skipped
## 18.7 CONCLUSION
This chapter brought together two techniques that seem disparate at first: mutable state and higher- order functions.

Mutable state was used to simulate physical entities whose state changes over time. 

Higher-order functions were used in the simulation framework to execute actions at specified points in simulated time.

