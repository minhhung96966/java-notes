点击关注[公众号](#公众号)及时获取笔主最新更新文章，并可免费领取本文档配套的《Java面试突击》以及Java工程师必备学习资源。

随着 Java 8 的普及度越来越高，很多人都提到面试中关于Java 8 也是非常常问的知识点。应各位要求和需要，我打算对这部分知识做一个总结。本来准备自己总结的，后面看到Github 上有一个相关的仓库，地址：
[https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial)。这个仓库是英文的，我对其进行了翻译并添加和修改了部分内容，下面是正文了。

<!-- MarkdownTOC -->

- [Java 8 Tutorial](#java-8-tutorial)
    - [接口的默认方法\(Default Methods for Interfaces\)](#接口的默认方法default-methods-for-interfaces)
    - [Lambda表达式\(Lambda expressions\)](#lambda表达式lambda-expressions)
    - [函数式接口\(Functional Interfaces\)](#函数式接口functional-interfaces)
    - [方法和构造函数引用\(Method and Constructor References\)](#方法和构造函数引用method-and-constructor-references)
    - [Lamda 表达式作用域\(Lambda Scopes\)](#lamda-表达式作用域lambda-scopes)
      - [访问局部变量](#访问局部变量)
      - [访问字段和静态变量](#访问字段和静态变量)
      - [访问默认接口方法](#访问默认接口方法)
    - [内置函数式接口\(Built-in Functional Interfaces\)](#内置函数式接口built-in-functional-interfaces)
      - [Predicate](#predicate)
      - [Function](#function)
      - [Supplier](#supplier)
      - [Consumer](#consumer)
      - [Comparator](#comparator)
  - [Optional](#optional)
  - [Streams\(流\)](#streams流)
    - [Filter\(过滤\)](#filter过滤)
    - [Sorted\(排序\)](#sorted排序)
    - [Map\(映射\)](#map映射)
    - [Match\(匹配\)](#match匹配)
    - [Count\(计数\)](#count计数)
    - [Reduce\(规约\)](#reduce规约)
  - [Parallel Streams\(并行流\)](#parallel-streams并行流)
    - [Sequential Sort\(串行排序\)](#sequential-sort串行排序)
    - [Parallel Sort\(并行排序\)](#parallel-sort并行排序)
  - [Maps](#maps)
  - [Date API\(日期相关API\)](#date-api日期相关api)
    - [Clock](#clock)
    - [Timezones\(时区\)](#timezones时区)
    - [LocalTime\(本地时间\)](#localtime本地时间)
    - [LocalDate\(本地日期\)](#localdate本地日期)
    - [LocalDateTime\(本地日期时间\)](#localdatetime本地日期时间)
  - [Annotations\(注解\)](#annotations注解)
  - [Where to go from here?](#where-to-go-from-here)

<!-- /MarkdownTOC -->


# Java 8 Tutorial 

Welcome to my introduction to Java 8. This tutorial will guide you step by step through all the new language features. Based on short code examples, you will learn how to use default interface methods, lambda expressions, method references and repeatable comments. At the end of this article, you will be familiar with the latest API changes, such as streams, Functional Interfaces, extensions to the Map class, and the new Date API. There are no large sections of boring text, only a bunch of commented code snippets.


### Default methods for interfaces

Java 8 enables us to add non-abstract method implementations to interfaces by using the `default` keyword. This function is also called [Virtual Extension Method](http://stackoverflow.com/a/24102730)。

The first example:

```java
interface Formula{

    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
```

In addition to the abstract method calculation interface formula, the Formula interface also defines the default method `sqrt`. The class that implements this interface only needs to implement the abstract method `calculate`. The default method `sqrt` can be used directly. Of course, you can also create an object directly through the interface, and then implement the default method in the interface. Let's demonstrate this way through code.

```java
public class Main {

  public static void main(String[] args) {
    // Access interface through anonymous inner class
    Formula formula = new Formula() {
        @Override
        public double calculate(int a) {
            return sqrt(a * 100);
        }
    };

    System.out.println(formula.calculate(100));     // 100.0
    System.out.println(formula.sqrt(16));           // 4.0

  }

}
```

The formula is implemented as an anonymous object. The code is very easy to understand, 6 lines of code implement the calculation `sqrt(a * 100)`. In the next section, we will see that there is a better and more convenient way to implement a single method object in Java 8.

**Translator's Note:** Both abstract classes and interfaces can be accessed through anonymous inner classes. You cannot create objects directly through abstract classes or interfaces. For the above access to the interface through the anonymous inner class, we can understand this: an inner class implements the abstract method in the interface and returns an inner class object, and then we let the interface reference point to this object.

### Lambda expressions

First look at how to arrange strings in older versions of Java:

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

Just pass in a List object and a comparator to the static method `Collections.sort` to sort them in the specified order. The usual practice is to create an anonymous comparator object and pass it to the `sort` method.

In Java 8, you don’t need to use this traditional way of anonymous objects. Java 8 provides a more concise syntax, lambda expressions:

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

As you can see, the code has become more segmented and more readable, but it can actually be written shorter:

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

For the function body with only one line of code, you can remove the curly braces {} and the return keyword, but you can also write shorter:

```java
names.sort((a, b) -> b.compareTo(a));
```

The List class itself has a `sort` method. And the Java compiler can automatically deduce the parameter type, so you don't need to write the type again. Next, let's see what other uses of lambda expressions are available.

### Functional Interfaces

**Translator's Note:** The original text does not explain this part clearly, so it has been revised!

Java language designers have invested a lot of energy to think about how to make existing functions friendly to support Lambda. The final approach is to increase the concept of functional interfaces. **“"Functional interface" refers to an interface that contains only one abstract method, but can have multiple non-abstract methods (that is, the default method mentioned above).** Interfaces like this can be implicitly converted to lambda expressions. `java.lang.Runnable` and `java.util.concurrent.Callable` are the two most typical examples of functional interfaces. Java 8 adds a special annotation `@FunctionalInterface`, but this annotation is usually not necessary (recommended in some cases). As long as the interface contains only one abstract method, the virtual machine will automatically determine that the interface is a functional interface. It is generally recommended to use the `@FunctionalInterface` annotation on the interface to declare. In this case, the compiler will report an error if it finds that the interface you have annotated with this annotation has more than one abstract method, as shown in the figure below

![@FunctionalInterface annotation](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-2/@FunctionalInterface.png)

Example:

```java
@FunctionalInterface
public interface Converter<F, T> {
  T convert(F from);
}
```

```java
    // TODO Convert numeric string to integer type
    Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass()); //class java.lang.Integer
```

**Translator's Note:** Most of the functional interfaces do not need to be written by ourselves. Java 8 has implemented them for us. These interfaces are all in the java.util.function package.

### Method and Constructor References

The code in the previous section can also be represented by static method references:

```java
    Converter<String, Integer> converter = Integer::valueOf;
    Integer converted = converter.convert("123");
    System.out.println(converted.getClass());   //class java.lang.Integer
```
Java 8 allows you to pass a reference to a method or constructor via the `::` keyword. The above example shows how to reference static methods. But we can also refer to object methods:

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```

Next, let’s see how the constructor is referenced using the `::` keyword. First, we define a simple class that contains multiple constructors:

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```
Next we specify an object factory interface used to create Person objects:

```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

Here we use constructor references to associate them, instead of manually implementing a complete factory:

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```
We only need to use `Person::new` to get a reference to the Person class constructor, and the Java compiler will automatically select the appropriate constructor based on the parameter type of the `PersonFactory.create` method.

### Lambda Scopes

#### Access local variables

We can directly access external local variables in lambda expressions:

```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

But unlike anonymous objects, the variable num here does not need to be declared as final, and the code is also correct:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

However, the num here must not be modified by the following code (that is, implicitly with final semantics). For example, the following cannot be compiled:

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;//Attempting to modify num in a lambda expression is also not allowed.
```

#### Access fields and static variables

Compared with local variables, we have read and write access to instance fields and static variables in lambda expressions. This behavior is consistent with anonymous objects.

```java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```

#### Access the default interface method

Remember the formula example in the first section? The `Formula` interface defines a default method `sqrt`, which can be accessed from every formula instance that contains an anonymous object. This does not apply to lambda expressions.

The default method cannot be accessed from the lambda expression, so the following code cannot be compiled:

```java
Formula formula = (a) -> sqrt(a * 100);
```

### Built-in Functional Interfaces

The JDK 1.8 API contains many built-in functional interfaces. Some of these interfaces are more common in older versions of Java, such as: `Comparator` or `Runnable`, these interfaces have added `@FunctionalInterface` annotations so that they can be used in lambda expressions.

But the Java 8 API also provides a lot of brand new functional interfaces to make your programming work more convenient. Some interfaces are from [Google Guava](https://code.google.com/p/guava-libraries/) Curry, even if you are familiar with these, it is still necessary to see how these are extended to use on lambda.
#### Predicate

The Predicate interface is an **predicate** interface that returns a boolean value with only one parameter. This interface contains a variety of default methods to combine Predicate into other complex logic (for example: AND, OR, NOT):

**Translator's Note:** The source code of the Predicate interface is as follows

```java
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {
    
    // This method accepts an incoming type and returns a boolean value. This method should be used for judgment.
    boolean test(T t);

    // The and method is similar to the relational operator "&&", both sides are true before returning true
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    // Similar to the relational operator "!", it negates the judgment
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    // The or method is similar to the relational operator "||", as long as one of the two sides is true, it returns true
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
   // This method receives an Object object and returns a Predicate type. This method is used to determine that the method of the first test is equal to the method of the second test.
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

Example:

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

#### Function

The Function interface accepts a parameter and generates the result. The default method can be used to link multiple functions together (compose, andThen):

**Translator's Note:** The function interface source code is as follows

```java

package java.util.function;
 
import java.util.Objects;
 
@FunctionalInterface
public interface Function<T, R> {
    
    //The Function object is applied to the input parameters, and then the calculation result is returned.
    R apply(T t);
    //Integrate two Functions and return a Function object that can perform the functions of the two Function objects.
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    // 
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
 
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```



```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

#### Supplier

The Supplier interface produces a result of a given generic type. Unlike the Function interface, the Supplier interface does not accept parameters.

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

#### Consumer

The Consumer interface represents the operation to be performed on a single input parameter.

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

#### Comparator

Comparator is a classic interface in old Java. Java 8 adds a variety of default methods on top of it:

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```

## Optional

Optional is not a functional interface, but a beautiful tool for preventing NullPointerException. This is an important concept in the next section, let us quickly understand how Optional works.

Optional is a simple container whose value may be null or not. Before Java 8, a function should return a non-null object but sometimes returns nothing. In Java 8, you should return Optional instead of null.

Translator's Note: The role of each method in the example has been added.

```java
// of(): Create an Optional for non-null values
Optional<String> optional = Optional.of("bam");
// isPresent(): Returns true if the value exists, otherwise returns false
optional.isPresent();           // true
// get(): If Optional has a value, return it, otherwise throw NoSuchElementException
optional.get();                 // "bam"
// orElse(): If there is a value, it will be returned, otherwise it will return the specified other value
optional.orElse("fallback");    // "bam"
// ifPresent(): If the Optional instance has a value, the consumer is called, otherwise no processing is done
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

Recommended reading: [[Java8]How to use Optional properly](https://blog.kaaass.net/archives/764)

## Streams

`java.util.Stream` represents a sequence of operations that can be applied to a set of elements at one time. Stream operations are divided into intermediate operations or final operations. The final operation returns a specific type of calculation result, and the intermediate operation returns the Stream itself, so you can chain multiple operations in sequence. The creation of Stream needs to specify a data source, such as a subclass of `java.util.Collection`, List or Set, Map does not support. Stream operations can be executed serially or in parallel.

First look at how Stream is used, first create a list of data used by the example code:

```java
List<String> stringList = new ArrayList<>();
stringList.add("ddd2");
stringList.add("aaa2");
stringList.add("bbb1");
stringList.add("aaa1");
stringList.add("bbb3");
stringList.add("ccc");
stringList.add("bbb2");
stringList.add("ddd1");
```

Java 8 extends the collection class, you can create a Stream through Collection.stream() or Collection.parallelStream(). The following sections will explain in detail the commonly used Stream operations:

### Filter

Filtering uses a predicate interface to filter and retain only eligible elements. This operation is an **intermediate operation**, so we can apply other Stream operations (such as forEach) to the filtered result. forEach needs a function to execute the filtered elements in sequence. forEach is a final operation, so we cannot perform other Stream operations after forEach.

```java
        // Test Filter
        stringList
                .stream()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);//aaa2 aaa1
```

forEach is designed for Lambda, maintaining the most compact style. And the Lambda expression itself can be reused, which is very convenient.

### Sorted

Sorting is an **intermediate operation**, which returns the sorted Stream. **If you don't specify a custom Comparator, the default sort will be used.**

```java
        // Test Sort
        stringList
                .stream()
                .sorted()
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);// aaa1 aaa2
```

It should be noted that sorting only creates a stream after sorting, without affecting the original data source. The original data stringCollection will not be modified after sorting:

```java
    System.out.println(stringList);// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```

### Map

The intermediate operation map will turn the elements into other objects in turn according to the specified Function interface.

The following example shows the conversion of a string to uppercase. You can also use map to convert objects into other types. The Stream type returned by map is determined by the return value of the function passed into your map.

```java
        // Test the Map operation
        stringList
                .stream()
                .map(String::toUpperCase)
                .sorted((a, b) -> b.compareTo(a))
                .forEach(System.out::println);// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```



### Match

Stream provides a variety of matching operations, allowing to detect whether the specified Predicate matches the entire Stream. All matching operations are **final operations** and return a boolean value.

```java
        // Test the match operation
        boolean anyStartsWithA =
                stringList
                        .stream()
                        .anyMatch((s) -> s.startsWith("a"));
        System.out.println(anyStartsWithA);      // true

        boolean allStartsWithA =
                stringList
                        .stream()
                        .allMatch((s) -> s.startsWith("a"));

        System.out.println(allStartsWithA);      // false

        boolean noneStartsWithZ =
                stringList
                        .stream()
                        .noneMatch((s) -> s.startsWith("z"));

        System.out.println(noneStartsWithZ);      // true
```



### Count

Counting is a **final operation**, which returns the number of elements in the stream. **The return value type is long**.

```java
      //Test the Count operation
        long startsWithB =
                stringList
                        .stream()
                        .filter((s) -> s.startsWith("b"))
                        .count();
        System.out.println(startsWithB);    // 3
```

### Reduce

This is a **final operation** that allows multiple elements in the stream to be reduced to one element through a specified function, and the result of the reduction is expressed through the Optional interface:

```java
        //测试 Reduce (规约)操作
        Optional<String> reduced =
                stringList
                        .stream()
                        .sorted()
                        .reduce((s1, s2) -> s1 + "#" + s2);

        reduced.ifPresent(System.out::println);//aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2
```



**Translator's Note:** The main function of this method is to combine Stream elements. It provides an initial value (seed), and then combines it with the first, second, and nth elements of the previous Stream according to the operation rules (BinaryOperator). In this sense, string concatenation, numerical sum, min, max, and average are all special reduce. For example, the sum of Stream is equivalent to `Integer sum = integers.reduce(0, (a, b) -> a+b);` There is also a case where there is no starting value. At this time, the first two elements of Stream will be combined , The return is Optional.

```java
// String concatenation, concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
// Find the minimum value, minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min); 
// Sum, sumValue = 10, with starting value
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// Sum, sumValue = 10, no starting value
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// Filtering, string concatenation, concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```

The above code is for example reduce() in the first example, the first parameter (blank character) is the starting value, and the second parameter (String::concat) is BinaryOperator. This kind of reduce() with an initial value returns a specific object. As for reduce() with no starting value in the fourth example, since there may not be enough elements, it returns Optional. Please pay attention to this difference. For more information, please see: [IBM: Streams API in Java 8](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html) 

## Parallel Streams

As mentioned earlier, there are two types of streams: serial and parallel. The operations on the serial stream are completed in sequence in one thread, while the parallel stream is executed on multiple threads at the same time.

The following example shows how to improve performance through parallel Stream:

First, we create a large table with no duplicate elements:

```java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```

We sort them in serial and parallel respectively, and finally look at the comparison of the time taken.

### Sequential Sort

```java
// Serial sort
long t0 = System.nanoTime();
long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));
```

```
1000000
sequential sort took: 709 ms // Time used for serial sorting
```

### Parallel Sort

```java
// Parallel sort
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

```

```java
1000000
parallel sort took: 475 ms // Time used for serial sorting
```

The above two codes are almost the same, but the parallel version is about 50% faster. The only change that needs to be done is to change `stream()` to `parallelStream()`.

## Maps

As mentioned earlier, the Map type does not support streams, but Map provides some new and useful methods to handle some daily tasks. The Map interface itself has no available `stream()` method, but you can create a special stream on the key and value or through `map.keySet().stream()`, `map.values().stream()` And `map.entrySet().stream()`.

In addition, Maps supports a variety of new and useful methods to perform common tasks.

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val)); // val0 val1 val2 val3 val4 val5 val6 val7 val8 val9
```

`putIfAbsent` prevents us from writing additional code when checking for null; `forEach` accepts a consumer to operate on each element in the map.

This example shows how to use a function to calculate code on the map:

```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
```

Next, I will show how to delete an item whose key and value all match in the Map:

```java
map.remove(3, "val3");
map.get(3);             // val33
map.remove(3, "val33");
map.get(3);             // null
```

Another useful method:

```java
map.getOrDefault(42, "not found");  // not found
```

It has also become very easy to merge the elements of the Map:

```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9
map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9concat
```

What Merge does is to insert if the key name does not exist, otherwise it merges the value corresponding to the original key and reinserts it into the map.

## Date API

Java 8 includes a brand new date and time API under the `java.time` package. The new Date API is similar to the Joda-Time library, but they are different. The following example covers the most important parts of this new API. The translator made most of the changes to this part of the content with reference to related books.

**Translator's Note (Summary):**

- Clock 类提供了访问当前日期和时间的方法，Clock 是时区敏感的，可以用来取代 `System.currentTimeMillis()` 来获取当前的微秒数。某一个特定的时间点也可以使用 `Instant` 类来表示，`Instant` 类也可以用来创建旧版本的`java.util.Date` 对象。

- 在新API中时区使用 ZoneId 来表示。时区可以很方便的使用静态方法of来获取到。 抽象类`ZoneId`（在`java.time`包中）表示一个区域标识符。 它有一个名为`getAvailableZoneIds`的静态方法，它返回所有区域标识符。

- jdk1.8中新增了 LocalDate 与 LocalDateTime等类来解决日期处理方法，同时引入了一个新的类DateTimeFormatter 来解决日期格式化问题。可以使用Instant代替 Date，LocalDateTime代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat。

  

### Clock

Clock 类提供了访问当前日期和时间的方法，Clock 是时区敏感的，可以用来取代 `System.currentTimeMillis()` 来获取当前的微秒数。某一个特定的时间点也可以使用 `Instant` 类来表示，`Instant` 类也可以用来创建旧版本的`java.util.Date` 对象。

```java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();
System.out.println(millis);//1552379579043
Instant instant = clock.instant();
System.out.println(instant);
Date legacyDate = Date.from(instant); //2019-03-12T08:46:42.588Z
System.out.println(legacyDate);//Tue Mar 12 16:32:59 CST 2019
```

### Timezones(时区)

在新API中时区使用 ZoneId 来表示。时区可以很方便的使用静态方法of来获取到。 抽象类`ZoneId`（在`java.time`包中）表示一个区域标识符。 它有一个名为`getAvailableZoneIds`的静态方法，它返回所有区域标识符。

```java
//输出所有区域标识符
System.out.println(ZoneId.getAvailableZoneIds());

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());// ZoneRules[currentStandardOffset=+01:00]
System.out.println(zone2.getRules());// ZoneRules[currentStandardOffset=-03:00]
```

### LocalTime(本地时间)

LocalTime 定义了一个没有时区信息的时间，例如 晚上10点或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位计算两个时间的时间差：

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
```

LocalTime 提供了多种工厂方法来简化对象的创建，包括解析时间字符串.

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

### LocalDate(本地日期)

LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。

```java
LocalDate today = LocalDate.now();//获取现在的日期
System.out.println("今天的日期: "+today);//2019-03-12
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
System.out.println("明天的日期: "+tomorrow);//2019-03-13
LocalDate yesterday = tomorrow.minusDays(2);
System.out.println("昨天的日期: "+yesterday);//2019-03-11
LocalDate independenceDay = LocalDate.of(2019, Month.MARCH, 12);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println("今天是周几:"+dayOfWeek);//TUESDAY
```

从字符串解析一个 LocalDate 类型和解析 LocalTime 一样简单,下面是使用  `DateTimeFormatter` 解析字符串的例子：

```java
    String str1 = "2014==04==12 01时06分09秒";
        // 根据需要解析的日期、时间字符串定义解析所用的格式器
        DateTimeFormatter fomatter1 = DateTimeFormatter
                .ofPattern("yyyy==MM==dd HH时mm分ss秒");

        LocalDateTime dt1 = LocalDateTime.parse(str1, fomatter1);
        System.out.println(dt1); // 输出 2014-04-12T01:06:09

        String str2 = "2014$$$四月$$$13 20小时";
        DateTimeFormatter fomatter2 = DateTimeFormatter
                .ofPattern("yyy$$$MMM$$$dd HH小时");
        LocalDateTime dt2 = LocalDateTime.parse(str2, fomatter2);
        System.out.println(dt2); // 输出 2014-04-13T20:00

```

再来看一个使用 `DateTimeFormatter` 格式化日期的示例

```java
LocalDateTime rightNow=LocalDateTime.now();
String date=DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
System.out.println(date);//2019-03-12T16:26:48.29
DateTimeFormatter formatter=DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss");
System.out.println(formatter.format(rightNow));//2019-03-12 16:26:48
```

### LocalDateTime(本地日期时间)

LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime 和 LocalTime还有 LocalDate 一样，都是不可变的。LocalDateTime 提供了一些能访问具体字段的方法。

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

只要附加上时区信息，就可以将其转换为一个时间点Instant对象，Instant时间点对象可以很容易的转换为老式的`java.util.Date`。

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

格式化LocalDateTime和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式：

```java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

和java.text.NumberFormat不一样的是新版的DateTimeFormatter是不可变的，所以它是线程安全的。
关于时间日期格式的详细信息在[这里](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)。

## Annotations(注解)

在Java 8中支持多重注解了，先看个例子来理解一下是什么意思。
首先定义一个包装类Hints注解用来放置一组具体的Hint注解：

```java
@interface Hints {
    Hint[] value();
}
@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

Java 8允许我们把同一个类型的注解使用多次，只需要给该注解标注一下`@Repeatable`即可。

例 1: 使用包装类当容器来存多个注解（老方法）

```java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

例 2：使用多重注解（新方法）

```java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

第二个例子里java编译器会隐性的帮你定义好@Hints注解，了解这一点有助于你用反射来获取这些信息：

```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null
Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

即便我们没有在 `Person`类上定义 `@Hints`注解，我们还是可以通过 `getAnnotation(Hints.class) `来获取 `@Hints`注解，更加方便的方法是使用 `getAnnotationsByType` 可以直接获取到所有的`@Hint`注解。
另外Java 8的注解还增加到两种新的target上了：

```java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```

## Where to go from here?

That's it for the new features of Java 8. There are definitely more features waiting to be discovered. There are many useful things in JDK 1.8, such as `Arrays.parallelSort`, `StampedLock` and `CompletableFuture` and so on.

## 公众号

如果大家想要实时关注我更新的文章以及分享的干货的话，可以关注我的公众号。

**《Java面试突击》:** 由本文档衍生的专为面试而生的《Java面试突击》V2.0 PDF 版本[公众号](#公众号)后台回复 **"Java面试突击"** 即可免费领取！

**Java工程师必备学习资源:** 一些Java工程师常用学习资源[公众号](#公众号)后台回复关键字 **“1”** 即可免费无套路获取。 

![我的公众号](https://user-gold-cdn.xitu.io/2018/11/28/167598cd2e17b8ec?w=258&h=258&f=jpeg&s=27334)