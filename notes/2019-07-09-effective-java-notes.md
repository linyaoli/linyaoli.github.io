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

