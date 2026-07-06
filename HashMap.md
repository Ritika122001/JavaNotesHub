# How HashMap Works Internally

## What is HashMap?

- HashMap is one of the most commonly used collections in the Java Collections Framework.
- It stores data in the form of **Key-Value pairs**.
- Every **key should be unique**, whereas **values can be duplicate**.

Example:

```java
HashMap<Integer, String> map = new HashMap<>();

map.put(1, "Alice");
map.put(2, "Bob");
map.put(3, "Charlie");
```

Output:

```
Key    Value
1   -> Alice
2   -> Bob
3   -> Charlie
```

---

# Internal Working of HashMap

## 1. Bucket

Internally, HashMap maintains an **array of buckets**.

- Every bucket has an **index**.
- Every bucket stores one or more **Nodes**.

Each Node contains:

- Key
- Value
- Hash
- Reference to the next node (if required)

### Imagine HashMap like this

```
Bucket Array

Index
0  --> null

1  --> [Key=101, Value=Alice]

2  --> null

3  --> [Key=105, Value=Bob]
         |
         v
      [Key=109, Value=John]

4  --> null

5  --> [Key=120, Value=David]
```

Each box is called a **Node**.

---

# How does HashMap decide where to store a key?

Suppose we insert

```java
map.put("ABC", 100);
```

---

## Step 1: Calculate HashCode

HashMap first calculates the hash code of the key.

```
Key = "ABC"

hashCode("ABC") = 100
```

---

## Step 2: Calculate Bucket Index

HashMap calculates the bucket index using

```
index = hash & (capacity - 1)
```

Example

```
Capacity = 16

index = 100 & (16 - 1)

index = 100 & 15

index = 4
```

So the entry will be stored inside **Bucket 4**.

> **Note:** HashMap capacity is always a **power of 2** (16, 32, 64, 128...).

---

## Step 3: Go to that Bucket

### Case 1: Bucket is Empty

HashMap simply inserts the new node.

```
Bucket 4

Before

4 --> null

After

4 --> [ABC,100]
```

---

### Case 2: Bucket is Not Empty (Collision)

Suppose another key also maps to Bucket 4.

```
Bucket 4

[ABC,100]
```

Now we insert

```java
map.put("XYZ", 500);
```

If the calculated index is also **4**, then a **collision** occurs.

HashMap now checks

- Does the key already exist using `equals()`?
- If yes → Update the value.
- If no → Add a new node in the same bucket.

```
Bucket 4

[ABC,100]
      |
      v
[XYZ,500]
```

---

# Collision

A **collision** occurs when **different keys produce the same bucket index**.

Example

```
ABC ----\
         \
          Bucket 4

XYZ ----/
```

Both keys are different but stored inside the same bucket.

---

# Which Data Structure is used inside a Bucket?

## Before Java 8

HashMap used a **LinkedList**.

```
Bucket 4

[ABC]
   |
   v
[XYZ]
   |
   v
[PQR]
   |
   v
[DEF]
```

Searching becomes slower because every node has to be traversed one by one.

Time Complexity

```
Worst Case = O(n)
```

---

## After Java 8

If a bucket contains **too many nodes**, HashMap converts the LinkedList into a **Red-Black Tree** (a self-balancing Binary Search Tree).

```
             XYZ
            /   \
         ABC     PQR
                /   \
             MNO    TUV
```

Searching becomes much faster.

```
Worst Case = O(log n)
```

---

# What happens if the Key already exists?

Suppose

```java
map.put(101, "Alice");
```

Again

```java
map.put(101, "Emma");
```

HashMap checks

```
equals()
```

Since the key already exists,

```
Old

101 -> Alice

New

101 -> Emma
```

The value is **updated**, not inserted again.

---

# What if HashMap becomes too full?

HashMap uses **Load Factor** to decide when to increase its size.

Default values

```
Initial Capacity = 16

Load Factor = 0.75
```

Threshold

```
16 × 0.75 = 12
```

Once more than **12 entries** are inserted,

HashMap resizes itself.

```
Before Resize

Capacity = 16

0
1
2
3
...
15
```

↓

```
After Resize

Capacity = 32

0
1
2
3
...
31
```

All existing entries are moved into the new bucket array.

This process is called **Rehashing**.

Capacity grows like

```
16
 ↓
32
 ↓
64
 ↓
128
 ↓
256
```

---

# Complete Flow

```
put(key, value)

        │
        ▼
Calculate hashCode()

        │
        ▼
Find Bucket Index

index = hash & (capacity - 1)

        │
        ▼
Go to Bucket

        │
 ┌──────┴──────┐
 │             │
 ▼             ▼

Empty       Not Empty
 │             │
 ▼             ▼

Insert     Compare Keys
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼

Key Found        Key Not Found
    │                  │
    ▼                  ▼

Update Value     Add New Node
```

---

# Time Complexity

| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| `put()` | O(1) | O(log n) |
| `get()` | O(1) | O(log n) |
| `remove()` | O(1) | O(log n) |

---

# Summary

- HashMap stores data as **Key-Value pairs**.
- Internally, it maintains an **array of buckets**.
- `hashCode()` is used to calculate the bucket index.
- Different keys mapping to the same bucket is called a **collision**.
- `equals()` is used to identify the correct key within a bucket.
- Before Java 8, collisions were handled using a **LinkedList**.
- From Java 8 onwards, heavily populated buckets are converted into a **Red-Black Tree** for better performance.
- HashMap uses a **Load Factor** to decide when to resize.
- During resizing, all entries are moved into a new bucket array through **Rehashing**.
