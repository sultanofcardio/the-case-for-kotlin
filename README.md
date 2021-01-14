# The case for Kotlin

Kotlin makes a compelling case as a drop-in replacement language for JVM developers current writing Java. If you're a 
Java developer, you can probably relate to the little annoyances that come with writing Java code.

This is my attempt to convince you that replacing Java with Kotlin will make for a much better developer experience.
Failing that, I hope to persuade you to augment your existing Java codebase with Kotlin.

## New shiny
This one is obvious: Kotlin is much younger than Java and was designed from the 
ground up with modern features in mind. While newer versions of Java may have 
some of these features, the language is carrying around decades of backwards 
compatibility that makes utilizing certain concepts awkward.

## Java interoperability
The Kotlin compiler (kotlinc) produces JVM bytecode compatible with Java 6, 7, and 8 
depending on the flag specified. Switching versions requires no change to 
source code, allowing developers to use modern language features while producing 
bytecode compatible with JRE 1.6.

What's more is that kotlinc supports mixed codebases of Java and Kotlin, allowing for 
gradual migration from Java to Kotlin. One could also maintain a long-term 
mixed codebase if desired.

Finally, Kotlin supports all the build systems, libraries, frameworks, and tooling
that Java does.

## More features than Java

### First class functions

These are supported in Java 8 through the single abstract method (SAM) interface syntax sugar

```java
interface SAMInterface {
    void handle(String data);
}

class TestClass {
    public static void receive(SAMInterface handler) {}
}

class Main {
    public static void main(String[] args) {
        SAMInterface sam = data -> {};
        TestClass.receive(data -> {});
        TestClass.receive(sam);
    }
}
```

In Kotlin, functions are first class by default. This makes the following possible

```kotlin

fun topLevelHandler(data: String) {}

fun receive(handler: (String) -> Unit)

fun main() {
    val sam: (String) -> Unit = { data -> }
    val sam2 = fun(data: String) {}
    receive(sam)
    receive(sam2)
    receive(::topLevelHandler)
    receive { data -> }
    
}
```

This eliminates the need for functional interfaces

### Generic type variance
### Built-in singleton support

How do we go about creating a singleton in Java? A widely accepted, thread-safe approach is the following

```java
public class Singleton {
  
  private Singleton()  { } 
  
  private static class BillPughSingleton { 
    private static final GFG INSTANCE = new GFG(); 
  } 
  
  public static GFG getInstance() {
    return BillPughSingleton.INSTANCE; 
  }
}
```

Some may prefer other approaches, but there is no standard provided by the language. In Kotlin, the singleton pattern 
is built into the language. The `object` keyword creates a singleton

```kotlin
object Singleton
```

### Properties not fields

In Java, class member variables are treated as private fields, and the developer is encouraged to create access methods
in accordance with encapsulation.

```java
public class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getName() {
        return this.name;
    }
}
```

The combination of getName and setName constitute what is known as the `name` property. Consider the following 
alternative

```java
public class Person2 {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getFirstName() {
        return this.name.split(" ")[0];
    }
    
    public String getLastName() {
        return this.name.split(" ")[0];
    }
}
```

This example has two read-only properties: `firstName` and `lastName`.

Consider the same thing in Kotlin

```kotlin
class Person(val name: String)

class Person2(name: String) {
    val firstName = name.split(" ").first()
    val lastName = name.split(" ")[1]
}
```

In Kotlin, class member variables are properties by default. This means they have implicit getters (and setters for 
mutable properties). You can override these functions if necessary, but you get encapsulation by default without being
forced to manually create them.

### Extension functions

We've all done this at some point. You find yourself doing something repeatedly and decide to be dry and create
a method for it. Still, it doesn't really fit anywhere, so you create a utility class

```java
public final class Utils {

    private Utils() {
    }

    public static void stringUtilMethod(String name) {
        System.out.println("Utility method");
    }
}


class Main {
    public static void main(String[] args) {
        Utils.stringUtilMethod("Some string");
    }
}
```

Extension functions make the Kotlin equivalent of this a much nicer experience

```kotlin
fun String.stringUtilMethod() {
    println("Utility method")
}

fun main() {
    "Some string".stringUtilMethod()
}
```

### Extension properties

We can also define extension properties on existing classes, since they're just a combination of getter and setter 
functions.

Say we have an existing Java class that we can't modify

```java
public class Temperature {
    private float celsius;

    public Temperature(float celsius) {
        this.celsius = celsius;
    }

    public float getCelsius() {
        return this.celsius;
    }

    public void setCelsius(float celsius) {
        this.celsius = celsius;
    }
}
```

We could add a fahrenheit property to this class in Kotlin

```kotlin
var Temperature.fahrenheit: Float
    get() = (celsius * 9 / 5) + 32
    set(value) {
        celsius = (value - 32) * 5 / 9
    }

fun main() {
    val temp = Temperature(100f)
    println(temp.fahrenheit) // 212.0
    temp.fahrenheit = 100f
    println(temp.celsius) // 37.77778
}
```

There's no direct equivalent for this in java

### Coroutines are better than threads
### Nullable types

Oh, the dreaded [NullPointerException](https://docs.oracle.com/javase/8/docs/api/java/lang/NullPointerException.html)!
We get this when we attempt to dereference a variable that hasn't been initialized, or set to null.

In Java, this leads to a lot of code like this

```java
public class Person {
    public Person(String name) {
        if (name == null) {
            throw new IllegalArgumentException("Name should not be null");
        }
        // ...
    }
}
```

In Kotlin, null is accounted for in the type system. We are able to declare at compile time whether a value can be
null

```kotlin
class Person(name: String)

fun main() {
    val person = Person(null) // error: Null can not be a value of a non-null type String
}
```

To indicate that the value can be null, we instead use the nullable type `String?`

```kotlin
class Person(name: String?)

fun main() {
    val person = Person(null)
    println(person.name.toUpperCase()) // Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
}
```

We are, however, forced to check if the value is null before using it. This way, we are less likely to run into runtime
null pointer exceptions. 

The correct usage would be `println(person.name?.toUpperCase())`

### Value types

Sometimes you just want to treat a POJO as a value, i.e. it is simply a holder of data, and you would like the ability
to uniquely distinguish it from other objects (e.g. in a collection). In Java, this requires you to override the 
[equals](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) 
and [hashCode](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) methods.

You can have your IDE generate these for you, or you can have the compiler do it.

```kotlin
data class Point(val x: Int, val y: Int)
```

In Kotlin, data classes have automatically generated equals and hashCode methods. This makes them suitable value types.

### Property delegates
### Named arguments
### Object destructuring


## Kotlin is more pleasant to write
### Conciseness
#### `public static void main`

## Classes of errors avoided in Kotlin

## Companies using Kotlin

## Google’s first-class support
