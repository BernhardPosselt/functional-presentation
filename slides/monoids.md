class: center, middle

# Monoids

---

# Map-Reduce

```java
List<String> names = Lists.newArrayList("a", "test", "hi");
Integer numChars = names.stream()
    .map(String::length)
    .reduce(0, (a, b) -> a + b)
```
Parallel:
```java
Integer numChars = names.parallelStream()
    .map(String::length)
    .reduce(0, (a, b) -> a + b)
```

Distributed:
```java
Integer numChars = names.streamOnAmazon()
    .map(String::length)
    .reduce(0, (a, b) -> a + b)
```

---

# Reduce

```java
T reduce(T identity, BinaryOperator<T> accumulator)
```

```java
BinaryOperator<T> = BiFunction<T,T,T>
```


For integers:

```java
Integer reduce(Integer identity, BinaryOperator<Integer> accumulator)
```

---

# (Some) Algebraic Structures

Given an binary operation of form a • b in S

**Closure (Magma)**

S is closed under operation a • b

**Associativity (+ all of above = Semigroup)**

For all a, b and c in S, the equation (a • b) • c = a • (b • c) holds.

**Identity element (+ all of above = Monoid)**

There exists an element e in S such that for every element a in S, the equations e • a = a • e = a hold.

**Commutativity (+ all of above = Commutative Monoid)**

For all a, b in A, a • b = b • a.

---

# What the Math Stuff Really Means

Binary operation = I want to combine two values of the same type

**Closure (Magma)**

Output type is always equal to the input types (no null!)

**Associativity (Semigroup)**

It does not matter if I start from the left, middle or right side, enables parallelization through partitioning

**Identity element (+ all of above = Monoid)**

I do not need to handle empty things differently (e.g. Empty lists/optionals)

**Commutativity (+ all of above = Commutative Monoid)**

I do not need to sort my values beforehand, enables dealing with unordered values

---

# Reduce

```java
List<String> names = Lists.newArrayList("a", "test", "hi");
Integer numChars = names.stream()
    .map(String::length)
    .reduce(0, (a, b) -> a + b)
```

Reduction above: Commutative Monoid<+, Integer>

* **Associativity**: 1 + (2 + 3) = (1 + 2) + 3
* **Identity**: 0
* **Commutative**: 1 + 2 = 2 + 1

---

# Define Reductions Using Monoids

```java
class Sum {
    Integer identity() {
        return 0;
    }
    Integer sum (Integer a, Integer b) {
        return a + b;
    }
}
```

```java
List<String> names = Lists.newArrayList("a", "test", "hi");
Sum sum = new Sum();
Integer numChars = names.stream()
    .map(String::length)
    .reduce(sum.identity(), sum::sum)
```

---

# Extract Generic Monoid

```java
T reduce(Monoid<T> monoid)

stream.reduce(new Sum())
```

```java
interface Monoid<T> {
    T identity();
    T combine(T a, T b);
}
```

```java
class Sum implements Monoid<Integer> {
    Integer identity() {
        return 0;
    }
    Integer combine(Integer a, Integer b) {
        return a + b;
    }
}
```

---

# Commutative Monoid

```java
interface CommunativeMonoid<T> extends Monoid<T> {}
```

```java
class Sum implements CommunativeMonoid<Integer> {
    Integer identity() {
        return 0;
    }
    Integer combine(Integer a, Integer b) {
        return a + b;
    }
}
```

Typesafe way to use unordered streams

---

# Combining Optionals


```java
class FirstPresent<T> implements Monoid<Optional<T>> {
    Optional<T> identity() {
        return Optional.empty();
    }
    Optional<T> combine(Optional<T> left, Optional<T> right) {
        if (left.isPresent()) {
            return left;
        } else {
            return right;
        }
    }
}
```

```java
List<String> names = Lists.newArrayList(null, "a", "test", null, "hi");
Optional<String> first = names.stream()
    .map(Optional::ofNullable)
    .reduce(new FirstPresent<>())  // findFirst() for free
```

---

class: center, middle

# Questions?
