# final vs finally vs finalize

## final

### Definition
- `final` is a keyword that prevents modification.
- It is used to make a variable constant, prevent method overriding, or prevent class inheritance.

### Where can it be used?
- Variable
- Method
- Class

### Types

#### Final Variable
- Once a value is assigned, it cannot be changed.

Example:
```java
private final int a = 10;
```

Here, `a` is assigned the value `10` and it cannot be modified later.

#### Final Method
- A final method cannot be overridden by a child class.

Example:
```java
class A {
    public final void display() {
        System.out.println("Hello");
    }
}

class B extends A {
    public void display() {   // ❌ Compile-time error
        System.out.println("Hi");
    }
}
```

#### Final Class
- A final class cannot be extended.

Example:
```java
final class A {
}

// class B extends A {} // Compilation Error
```

### Real-time Usage
- In Spring Boot, `final` is commonly used for **constructor-based Dependency Injection**.

Example:

```java
class B {

    private final A a;

    public B(A a) {
        this.a = a;
    }
}
```

Since `a` is `final`, it can only be assigned once inside the constructor.

---

## finally

### Definition
- `finally` is a block used with `try` and `catch`.
- It usually executes whether an exception occurs or not.

### Why do we use it?
- To close resources like:
  - Database connections
  - Files
  - Input/Output streams

### Example

```java
try {
    int a = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Exception caught");
} finally {
    System.out.println("Finally block executed");
}
```

### Output

```
Exception caught
Finally block executed
```

### Points to remember 💡
- `finally` executes even if there is a `return` statement inside `try` or `catch`.
- It may not execute if the JVM terminates unexpectedly (for example, `System.exit()`).
- In modern Java, **try-with-resources** is preferred because it closes resources automatically.

---

## finalize()

### Definition
- `finalize()` is a method of the `Object` class.
- It was used to perform cleanup before an object is garbage collected.

### Example

```java
class Test {

    @Override
    protected void finalize() {
        System.out.println("Finalize called");
    }
}

Test t = new Test();
t = null;

Will finalize() run immediately?

❌ No.

Will it run after 5 seconds?

❌ Not guaranteed.

Will it definitely run before the program ends?

❌ Not guaranteed.

That's why developers stopped using it.
```

### Points to remember 💡
- `finalize()` is **deprecated**.
- There is **no guarantee** that it will be executed.
- It should not be used in new applications.
- Modern Java recommends using **try-with-resources** or explicit resource cleanup.

---

# Difference Between final, finally and finalize()

| Feature | final | finally | finalize() |
|---------|--------|----------|------------|
| Type    | Keyword | Block | Method |
| Used With | Variable, Method, Class | try-catch | Object class |
| Purpose | Prevent modification | Execute cleanup code | Cleanup before garbage collection |
| Status  | Used | Used | Deprecated |

---

# Points to remember 💡

- **final** → Prevents modification.
- **finally** → Executes cleanup code after `try` and `catch`.
- **finalize()** → Deprecated method that was intended to run before garbage collection.
