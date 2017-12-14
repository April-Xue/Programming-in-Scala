# 26-Extractors
You might wish to be able to create your own kinds of patterns.

## 26.1 AN EXAMPLE: EXTRACTING EMAIL ADDRESSES
Analyze strings that represent email addresses,    
you want to print the user and domain parts of the address.

```
def isEMail(s: String): Boolean 
def domain(s: String): String 
def user(s: String): String

The traditional way

if (isEMail(s)) println(user(s) + " AT " + domain(s)) 
else println("not an email address")

using pattern matching

s match { 
    case EMail(user, domain) => println(user + " AT " + domain) 
    case _ => println("not an email address") 
}

strings are not case classes
they do not have a representation that conforms to EMail(user, domain).
```

## 26.2 EXTRACTORS







## 26.3 PATTERNS WITH ZERO OR ONE VARIABLES







## 26.4 VARIABLE ARGUMENT EXTRACTORS







## 26.5 EXTRACTORS AND SEQUENCE PATTERNS







## 26.6 EXTRACTORS VERSUS CASE CLASSES







## 26.7 REGULAR EXPRESSIONS







## 26.8 CONCLUSION









