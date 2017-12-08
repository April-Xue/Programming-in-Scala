# 23-For-Expressions-Revisited
You want to find out the names of all pairs of mothers and their children in that list.

```
case class Person(name: String, isMale: Boolean, children: Person*)
val lara = Person("Lara", false) 
val bob = Person("Bob", true) 
val julie = Person("Julie", false, lara, bob) 
val persons = List(lara, bob, julie)

//withFilter avoid the creation of an intermediate data structure
persons.withFilter(p => !p.isMale)
       .flatMap( p => p.children.map(c => (p.name, c.name)))

for (p <- persons; if !p.isMale; c <- p.children) 
    yield (p.name, c.name)
```

all for expressions that yield a result   
are translated by the compiler into combinations   
of invocations of the higher-order methods map, flatMap, and withFilter.

All for loops without yield   
are translated into a smaller set of higher-order functions:   
just withFilter and foreach.

## 23.1 FOR EXPRESSIONS

```
for ( seq ) yield expr

for (p <- persons; n = p.name; if (n startsWith "To")) 
    yield n

for { p <- persons           // a generator 
      n = p.name             // a definition 
      if (n startsWith "To") // a filter
} yield n

scala> for (x <- List(1, 2); y <- List("one", "two", "three")) 
        yield (x, y)
res8: List[(Int, String)] = 
    List((1,one), (1,two), (1,three), (2,one), (2,two), (2,three))
```

## 23.2 THE N-QUEENS PROBLEM
Skipped

## 23.3 QUERYING WITH FOR EXPRESSIONS
The for notation is essentially equivalent to common operations of database query languages.

```
case class Book(title: String, authors: String*)

val books: List[Book] =
    List( 
        Book( 
            "Structure and Interpretation of Computer Programs", 
            "Abelson, Harold", "Sussman, Gerald J."), 
        Book( 
            "Principles of Compiler Design", "Aho, Alfred", 
            "Ullman, Jeffrey" 
        ), 
        Book( 
            "Programming in Modula-2", 
            "Wirth, Niklaus" 
        ),
        Book( 
            "Elements of ML Programming", 
            "Ullman, Jeffrey" 
        ), 
        Book(
            "The Java Language Specification", 
            "Gosling, James",
            "Joy, Bill", 
            "Steele, Guy", 
            "Bracha, Gilad" 
        )
    )
    
for (b <- books; a <- b.authors if a startsWith "Gosling") 
    yield b.title  
    
for {
    b1 <- books 
    b2 <- books if b1 != b2
    a1 <- b1.authors
    a2 <- b2.authors if a1 == a2
} yield a1
```

## 23.4 TRANSLATION OF FOR EXPRESSIONS
Every for expression can be expressed in terms of the three higher-order functions map, flatMap, and withFilter.

## 23.5 GOING THE OTHER WAY
Every application of a map, flatMap, or filter can be represented as a for expression.

## 23.6 GENERALIZING FOR

```
• If your type defines just map, 
it allows for expressions consisting of a single generator.

• If it defines flatMap as well as map, 
it allows for expressions consisting of several generators.

• If it defines foreach, 
it allows for loops (both with single and multiple generators).

• If it defines withFilter, 
it allows for filter expressions starting with an if in the for expression.
```

## 23.7 CONCLUSION
Skipped










