# Immutablity

Most appeals to immutablity are made on theoretical grounds, using words like *purity*. Let's consider the practical advantages:

1. *Your program is easier to reason about.* You do not need to concern yourself with how the state of an object changes over time because there is only one state an object has – the state it was created with. When you log such an object, or pause in the debugger to inspect it, you are looking at the state of the object for its entire lifetime in the program.
1. *Your program can be more efficient.* This is because derived data is also immutable. The hash code of an immutable object, the length of immutable string, the area or perimeter of an immutable shape, and so on are also all immutable. Instead of computing these values repeatedly when they are needed repeatedly, you can compute it once and [memoize it](https://en.wikipedia.org/wiki/Memoization).
1. *Your program is safer*. We can pass immutable objects to other methods without worrying that the callee will modify it. Similarly our code can accept and retain a reference to an immutable object without worrying that the caller will later modify it. Immutable objects are also inherently thread safe, since thread safety is concerned with multiple threads safely mutating an object, but immutable objects cannot be mutated.

I believe that immutablity leads to simpler, more efficient, and safer programs. If you view writing code as a craft, that should be reason enough for embracing immutablity.

## Practical examples from the standard library

You might worry that embracing immutablity will be stifling – that it will conflict with the need to write code and ship products. But consider two fundamental classes in Java that are immutable and which are easy to use.

### `java.time.Instant`

The [Joda-Time library](https://www.joda.org/joda-time/) is a third-party library that filled the large gaps in functionality left by the inadequate date and time abstractions that preceded Java 8. It defined the `Instant` abstraction, which represented an instantaneous point in time. One concrete implementation of the abstraction was `MutableDateTime`, which allowed clients to mutate the represented time. For example:

```java
MutableDateTime dateTime = MutableDateTime.now();
Instant instant = dateTime;
System.out.println(instant);

dateTime.addHours(1);
System.out.println(instant);  // Prints one hour later
```

Because an `Instant` might be mutable, its interface defined a `toInstant` method that allowed creating an immutable copy. This would allow clients to defensively program against other code modifying the instance:

```java
MutableDateTime dateTime = MutableDateTime.now();
Instant instant = dateTime.toInstant();
System.out.println(instant);

dateTime.addHours(1);
System.out.println(instant);  // Prints the same time as before
```

The `java.time` package introduced in Java 8 sought to make the Joda-Time library obsolete, and introduced a simpler set of date and time abstractions. One change was that its `Instant` class was a single immutable type, similar to `String`. To add one hour to an `Instant`, you would have to create a new `Instant`:

```java
Instant now = Instant.now();
System.out.println(now);

Instant later = now.plus(Duration.ofHours(1));
System.out.println(later);  // Prints one hour later
```

Clients that were passed an `Instant` that is part of the `java.time` package are ensured it is immutable and do not need to defensively copy it.

### `java.lang.String`

A more fundamental class is the ubiquitous `String` class from Java. Underlying this class is an `char[]` array – which is intrinsically mutable. But the `String` class represents an immutable abstraction on top of that array.

Likely you have never been annoyed on account of the `String` class being immutable. Maybe you view `String` as such a primitive type that it never occurred to you that `String` could have been immutable. This is the case in some languages. Take C++ for example:

```c++
string s = "We invited the strippers, JFK, and Stalin";
cout << s << endl;
s.erase(24, 1);      // Removes the first comma at index 24
cout << s << endl;   // Prints "We invited the strippers, JFK and Stalin"
```

Perhaps the reason that you've never considered it stifling that `String` is immutable is because we can convert a `String` into a `StringBuilder` representation. This mutable representation allows us to add, replace, or remove characters efficiently until we call its `toString()` method to create a new immutable `String` object with the performed edits. For example:

```java
String s = "We invited the strippers, JFK, and Stalin";
System.out.println(s);
String updatedString = new StringBuilder(s)
    .delete(24)                     // Removes the first comma at index 24
    .toString();
System.out.println(updatedString);  // Prints "We invited the strippers, JFK and Stalin"
```

(Another reason that shouldn't be overlooked is that we can easily form new strings by concatenating them using `+` – the singular case of operator overloading in Java.)

But the use of an accompanying *builder class* is a theme we will explore below: Immutable classes, but with accompanying builder classes that allow us to convert any immutable instance into a mutable form. And after performing all necessary mutations, a method to convert the builder representation to a new immutable instance.

## Creating immutable classes

What we want are classes like `String` and `Instant` – each of which is an *immutable value type*.

Let's first define what a *value type* is first. A value type is a type with [value semantics](https://en.wikipedia.org/wiki/Value_semantics). This means that `equals` and `hashCode` are well defined for the type, and if `a.equals(b)` returns `true` then instances `a` and `b` are considered interchangeable.

The `String` and `Instant` classes we saw earlier are examples of value types. Collection implementations like `ArrayList`, `TreeSet`, and `HashMap` are also value types. Other examples could include:

1. A `Fraction` class that composes a numerator and a denominator.
1. A `Money` class that composes a `Currency` enum and some `long` value representing the amount.
1. An `HttpRequest` class or `HttpResponse` class that consists of HTTP headers and an optional body.

And *immutable* means that given an instance of the type, there are no methods on the object to mutate its composed data. Or given two distinct instances of the class named `a` and `b`, if `a.equals(b)` returns `true` then there is no way to mutate either `a` or `b` such that `a.equals(b)` returns `false`. Immutability is a binary property – just like you can't be _kind of_ pregnant, you can't be _kind of_ immutable. Even if just one bit can be flipped in an object that spans gigabytes of memory, then the object is mutable.

Creating immutable value types in Java is usually laborious because:

1. defining even an immutable type requires so much boilerplate, namely defining getters and a reasonable `equals` and `hashCode` implementation
1. many of the standard collection implementations you rely on, such as `ArrayList`, `TreeSet`, and `HashMap`, are mutable

But with the help of third party libraries, creating such types is relatively easy.

First we consider the libraries AutoValue and Lombok, which address the first point above. Second we consider the immutable collection implementations that are part of Google's Guava library, which address the second point above.

### AutoValue

TODO

### Lombok

TODO

### Google Guava

TODO

