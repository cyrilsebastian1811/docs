# __:material-language-java: Java__

## Generics

- Generics in Java allow you to write code that works with different types.
- Why use:
    - Type Safety: Catch errors at compile time.
    - Code Reuse: Write logic once for many types.
    - Eliminate Casting: No need for explicit type casts.
    - Better Readability: Self-documenting code.

```java
// Generic class
// <T>: declares T is generic type
class Box<T> {
    private T value;

    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<String> stringBox = new Box<>();

class Calculator {
    // Generic Method
    // Here, Type parameter T not declared at class level
    public <T extends Number> double add(T a, T b) {
        return a.doubleValue() + b.doubleValue();
    }
}
```

| Feature                     | Generic Class                                                 | Generic Method                                |
| --------------------------- | ------------------------------------------------------------- | --------------------------------------------- |
| Definition Scope        | Type parameter is defined at the class level                  | Type parameter is defined at the method level |
| Reusability             | All methods in the class can use the same type                | Only that method uses the generic type        |
| When to Use             | When the same type is used across multiple methods            | When only one method needs to be generic      |
| Syntax                  | `class Box<T> { ... }`                                        | `<T> T identity(T value) { ... }`             |
| Type parameter lifetime | Exists for the life of the object                             | Exists only during the method call            |
| Can be static?          | Can't be used in static methods | Can be used in static methods            |


### Wildcards Generics

- Wildcards allow more flexible method parameters

=== "Unbounded Wildcard: `<?>`"

    ```java
    List<?> anything = List.of("abc", 123, true);
    ```

=== "Upper Bounded Wildcard: `<? extends T>`"

    ```java
    // Accepts List<Integer>, List<Double>, etc.
    public void printAll(List<? extends Number> numbers) { ... }
    ```

=== "Lower Bounded Wildcard: `<? super T>`"

    ```java
    // Accepts List<Integer>, List<Number>, List<Object>
    public void addTo(List<? super Integer> list) {
        list.add(42);
    }
    ```


## Lambda Expressions

- A lambda is an anonymous function — it lets you treat functionality as method arguments.

```java
List<String> names = Arrays.asList("A", "B", "C");
names.forEach(name -> System.out.println(name));
```

## Function Interfaces

- An interface with exactly one abstract method, used as the basis for lambda expressions.
- A functional interface is an interface that has exactly one abstract method.
- `@FunctionalInterface` is optional, but recommended — it tells the compiler to ensure that the interface has only one abstract method.
- ==Can have default and static methods too — as long as there's only one abstract method.==

```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b);
}

public class Main {
    public static void main(String[] args) {
        MathOperation add = (a, b) -> a + b;
        MathOperation multiply = (a, b) -> a * b;

        System.out.println(add.operate(5, 3));       // 8
        System.out.println(multiply.operate(5, 3));  // 15
    }
}
```

## Stream API

- The Stream API is used to process collections (like List/Set) in a functional style — with support for operations like map, filter, reduce, etc.

```java
List<String> names = Arrays.asList("Tom", "Jerry", "Tim");
names.stream().filter(n -> n.startsWith("T")).forEach(System.out::println);

// map() transforms each element
Stream<String> upper = names.stream().map(String::toUpperCase);

// flatMap() flattens nested streams
Stream<String> all = listOfLists.stream().flatMap(List::stream);
```

### Terminal operations

- Terminal operations end the stream pipeline.
- They produce a result or side-effect.
- Once a terminal operation is called, the stream can’t be reused.
- Types:
    - Result-Producing (Reduction)
    - Matching / Predicate-Based
    - Side-Effect

=== "Result-Producing (Reduction)"

    | Method            | Description                                  | Example                           |
    | ----------------- | -------------------------------------------- | --------------------------------- |
    | `collect()`       | Reduces stream into a collection or result   | `.collect(Collectors.toList())`   |
    | `reduce()`        | General-purpose reduction (e.g. sum, concat) | `.reduce((a, b) -> a + b)`        |
    | `count()`         | Returns the number of elements               | `.count()`                        |
    | `min()` / `max()` | Returns min or max element using comparator  | `.min(Comparator.naturalOrder())` |
    | `findFirst()`     | Returns the first element (if any)           | `.findFirst()`                    |
    | `findAny()`       | Returns any element (good for parallel)      | `.findAny()`                      |

=== "Matching / Predicate-Based"

    | Method        | Description                                 | Example                  |
    | ------------- | ------------------------------------------- | ------------------------ |
    | `anyMatch()`  | `true` if **any** element matches predicate | `.anyMatch(x -> x > 10)` |
    | `allMatch()`  | `true` if **all** elements match            | `.allMatch(x -> x > 0)`  |
    | `noneMatch()` | `true` if **no** elements match             | `.noneMatch(x -> x < 0)` |

=== "Side-Effect"

    | Method             | Description                                                                   | Example                         |
    | ------------------ | ----------------------------------------------------------------------------- | ------------------------------- |
    | `forEach()`        | Applies an action to each element (order not guaranteed for parallel streams) | `.forEach(System.out::println)` |
    | `forEachOrdered()` | Applies action in encounter order                                             | `.forEachOrdered(...)`          |


??? note "`findFirst()` v/s `findAny()`"

    - Both are terminal operations in Java Streams that return an element from the stream wrapped in an `Optional<T>`.

    | Feature         | `findFirst()`                            | `findAny()`                                      |
    | --------------- | ---------------------------------------- | ------------------------------------------------ |
    | Guarantees      | Returns the __first__ element (in order) | Returns __any__ element (no guarantee of order)  |
    | Parallel Stream | May be slower (preserves order)          | Can be __faster__ (doesn't preserve order)       |
    | Return Type     | `Optional<T>`                            | `Optional<T>`                                    |
    | Use Case        | When __order matters__                   | When __any match is fine & performance matters__ |

    ```java
    List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

    // findFirst example
    Optional<String> first = names.stream()
                                .filter(name -> name.startsWith("C"))
                                .findFirst();
    first.ifPresent(System.out::println);

    // findAny example
    Optional<String> any = names.parallelStream()
                                .filter(name -> name.startsWith("C"))
                                .findAny();
    any.ifPresent(System.out::println);  // Could be Charlie — or something else, if unordered
    ```

## Optional

- `Optional` is a container to avoid null pointer exceptions.
- Why Use:
    - Helps avoid NullPointerException.
    - Promotes null safety by making you explicitly deal with missing values.
    - Encourages more declarative, functional-style code.

```java
// Creates an Optional with a non-null value.
// Will throw NullPointerException if passed null.
Optional<String> opt = Optional.of("Hello");

// Creates an Optional that may hold a null value.
Optional<String> opt = Optional.ofNullable(maybeNull);

// Creates an empty Optional.
Optional<String> emptyOpt = Optional.empty();

// Returns true if a value is present.
if (opt.isPresent()) {
    System.out.println("Value exists");
}

// Returns true if the Optional is empty.
if (opt.isEmpty()) {
    System.out.println("No value present");
}

// Returns the value if present, else throws NoSuchElementException.
String value = opt.get();  // ⚠️ Be careful; always check isPresent() first.

// Executes the lambda only if a value is present.
opt.ifPresent(System.out::println);

// Like ifPresent, but runs a fallback if value is absent.
opt.ifPresentOrElse(
    val -> System.out.println(val),
    () -> System.out.println("No value")
);

// Throws an exception if value is missing.
String value = opt.orElseThrow();  // Throws NoSuchElementException
String value2 = opt.orElseThrow(() -> new IllegalStateException("No value"));

// Transforms the value if present, returns a new Optional.
Optional<Integer> nameLength = opt.map(String::length);

// Like map(), but avoids nested Optionals.
Optional<String> name = Optional.of("Alice");
Optional<String> upper = name.flatMap(n -> Optional.of(n.toUpperCase()));

// Combining Optionals
Optional<String> opt1 = Optional.of("data");
Optional<String> result = opt1
    .filter(d -> d.length() > 3)
    .map(String::toUpperCase)
    .or(() -> Optional.of("DEFAULT"));
```

## Default and Static methods in interfaces

They allow you to add new methods to interfaces without breaking existing implementations.

```java
interface Vehicle {
    default void start() {
        System.out.println("Starting...");
    }
}
```


## Method references

- A shorthand for calling a method using :: operator.
- It’s a method reference, which is a shorthand for a lambda expression that calls a method.
- e.g. `String::toUpperCase` is equivalent to `(str) -> str.toUpperCase()`
- `String::toUpperCase` tells the compiler. For each element in the stream (a String), call `toUpperCase()` on it.
- You can use it when:
    - The method takes one argument (the stream element),
    - And the method belongs to the type of that argument.
- Types:
    - Static (Class::staticMethod)
    - Instance (object::instanceMethod)
    - Constructor (Class::new)

    
=== "Static"

    ```java
    public class Utils {
        public static int square(int x) {
            return x * x;
        }
    }

    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

    // Using method reference
    List<Integer> squares = numbers.stream()
        .map(Utils::square)
        .collect(Collectors.toList());
    ```

=== "Instance"

    ```java
    public class Greeter {
        public void greet(String name) {
            System.out.println("Hello, " + name);
        }
    }

    Greeter greeter = new Greeter();
    List<String> names = Arrays.asList("Alice", "Bob");

    // Using method reference
    names.forEach(greeter::greet);
    ```

=== "Constructor"

    ```java
    public class Person {
        String name;
        public Person(String name) {
            this.name = name;
        }
    }

    List<String> names = Arrays.asList("Alice", "Bob");

    // Convert names to Person objects
    List<Person> people = names.stream()
        .map(Person::new)
        .collect(Collectors.toList());
    ```


## Comparable v/s Comparator

=== "Comparable"

    - Used when an object knows how to compare itself to another object.
    - Defined inside the class.
    - ==Used to define single internal sort orders (i.e. natural order).==

    ```java hl_lines="15-16"
    public class Student implements Comparable<Student> {
        int id;
        String name;

        public Student(int id, String name) {
            this.id = id; this.name = name;
        }

        @Override
        public int compareTo(Student other) {
            return this.id - other.id;  // sort by ID
        }
    }

    List<Student> list = ...;
    Collections.sort(list);  // Uses compareTo()
    ```

=== "Comparator"

    - Used to define multiple or external sort orders (e.g., by name, by date, etc.).
    - Defined outside the class (or inline).

    ```java
    Comparator<Student> nameComparator = (s1, s2) -> s1.name.compareTo(s2.name);
    Collections.sort(list, nameComparator);

    // or

    list.stream()
        .sorted(nameComparator)
        .forEach(System.out::println);
    ```