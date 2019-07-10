# Notes of Effective Java 2nd Edition
### 1. Consider static factory instead of constructors.
Static factory method returns an instance of the class, however it is not the same as Factory Pattern. One example is `Boolean.valueOf(boolean b)`. It is not required to create a new object each time when invoked.
### 2. Consider a builder when faced with many constructor params
The Builder Pattern is a good choice when designing classes which have more than a handful of instantiation params.
### 3. Enforce the singleton property with a private constructor or an enum type
A singleton is a class that is instantiated exactly once.
### 4. Enforce noninstantiability with a private constructor
A class can be made noninstantiable by including a private constructor.
```
public class FooClass {
  private FooClass() { throw new AssertionError(); }
}
```
### 5. Avoid creating unnecessary objects
### 6. Eliminate obsolete object references
### 7. Avoid finalizers
Finalizers are unpredictable, often dangerous and generally unnecessary, and there is a severe performance penalty for using finalizers.
### 8. Obey general contract when overriding equals
The overriding must satisfy the following requirements:
- Reflexive: `x.equals(x)`
- Symmetric: `x.equals(y)` is same as `y.equals(x)`
- Transitive: `x.equals(y)` and `y.equals(z)` give you `x.equals(z)`
- Consistent: always return the same result regardless of how many times it's been called.
- Not null: `x.equals(null)` should always return `false`.
### 9. Always override hashCode when you override equals
`hashCode` is an integer which is generated from the address of the object, the purpose of it is to allow object to be saved as key in a `map`. Also when you compare two objects, you are essentially comparing the `hashCode`.
### 10. Always override toString
`toString` returns a string formatted as `{className}@{hexRepresentationOfHashCode}`, but sometimes it could be much more useful if `toString` gives you custom information, e.g. JSON representation.
### 11. Override clone judiciously
In practice, a class that implements `Cloneable` is expected to provide a properly functioning public `clone` method.
### 12. Consider implementing Comparable
`Comparable` class, or class implemented with `compareTo`, can be used in sorting methods, e.g. `Arrays.sort(a)`.
### 13. Minimize the accessibility of classes and members
A well-designed module hides all of its implementation details, cleanly separating its API from its implementation.
- private: only accessible in the class its own.
- package-private: only accessible from any class in the same package.
- protected: accessible from subclasses and any class in the same package.
- public: accessible from anywhere.
### 14. Use accessor methods not public fields in public classes
- Getter and setters.
### 15. Minimize mutability
Immutable classes are less prone to error and are more secure. Rules to make a class immutable:
- Don't provide method that modifies state.
- Ensure that the class can't be extended.
- Make all fields final.
- Make all fields private.
- Ensure exclusive access to any mutable components.
### 16. Favor composition over inheritance
- Inheritance violates encapsulation, it is powerful but yet overused.
- Overriding is not safe and easy-maintainable.
- In a nutshell, composition allows us to model objects that are made up of other objects, thus defining a `has-a` relationship between them. Furthermore, the composition is the strongest form of association, which means that the object(s) that compose or are contained by one object are destroyed too when that object is destroyed.
```
public class Computer {

    private Processor processor;
    private Memory memory;
    private SoundCard soundCard;

    // standard getters/setters/constructors

    public Optional<SoundCard> getSoundCard() {
        return Optional.ofNullable(soundCard);
    }
}
```
### 17. Design and document for inheritance or else prohibit it
### 18. Prefer interfaces to abstract classes
An interface is generally the best way to define a type that permits multiple implementations, but it is very important to design it correctly at the beginning.
### 19. Use interfaces only to define types
Do not use it as constant interface.
### 20. Prefer class hierarchies to tagged classes
Tagged classes are verbose, error-prone and inefficient.
```
   class Figure {
       enum Shape { RECTANGLE, CIRCLE };
       // ...
   }
```

Instead, we should use

```
abstract class Figure {
  // ...
}

class Rectangle extends Figure {
   // ...
}
```

### 21. Use function objects to represent strategies
A primary use of function pointers is to implement the Strategy pattern. To implement this pattern in Java, declare an interface to represent the strategy, and a class that implements this interface for each concrete strategy. When a concrete strategy is used only once, it is typically declared and instantiated as an anonymous class.

```
   class StringLengthComparator {
       public int compare(String s1, String s2) {
           return s1.length() - s2.length();
       }
}
```
### 22. Favor static member classes over nonstatic
A nested class should only exist to serve its enclosing class. There are 4 kinds of such classes: static member class, nonstatic member class, anonymous class and local class.
e.g. anonymous class

```
Test t = new Test()
{
   // data members and methods
   public void test_method()
   {
      ........
      ........
    }
};
```

## Generics
### 23. Don't use raw types in new code
```
// Uses raw type (List) - fails at runtime!
public static void main(String[] args) {
  List<String> strings = new ArrayList<String>();
  unsafeAdd(strings, new Integer(42));
  String s = strings.get(0); // Compiler-generated cast
}

private static void unsafeAdd(List list, Object o) {
  list.add(o);
}
```

### 24. Eliminate unchecked warnings
Every time you use an `@SuppressWarnings("unchecked")` annotation, add a comment saying why it is safe to do so.

### 25. Prefer lists to arrays
Arrays are covariant. E.g. if `Sub` is a subtype of `Super`, then array type `Sub[]` is a subtype of `Super[]`. Lists, or generics, on the other hand, are invariant - they are neither subtype nor supertype.

### 26. Favor generic types
An generic Stack:

```
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  // you can't create an array of a non-reifiable type,
  // e.g. elements = new E[FOO_BAR]
  // you can suppress unchecked warning once typesafe/cast is proved.
  @SuppressWarnings("unchecked")
  public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
  }
  public void push(E e) { ensureCapacity(); elements[size++] = e;
  }
  public E pop() { if (size==0)
    throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference return result;
  }
  ... // no changes in isEmpty or ensureCapacity
}
```
### 27. Favor generic methods
e.g.

```
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<E>(s1);
  result.addAll(s2);
  return result;
}
```

And there is also generic static factory method:

```
public static <K,V> HashMap<K,V> newHashMap() {
  return new HashMap<K,V>();
}
```

And generic singleton factory:

```

public interface UnaryFunction<T> {
  T apply(T arg);
}

// Generic singleton factory pattern
private static UnaryFunction<Object> IDENTITY_FUNCTION =
  new UnaryFunction<Object>() {
    public Object apply(Object arg) { return arg; }
  };

// IDENTITY_FUNCTION is stateless and its type parameter is
// unbounded so it's safe to share one instance across all types.
@SuppressWarnings("unchecked")
public static <T> UnaryFunction<T> identityFunction() {
  return (UnaryFunction<T>) IDENTITY_FUNCTION;
}
```

### 28. Use bounded wildcards to increase API flexibility
e.g. for a generic Stack class

```
public class Stack<E> {
       public Stack();
       public void push(E e);
       public E pop();
       public boolean isEmpty();
}
```

we can use wildcard as such:

```

// Wildcard type for parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e);
}


// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.

### 29. Consider typesafe heterogeneous containers
```
public class Favorites {
  private Map<Class<?>, Object> favorites =
    new HashMap<Class<?>, Object>();
  public <T> void putFavorite(Class<T> type, T instance) {
    if (type == null)
      throw new NullPointerException("Type is null");
    favorites.put(type, instance);
  }
  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```

### 30. Use enums instead of int constants
Enums provide compile-time type safety.
Use

```
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
over int enum pattern, which creates ambiguity of values.

```
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
```

### 31. Use instance fields instead of ordinals
`ordinal()` method returns the numeric position of each enum, it is very hard to maintain. Therefore, store the position value in instance field.

```
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
  SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
  NONET(9), DECTET(10), TRIPLE_QUARTET(12);

  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```

### 32. Use EnumSet instead of bit fields
### 33. Use EnumMap instead of ordinal indexing
### 34. Emulate extensible enums with interfaces
### 35. Use annotations to naming patterns
There is simply no reason to use naming patterns now that we have annotations.
### 36. Consistently use the Override annotation
e.g.

```
public boolean equals(Foo f)
```

does not override

```
public boolean equals(Object o)
```

it actually overloads it. Therefore it could be problematic. Make sure to annotate with `@override`.

### 37. Use marker interfaces to define types
A marker interface is an interface that contains no method declaration.

### 38. Check parameters for validity
e.g. index value must be non-negative.

### 39. Make defensive copies when needed
- You must program defensively, with the assumption that clients of your class will do their best to destroy its invariants.
- defensive copies are made before checking the validity of the parameters (Item 38), and the validity check is performed on the copies rather than on the originals.
- do not use the clone method to make a defensive copy of a parameter whose type is subclassable by untrusted parties.

### 40. Design method signatures carefully
Be careful with method names, providing convenience methods, long param lists.

### 41. Use overloading judiciously
- The choice of which overloading to invoke is made at compile time.
- selection among overloaded methods is static, while selection among overridden methods is dynamic.

### 42. Use varargs judiciously
```
static int sum(int... args) {
  for (int arg : args) {
   // ...
  }
  // ...
}
```
Do not assume the size of args.

### 43. Return empty arrays or collections, not nulls

### 44. Write doc comments for all exposed API elements

## General Programming
### 45. Minimize the scope of local variables
- The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.
- Nearly every local variable declaration should contain an initializer.
- Therefore prefer `for` loops to `while` loops.

### 46. Prefer for-each loops to traditional for loops
After Java 1.5,
```
for (Element e : elements) {
  doSomething(e);
}
```

### 47. Know and use the libraries
Every programmer should be familiar with the contents of `java.lang`, `java.util`, and, to a lesser extent, `java.io`. Also `java.util.concurrent` is very important.

### 48. Avoid float and double if exact answers are required
The float and double types are particularly ill- suited for monetary calculations because it is impossible to represent 0.1 (or any other negative power of ten) as a float or double exactly.
e.g.
```
System.out.println(1.03 - .42);
```

gives `0.6100000000000001`. don't use float or double for any calculations that require an exact answer.

### 49. Prefer primitive types to boxed primitives
Every primitive type has a corresponding reference type, called a boxed primitive. The boxed primitives corresponding to int, double, and boolean are Integer, Double, and Boolean.
Boxed primitives have distinct identities, nonfunctional value `null`(including in Integer etc.), and less time/space efficient than primitives.

### 50. Avoid strings where other types are more appropriate
Strings are poor substitutes for other value types.

### 51. Beware the performance of string concatenation
Because strings are immutable, string concatenation means two strings are copied and concatenated into a new string. Therefore for an acceptable performance, use `StringBuilder` in place of string.
```
public String statement() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++)
    b.append(lineForItem(i));
  return b.toString();
}

```

### 52. Refer to objects by their interfaces
Use
```
List<Subscriber> subscribers = new Vector<Subscriber>();
```

rather than

```
Vector<Subscriber> subscribers = new Vector<Subscriber>();
```

### 53. Prefer interfaces to reflection
By using reflection:
- You lose benefits of compile-time type checking, because it can be used to invoke nonexistent or inaccessible methods.
- It adds performance overhead, much slower than normal method invocation.

As a rule, objects should not be accessed reflectively in normal applications at runtime.

### 54. Use native methods judiciously
Java Native Interfaces(JNI) allows direct call to native methods, which are written in native language such as C/C++.
- May cause memory corruption.
- Previously it was mostly used for better performance, but nowadays it loses such advantage. e.g. BigInteger is rewritten in JAVA now.

### 55. Optimize judiciously
- Strive to write good programs rather than fast ones.
- Strive to avoid design decisions that limit performance.
- Consider the performance consequences of your API design decisions.
- Do profiling.

### 56. Adhere to generally accepted naming conventions
### 57. Use exceptions only for exceptional conditions
Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.

### 58. Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
Use checked exceptions for recoverable conditions and runtime exceptions for programming errors.

### 59. Avoid unnecessary use of checked exceptions
### 60. Favor the use of standard exceptions
- `IllegalArgumentException` : Non-null parameter value is inappropriate
- `IllegalStateException` : Object state is inappropriate for method invocation
- `NullPointerException` : Parameter value is null where prohibited
- `IndexOutOfBoundsException` : Index parameter value is out of range
- `ConcurrentModificationException` : Concurrent modification of an object has been detected where it is prohibited
- `UnsupportedOperationException` : Object does not support method

### 61. Throw exceptions appropriate to the abstraction
Higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction. This idiom is known as exception translation:
```
// Exception Translation
try {
  // Use lower-level abstraction to do our bidding
  ...
} catch(LowerLevelException e) {
  throw new HigherLevelException(...);
}
```

### 62. Document all exceptions thrown by each method
Use the Javadoc @throws tag to document each unchecked exception that a method can throw, but do not use the throws keyword to include unchecked exceptions in the method declaration.

### 63. Include failure-capture information in detailed messages
To capture the failure, the detail message of an exception should contain the values of all parameters and fields that contributed to the exception.

### 64. Strive for failure atomicity
Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation.

### 65. Don't ignore exceptions
i.e. do not use empty `catch` block.

### 66. Synchronize access to shared mutable data
- `synchronized` ensures that only one thread can execute a method or block at one time. Not only does synchronization prevent a thread from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.
- Synchronization is required for reliable communication between threads as well as for mutual exclusion.
- Synchronization has no effect unless both read and write operations are synchronized.
<b> When multiple threads share mutable data, each thread that reads or writes the data must perform synchronization.</b>

### 67. Avoid exccessive synchronization
- To avoid liveness and safety failures, never cede control to the client within a synchronized method or block.
- As a rule, you should do as little work as possible inside synchronized regions.

### 68. Prefer executors and tasks to threads
```
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable);
executor.shutdown();
```
This class allowed clients to enqueue work items for asynchronous processing by a background thread. If you want more than one thread to process requests from the queue, simply call a different static factory that creates a different kind of executor service called a thread pool.

### 69. Prefer concurrency utilities to wait and notify
- Given the difficulty of using wait and notify correctly, you should use the higher-level concurrency utilities instead.
- The concurrent collections provide high-performance concurrent implementations of standard collection interfaces such as List, Queue, and Map. To provide high concurrency, these implementations manage their own synchronization internally.locking it will have no effect but to slow the program.

### 70. Document thread safety
To enable safe concurrent use, a class must clearly document what level of thread safety it supports.
- immutable: instance of this class appears constant; no external synchronization is needed. e.g. String, Long and BigInteger.
- unconditionally thread safe: instance of this class are mutable, but it has sufficient internal synchronization that can be used concurrently. e.g. `Random` and `ConcurrentHashMap`.
- conditionally thread safe: some of them require external synchronization for safe concurrent use. e.g. `Collections.synchronized` wrappers.
- not thread-safe: instances of class are mutable. Each method must be enclosed by external synchronization lock.
- thread-hostile: there are very few of them, just don't use it concurrently. e.g. `System.runFinalizersOnExit`.

### 71. Use lazy initialization judiciously
Under most circumstances, normal initialization is preferable to lazy initialization. However if the initialization is costly, lazy loading has its use.

### 72. Don't depend on the thread scheduler
- Any program that relies on the thread scheduler for correctness or performance is likely to be nonportable.
- The best way to write a robust, responsive, portable program is to ensure that the average number of runnable threads is not significantly greater than the number of processors.

### 73. Avoid thread group
It was designed for applets for security purposes, and never worked as expected, and now it's obsolete.

### 74. Implement Serializable judiciously
- A major cost of implementing `Serializable` is that it decreases the flexi- bility to change a class's implementation once it has been released.
- it increases the likelihood of bugs and security holes.
- it increases the testing burden associated with releasing a new version of a class.

Classes designed for inheritance should rarely implement `Serializable`, and interfaces should rarely extend it.

### 75. Consider using a custom serialized form
- Do not accept the default serialized form without first considering whether it is appropriate.
- Use `readObject` to avoid null values, ensure invariants and security.

### 76. Write readObject methods defensively
e.g.
```
// readObject method with validity checking
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    // Check that our invariants are satisfied
    if (start.compareTo(end) > 0)
      throw new InvalidObjectException(start +" after "+ end);
}
```

### 77. For instance control, prefer enum types to readResolve
### 78. Consider serialization proxies instead of seriliazed instances
