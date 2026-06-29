# Why ConcurrentHashMap is required when HashMap is present?

## HashMap

If you are aware of HashMap's internal working, it becomes easier to understand why `ConcurrentHashMap` was introduced.

Suppose multiple threads are trying to modify a `HashMap`.

**Example**

```
T1 -> map.put(1, "A")
T2 -> map.put(2, "B")
```

Suppose both keys are mapped to the same bucket (Bucket 5).

Now both threads start executing simultaneously.

* T1 starts inserting its node (1 -> A)
* Before T1 completes, T2 also starts modifying the same bucket (2 -> B)

Since `HashMap` is **not thread-safe**, multiple threads can modify the same bucket at the same time. This may lead to:

* Race Condition
* Lost Updates
* Data Corruption
* In older Java versions, resizing could even create infinite loops.

Therefore, `ConcurrentHashMap` was introduced to support safe concurrent modifications.

---

# ConcurrentHashMap before Java 8

Before Java 8, `ConcurrentHashMap` used the concept of **Segments**.

1. The bucket array was divided into multiple segments.
2. Each segment behaved like an independent `HashMap`.
3. Every segment had its own lock.

Example

```
Segment 0 -> Buckets 1 - 5
Segment 1 -> Buckets 6 - 10
```

Suppose:

```
T1 wants to insert (1 -> A)
```

and that bucket belongs to **Segment 0**.

Now **Segment 0** gets locked.

* Any other thread trying to modify data inside **Segment 0** has to wait.
* But threads working on **Segment 1** can continue their execution.

Although this was much better than locking the entire map, concurrency was still limited because an entire segment was locked even if only one bucket was being modified.

---

# ConcurrentHashMap after Java 8

Java 8 removed the Segment concept.

Instead, it introduced Bucket-level synchronization along with CAS (Compare-And-Swap).

- If the target bucket is empty, ConcurrentHashMap first tries to insert the node using CAS without acquiring any lock.
- If the bucket already contains nodes, only that particular bucket is synchronized for modification.

This improves concurrency and scalability compared to the Segment-based approach.

Example

```
Bucket 1 🔒   <- Thread A

Bucket 5 🔒   <- Thread B

Bucket 8      <- Thread C (Read)

✅ Parallel Execution
```

Here,

* Thread A can modify Bucket 1.
* Thread B can modify Bucket 5 at the same time.
* If another thread wants to modify Bucket 1, it has to wait until Thread A completes.
* Since different buckets have different locks, multiple threads can work in parallel.

This improves scalability and overall performance.

---

# Read Operation

ConcurrentHashMap uses `volatile` internally to ensure memory visibility.

This means any update made by one thread becomes visible to other threads.

Therefore, read operations (`get()`) can read the latest consistent value without acquiring locks in most cases.
---

# Compare-And-Swap (CAS)

When inserting into an empty bucket, `ConcurrentHashMap` first tries to use **CAS (Compare-And-Swap)** instead of acquiring a lock.

## Normal Operation

Suppose

```
x = 10
```

Thread T1 wants

```
10 → 20
```

Thread T2 wants

```
10 → 30
```

Execution

* T1 checks if x == 10.
* Before T1 updates it, T2 changes x to 30.
* T1 then updates x to 20.

Here, T2's update is lost because both operations were not atomic.

---

## Compare-And-Swap

Conceptually,

```
IF currentValue == expectedValue

THEN update it to newValue

ELSE

Do nothing.
```

Example

```
Expected = 10

Current = 10

↓

Update to 20
```

If another thread has already changed the value,

```
Expected = 10

Current = 30
```

CAS fails.

No update happens.

The thread reads the latest value and retries if required.

Since comparison and update happen as one atomic operation, race conditions are avoided without locking.

---

### What happens if the bucket is not empty?

If the target bucket is empty, ConcurrentHashMap first tries to insert the node using CAS.

However, if the bucket already contains one or more nodes, CAS cannot be used directly.

In that case, ConcurrentHashMap synchronizes only that particular bucket and performs the insertion or update.

So internally:

- Empty Bucket → CAS (No Lock)
- Non-Empty Bucket → Bucket-level Synchronization

---

#Proper Example for CAS

Case 1: Empty Bucket → CAS (No Lock Required)

```Initially

Bucket[5]
   │
   ▼
  null


Thread A wants to insert (10 -> "A")

            CAS
(Expected = null, New = Node(10,"A"))

            │
            ▼

Is Bucket[5] still null?

        YES ✅

            │
            ▼

Bucket[5]
   │
   ▼
Node(10,"A")

✔ Node inserted successfully
✔ No lock acquired
```

Case 2: Two Threads Insert into Empty Bucket

```Initially

Bucket[5]
   │
   ▼
  null


          Thread A                  Thread B

CAS(null, NodeA)              CAS(null, NodeB)

        │                           │
        └──────────┬────────────────┘
                   ▼

         Bucket[5] == null ?

                   │
                   ▼

        Thread A wins the CAS

Bucket[5]
   │
   ▼
 NodeA


Now Thread B checks...

Expected = null
Actual   = NodeA

        ❌ CAS Fails
```

Thread B does NOT overwrite NodeA.
It retries using the synchronization path.

---

# Null Handling

`HashMap` allows `null` keys and `null` values.

Example:

```java
map.put(1, null);
```

Now suppose two threads are running.

**Thread T1**

```java
map.get(1);
```

**Thread T2**

```java
map.remove(1);
```

Execution:

* Initially, key `1` is present with value `null`.
* Before T1 executes `get(1)`, T2 removes the key.
* Now T1 executes `get(1)` and gets `null`.

Here, T1 cannot identify the reason for receiving `null`.

Is it because:

* the key exists but its value is `null`?

OR

* the key was removed and is no longer present?

Both situations return `null`, creating ambiguity.

---

In a single-threaded application, this can be handled using:

```java
if (map.containsKey(1)) {
    map.get(1);
} else {
    System.out.println("Key not present");
}
```

Since no other thread can modify the map between `containsKey()` and `get()`, the result is reliable.

However, in a multi-threaded environment, another thread may remove the key after `containsKey()` returns `true` but before `get()` is executed.

This again makes the result ambiguous.

Therefore, `ConcurrentHashMap` does **not** allow `null` keys or `null` values.

Now, if:

```java
map.get(key)
```

returns `null`, it has only one meaning:

> **The key is not present in the map.**

---

# Summary

* `HashMap` is **not thread-safe**.
* Multiple threads modifying a `HashMap` can lead to race conditions, lost updates, or data corruption.
* Java 7 `ConcurrentHashMap` used **Segment-level locking**.
* Java 8 removed segments and introduced **Bucket-level locking**.
* Read operations are mostly lock-free.
* Empty bucket insertions use **CAS**, while updates to non-empty buckets synchronize only the affected bucket.
* `ConcurrentHashMap` provides better concurrency and scalability compared to `HashMap` and `Collections.synchronizedMap()`.
