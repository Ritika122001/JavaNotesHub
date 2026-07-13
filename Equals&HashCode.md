# equals() and hashCode() Contract

## equals()

### 1. What is equals()?

`equals()` helps compare two objects and checks whether they represent
the same data or not.

By default, the implementation is:

``` java
public boolean equals(Object obj) {
    return (this == obj);
}
```

It simply uses the `==` operator, which compares the **memory
addresses** of the objects.

------------------------------------------------------------------------

### 2. Example

``` java
Person A = new Person("ABC", 100);
Person B = new Person("ABC", 100);
```

Now,

``` java
A.equals(B)
```

Output:

``` text
false
```

Why?

``` text
A ---> Memory Address A
B ---> Memory Address B
```

Different objects = Different memory addresses.

Even though both objects contain the same data, Java says they are **not
equal** because the default implementation compares object references.

------------------------------------------------------------------------

### 3. Then why do we override equals()?

Logically, both objects represent the **same person**, so ideally it
should return `true`.

That's why we override `equals()` and define our own comparison logic.

``` java
@Override
public boolean equals(Object obj) {

    if (this == obj)
        return true;

    if (!(obj instanceof Person))
        return false;

    Person other = (Person) obj;

    return id == other.id &&
           name.equals(other.name);
}
```

Now,

``` java
A.equals(B)
```

returns

``` text
true
```

because we're comparing the **object data** instead of the **memory
addresses**.

# hashCode()

### 1. What is hashCode()?

Its job is to generate an integer hash value for an object.

Example:

``` java
Person p1 = new Person(101, "ABC");
System.out.println(p1.hashCode());   // 4839267

Person p2 = new Person(101, "ABC");
System.out.println(p2.hashCode());   // 8339433
```

Even though both objects have the same data, the hash codes can be
different.

Because the default implementation is generally based on the **object's
identity (memory reference)**, not its contents.

------------------------------------------------------------------------

## Why do we override hashCode()?

``` java
Person A = new Person("ABC", 100);
Person B = new Person("ABC", 100);
```

``` java
@Override
public int hashCode() {
    return Objects.hash(id, name);
}
```

Now,

``` text
A.hashCode() → 12345
B.hashCode() → 12345
```

Since both objects contain the **same data**, they now generate the
**same hash code**.

**Same Data ⇒ Same hashCode()**

------------------------------------------------------------------------

# If equals() compares objects, why do we need hashCode()?

This is where `HashMap` comes into the picture.

When searching or inserting an object:

1.  `hashCode()` is used to calculate the **bucket index** where the
    object should be stored or searched.
2.  Once the correct bucket is found, `equals()` is used to compare the
    objects and find the **exact matching object**.

So they work together:

``` text
hashCode()
        ↓
Find Bucket
        ↓
equals()
        ↓
Find Exact Object
```

### In short

-   `hashCode()` tells **where to look**.
-   `equals()` tells **which object is the correct one**.



# Contract between equals() and hashCode()

### 1. If two objects are equal according to equals(), then their hashCode() must also be the same.

```java
Person A = new Person("ABC", 100);
Person B = new Person("ABC", 100);

A.equals(B);      // true

A.hashCode();     // 12345
B.hashCode();     // 12345
```

If `equals()` returns `true`, then both objects **must** have the same hash code.

---

### 2. If two objects have the same hashCode(), it does NOT guarantee that they are equal.

```java
Person A = new Person("ABC", 100);
Person B = new Person("DEF", 200);

A.hashCode();     // 12345
B.hashCode();     // 12345
```

Even though both objects have the same hash code, they may represent different data.

This is called a **Hash Collision**.

That's why `HashMap` first uses `hashCode()` to find the bucket and then uses `equals()` to identify the exact object.
