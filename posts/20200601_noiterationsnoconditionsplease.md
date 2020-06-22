# No Iterations, and No Conditions, Please

[Original at SabalTech.com](https://www.sabaltech.com/blog)

## The Problem

While the title doesn`t do justice to Radiohead's **No Surprises**, it fits well enough
with the subject I would like to discuss.

One day, what was pretty much a homework question popped up on my Quora feed. The question
was *"Can one write Java code that prints 1 through 100 without using any loop or conditions?"*.

Straightforward question with more or less inspired responses, each of them more or less wrong.

### The Jokers

I will leave aside the snarky, and low hanging fruit jokes, such as
```java
System.out.println("1 through 100 [without using any loop or condition]");
````
, and
```java
System.out.println("1");
...
System.out.println("99");
System.out.println("100");
```

I would like to point out that the latter is one correct solution to the problem,
albeit not the one that is expected, since the point of the exercise must be to
actually write a lot of numbers without automating the work, and without intensive
manual labor.

In other words, the solution must be solvable in a decent number of steps, not 
necessarily linearly dependent on the number of values, with a fixed number of print
calls that will still print all the numbers.

Imagine fully unrolling a loop and getting a smaller chunk of code than 
you would expect, which still does the same job.

### The Misguided

So what does no loops and no conditions mean? The most straightforward answer is that 
you are not allowed to solve this specific problem with a code that looks like this:

```java
for(int i=1; i<=100; i++) {
  System.out.println(i);
}
```

If the condition above was not explicit enough, it can't look like this, either:

```java
int i=1;
while(i <= 100) {
  System.out.println(i++);
}
```

However, most of the answers tried to circumvent the iteration and condition with
various tricks:

**If it's built-in, then it's not a loop**

```java
IntStream.range(1, 101).forEach(System.out::println);
```

**If it's a recursive call then it's not a loop...**

```java
boolean print(int n) {
   System.out.println(n);
   return n == 100 ? true: print(n + 1);
}
/* called with print(1) */
```

**... especially if it throws an exception to avoid a condition**
```java
void print(int n) {
    System.out.println(n);
    int i = 1 / (n - 100); /* runtime division by zero exception */
    print(n+1);
}
/* called with print(1) */
```

Now it's time to understand what we mean by iterations, loops, 
conditions, exceptions and recursion to make sure that we're not
making any confusions.

An **iteration** is the repeated application of a procedure to the
result of a previous application of that procedure for the purpose 
of refining the result.

A **loop** is a procedure (meaning a group of instructions) that is
continually repeated. The difference here being that there are no state
assumptions, so loop doesn't need to maintain a state, or to even complete,
for that matter. However, in the case of our problem, the loop involved would
be an **iteration**

A **condition** is the representation of a state of an entity. In the case of
**conditional** constructs in programming (if-then-else) a binary-state
is required and a boolean condition is used for the decision making process.
However, the condition does not need to be represented as a boolean, it's
whatever works to represent two mutually exclusive possible states.

An **exception** is an exceptional condition in the flow of an application,
a binary state differentiating between normal execution and abnormal execution.
There is no difference between an exeption and a conditional other than
the syntactic sugar that enables them in different languages. In fact, not
all languages have an exception handling mechanism, thus relying on explicit
conditionals.

**Recursion** is an approach to solving a specific problem by resolving
one or more other instances of the same problem. Recursion is a different
way to express an iteration, while hiding certain implementation details. 
Certain compilers will actually convert a tail-recursive call into an 
explicit loop.

Most of solutions given to this problem were a variation of these solutions.
However, in each case a repetitive operation (either explicit or implicit) is
clearly involved, and a condition is also present, whether hidden 
(within an iterator), or not (ternary operator, exceptions).

### The Close Encounter

There is one solution I have seen that deserves its own category, because
the thought process is a bit different, and because it almost avoids the
pitfalls from the other solutions. It is somewhat similar to the problem
of expressing a monetary value in amounts of differently valued coins.

The problem is that the output process still includes an iterative 
refinement, as can bee seen below:

```java
public class Printout {

  int currentValue = 1;

  void print1() {
    System.out.println(currentValue++);
  }

  void print2() {
    print1();
    print1();
  }

  void print5() {
    print1();
    print2();
    print2();
  }

  void print10() {
    print5();
    print5();
  }

  void print30() {
    print10();
    print10();
    print10();
  }

  void print100() {
    print30();
    print30();
    print30();
    print10();
  }

}

/* run with: new Printout().print100(); */
```

All the methods resolve to print1(), which will be called 100 times
without iterating over each call, however the output is iterated over
as a side-effect, by incrementing `currentValue`.

While not completely satisfactory, this problem points in the right direction
by managing to remove the conditional.

### The Solution

Let's consider the following question: Is there any way of representing a
set of 100 elements into multiple equivalent sets of smaller cardinalities
such that the sum of those cardinalities is less than 100? Keep in mind that
those cardinalities multiplied would still need to be 100.

The answer is yes. If we get the prime factors of 100 we obtain: 
`100 = 2 * 2 * 5 * 5`. This means that we can have a bijective function
that takes a number between 1 and 100 and can transform it into 4 numbers
each part of its respective interval. Furthermore, the sum of the
cardinalities of these sets is 14, which tells us that we can write
4 functions, each corresponding to an interval, and each providing a partial
mapping of the bijective function, which will require us to write
14 lines of code (leaving out the method declaration) calls between all of them. 
Applying the inverse function for the
given bijection we will get the equivalent number from the 100 elements interval.

This is not the first time this problem is encountered. Think of the representation
of a number as multi-byte. The single value may not fit in a single byte so
it will be broken into multiple values in the [0, 255] interval (if byte==8 bits).
The only difference between this problem and ours is that, in our case, 
we have to deal with intervals with different cardinalities, rather than 
settling on a  fixed representation. Sure, we could use a small enough fixed value,
but the smaller the value the more functions we have to write, because 
we get more intervals. As we're going to see, later on, these intervals may
require some extra help.

One thing to note here, which we will address later, is that we can actually
cut down the number of functions from 4 to 3, because 2 * 2 = 2 + 2 = 4, so
we can use the 100 = 4 x 5 x 5 equivalence without overhead, actually quite
the opposite.

We'll consider indexes starting from 0 to simplify things and we'll add the one
at when we print the number. This will not be a side-effect recursion because
the operation at each step is not based on a previous step and it will not help
the next step, we just add it as a starting value. Thus, a quick way to solve
the problem is the code below: 4 calls x 5 calls x 5 calls = 100 calls. 
4 calls + 5 calls + 5 calls = 14 lines of code.

```java
public class Printout1To100 {

    static void println(int index1, int index2) {
        System.out.println(25 * index1 +5 * index2 + 1);
        System.out.println(25 * index1 +5 * index2 + 2);
        System.out.println(25 * index1 +5 * index2 + 3);
        System.out.println(25 * index1 +5 * index2 + 4);
        System.out.println(25 * index1 +5 * index2 + 5);
    }

    static void println(int index1) {
        println(index1, 0);
        println(index1, 1);
        println(index1, 2);
        println(index1, 3);
        println(index1, 4);
    }

    public static void main(String[] argv) {
        println(0);
        println(1);
        println(2);
        println(3);
    }

}
```

Now imagine that your homework becomes not printing 1 to 100, but 1 to 101.
We get the prime factors and we're stumped. "There can be only one", and
it's 101!

More on that in the next iteration of this document, where we go meta.

## Going Meta

### Almost Metal

As we've seen, there is a way to write consecutive numbers without loops and
conditions if we can find an equivalent way to express them that is broken
into a product with a sum representing the number of cross calls.

The problem is that, while prime factorization may break the number down
into various intervals, it does not work for prime numbers. However, not 
all is lost. We can still lower the number of calls considerably, even for
prime numbers if we can find a good enough factorization for a lower number
such that the difference between the prime and the maximum value of the 
factorization is acceptable.

For example, in our previous situation, we decided that 100 = 4 x 5 x 5, so it
could be written in 14 calls. Well, 100 is just 1 print away from 101, so
all we would need to do is print all 100 numbers and add one more entry at
the end of the `main` method which says: 
```java
System.out.println(101);
```

... and we did by writing 15 calls, instead of 101.

Then the problem becomes: "Is there a way to generate an optimial
solution to the problem using this approach?". In other words, can 
we write a program that writes our program in a decent way? Probably,
otherwise this article may just end after this paragraph.

And it didn't. As we've seen, if we can write something like 100 = 4 x 5 x 5,
then we have 3 intervals (for the sake of simplicity, so we don't have to
complicate our way around modulo operations):

```text
A1 = {0, 1, 2, 3}
A2 = {0, 1, 2, 3, 4}
A3 = {0, 1, 2, 3, 4}
```

such that `c(I = {0, 1, 2, ..., 99}) = c(A1) x c(A2) x c(A3)`. Where `c(X)`
is the cardinality of the set `X`. We're talking finite sets of natural
numbers, so the cardinality is the number of elements.

Then we can create a bijective function from `f : A1xA2xA2 -> I` with
an inverse `fi: I -> A1xA2xA3`.

To get from a1, a2, a3 into an value in I by applying f we can do:

```text
i = a1 x c(A2) x c(A3) + a2 x c(A3) + a3
```

Example:

```text
a1 = 3
a2 = 4
a3 = 3

98 = 3 x 25 + 4 x 5 + 3
```

Applying the inverse involves a multi-step approach where the number
is divided (integer division) by the successive multipliers 
(cardinals) to get each index, and the remainder (talking about the modulo)
is used to compute the next index with the remaining multipliers.

Example:

```text
a1 = 98 / (5 * 5) = 3
r1 = 23

a2 = r1 / 5 = 23 / 5 = 4
r2 = 3

no more multipliers

a3 = r2 = 3
```

Alright, let's skip ahead so we don't get lost in numbers. This is why
we're writing a program, so it can get lost in numbers, remember?

So, there are a number of steps involved in coming out with a solution
for a given strictly positive natural number `n`:

* find the primes all the way to `n`

* get the prime factorizations for every number all the way to `n`

* we've mentioned that `2 + 2 = 2 x 2 = 4` which is neat because we can write
  either 2 functions of two calls, or one function of 4 calls. We would
  like to just call print 4 times in one function so we don't explode 
  vertically. For example, 64 can be written as 2^8 which means 8 
  functions of 2 calls each. However, if we group 2s together,
  two-by-two, we get 4^3, which is 3 calls of 4. Yes 4 + 4 + 4 = 16,
  but we write fewer functions. So we'd like to optimize that.

* handle the primes but looking at all the numbers smaller than them
  to see if they have a good factorization and are close enough to
  subtract from prime, for a smaller representation.

* generate the code.

The code, as it is presented, is only meant to solve a problem, not
to teach best practices, styles, and patterns. It is just a first
iteration that worked, and it's good enough. You are welcome to make it
prettier.

Let's start with a class called `SolutionGenerator` and start adding 
the methods to it (you can leave the comments out).

```java
public class SolutionGenerator {
  // ... more to come below
```

First method will build a sieve so we can incrementally find our primes
up to a value we choose:

```java
private static List<Long> buildSieve(final long max) {

    final List<Long> result = new LinkedList<>();

    if (max >= 3L) {
      result.add(3L);
    }
    if (max >= 5L) {
      result.add(5L);
    }

    if (max > 5L) {
      long divMax = 3L;
      long divPow = divMax * divMax;
      boolean byTwo = true;
      long current = 5L;

      while (true) {
        final long next = byTwo ? current + 2 : current + 4;
        byTwo = !byTwo;

        if (next <= max) {

          if (next > divPow) {
            divMax++;
            divPow = divMax * divMax;
          }

          final long itMax = divMax;

          if (!result.stream()
              .takeWhile(l -> l <= itMax)
              .anyMatch(l -> next % l == 0)) {
            result.add(next);
          }

          current = next;

        } else {
          break;
        }
      }
    }

    if (max >= 2L) {
      result.add(0, 2L);
    }

    return result;

  }
```

You'll see that I left 2 out and added it at the end (if smaller than our 
number). That's because we plan to only iterate over the odd numbers, 
and 2 is the only even prime.

You'll also see that I am adding 3, and 5 at the beginning if each of them
is also smaller than our number. That's because I plan to iterate over the
odd numbers in a 2 vs. 4 increments. Here's the thing, outside of 2 and 3,
all the prime numbers candidates can be written as either `6n+1`, or `6n+5`.
Think about it, they are the only odd numbers not divisible by 3, and we
don't care about even numbers.

And, by alternating the increment based on the previous value, we can skip
the 3s. Example: 3, 5 (+2), 7 (+4), 11 (+2), 13 (+4), 17 (+2), 19 and so on.
You will get to non-primes, don't get too excited. Just go two more and
you'll hit 25.

Next, we'll add a helper function which will find the factorization
of any number smaller than our maximum number:

```java
static List<Long> factorization(
      final long number,
      final List<Long> sieve) {
    if (number == 1) {
      return Collections.singletonList(1L);
    }
    List<Long> result = new LinkedList<>();
    long current = number;
    for (long sieveValue : sieve) {
      while (current % sieveValue == 0) {
        result.add(sieveValue);
        current = current / sieveValue;
      }
      if (sieveValue > current) {
        break;
      }
    }
    if (current != 1) {
      result.add(current);
    }
    return result;
  }
```

It will return 1 if the number is 1. Otherwise, it will try to find divisors
in the sieve. If one divisor is found, it is tried again. For example, 
12 = 2 x 2 x 3, so 2 we'll be retried once more, after it is found the
first time, and another time after the second one, and that will fail, since
3/2 doesn't work out.

Let's now introduce the function that will reduce the 2s in a factorization,
so all pair of 2s are converted to 4s:

```java
static List<Long> reduceTwos(
      final List<Long> factorization) {
    final Map<Long, Long> factorCount = factorization.stream()
        .collect(Collectors.toMap(
            Function.identity(),
            v -> 1L,
            (a, b) -> a + b
        ));
    final long twos = factorCount.getOrDefault(2L, 0L)
        + 2 * factorCount.getOrDefault(4L, 0L);
    final long twoCount = twos % 2;
    final long fourCount = twos / 2;
    factorCount.compute(2L, (k, v) -> twoCount == 0 ? null : twoCount);
    factorCount.compute(4L, (k, v) -> fourCount == 0 ? null : fourCount);
    return new TreeMap<>(factorCount).entrySet().stream()
        .flatMapToLong(e ->
            LongStream.range(0, e.getValue()).map(anyValue -> e.getKey()))
        .mapToObj(Long::valueOf).collect(Collectors.toList());

  }

```

We just did a quick and dirty counting of the prime factors, changed the 2s
into fours and updated the count for both 2s and 4s (if applicable, of
course), then we re-create the factorization in a sorted way.

And now, for a locally optimized version of the factorizations:

```java
static Map<Long, List<Long>> locallyReducedFactorizations(long max) {
    final List<Long> sieve = SolutionGenerator.buildSieve(max);
    return LongStream.rangeClosed(1, max)
        .mapToObj(Long::valueOf)
        .collect(Collectors.toMap(
            Function.identity(),
            v -> reduceTwos(factorization(v, sieve))
        ));
  }
```

Given our maximum value, this method goes through the motions necessary
to get the factorizations as neat as we wanted them.

At this point we will add a `NEWLINE` that we'll use for formatting stuff:

```java
private static final String NEWLINE = String.format("%n");
```

It just adds a new line in a platform dependent way. Who knows, 
you might be on a hand calculator.

Ok, for the next part we need a structure that will allow us to expand
a bit on the information that we can extract from the prime factorization.

Do not close the scope of the previous class. This one is internal.
Not on purpose, just laziness.

```java
static class SolutionInfo {
    public long number;
    public List<Long> factorization;
    public long extraAddition;
    public long operationCount;

    public SolutionInfo(
        final long numberValue,
        final List<Long> factorizationValue,
        final long extrAdditionValue,
        final long operationCount
    ) {
      this.number = numberValue;
      this.factorization = factorizationValue;
      this.extraAddition = extrAdditionValue;
      this.operationCount = operationCount;
    }

    /**
     * -1 - if this is a worse solution.
     * 0 - if this is no worse than other.
     * 1 - if this is a better solution.
     */
    public int compareAgainstOtherWithBaseline(final SolutionInfo baseLine,
                                               final SolutionInfo other) {
      final long valueForThis = baseLine.number - number + operationCount;
      final long valueForOther =
          baseLine.number - other.number + other.operationCount;
      if (valueForThis >= valueForOther) {
        return valueForThis == valueForOther ? 0 : -1;
      } else {
        return 1;
      }
    }

    @Override
    public String toString() {
      return String.format("{number: %d, factorization:[%s], " +
          "extraAddition: %d, operations: %d}",
          number,
          factorization.stream().map(String::valueOf)
              .collect(Collectors.joining(",")),
          extraAddition,
          operationCount);
    }
  }

  public static Map<Long, SolutionInfo> localSolutions(long max) {
    return locallyReducedFactorizations(max)
        .entrySet()
        .stream()
        .map(e -> new SolutionInfo(
            e.getKey(),
            e.getValue(),
            0L,
            e.getValue().stream().mapToLong(Long::longValue).sum()))
        .collect(Collectors.toMap(v -> v.number, Function.identity()));
  }
```

We have a class that can hold one of our numbers, its factorization (with
the 2s reduction), an extra addition field that we can use to see
how many extra values we need to print to get to another number,
and the operation count mathing the factorization and the extra addition.

The `compareAgainstOtherWithBaseline` will compare this solution versus
another solution to see which one optimizes the `baseLine` solution the
best. If this uses fewer operations then 1 is returned, if they're even
then 0 is returned, and if this uses more operations, then -1 is returned.

We will use this next method to get us all the local solutions up to
our maximum number:

```java
public static Map<Long, SolutionInfo> localSolutions(long max) {
    return locallyReducedFactorizations(max)
        .entrySet()
        .stream()
        .map(e -> new SolutionInfo(
            e.getKey(),
            e.getValue(),
            0L,
            e.getValue().stream().mapToLong(Long::longValue).sum()))
        .collect(Collectors.toMap(v -> v.number, Function.identity()));
  }
```

The following method will optimize a value from our set taking the
local solutions into consideration:

```java
static SolutionInfo optimizeValue(
      final long value,
      final Map<Long, SolutionInfo> locallyOptimized) {
    final SolutionInfo valueSolutionInfo = locallyOptimized.get(value);
    return locallyOptimized.entrySet().stream()
        .map(e -> e.getValue())
        .filter(si -> si.number < value)
        .max((si1, si2) -> si1.compareAgainstOtherWithBaseline(
            valueSolutionInfo, si2))
        .filter(si -> si.compareAgainstOtherWithBaseline(
            valueSolutionInfo, valueSolutionInfo) > 0)
        .map(si -> new SolutionInfo(
            valueSolutionInfo.number,
            si.factorization,
            value - si.number,
            si.operationCount + value - si.number))
        .orElse(valueSolutionInfo);
  }
```

And this one ties everything together, returning the best optimizations
it can find, for every number between 1 and our maximum number:

```java
public static Map<Long, SolutionInfo> optimizedSolutions(long max) {
    final Map<Long, SolutionInfo> localSolutions = localSolutions(max);
    return localSolutions.entrySet().stream()
        .map(e -> optimizeValue(e.getKey(), localSolutions))
        .collect(Collectors.toMap(si -> si.number, Function.identity()));
  }
```

The following two methods generate the solution code for a given 
`SolutionInfo` (from our set), with a specific starting value. Remember,
we are 0 based, so printing 1 to 100 will print 0 to 99, by default. 
The startingValue is added to any output, and it can be whatever we want.
So we can actually print consecutive numbers starting from a value higher 
than one.

```java
private static String getCall(
      final int methodIndex,
      final long factor,
      final long multiplier,
      final String nextMethodName) {
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder
        .append(String.format("private static void print%d(long seed) {", methodIndex))
        .append(NEWLINE);

    for (long domainValue = 0; domainValue < factor; domainValue++) {
      stringBuilder
          .append(
              String.format("%s(seed + %d);",
                  nextMethodName, domainValue * multiplier))
          .append(NEWLINE);
    }

    return stringBuilder.append("}").append(NEWLINE).toString();
  }

  public static String generateSolution(
      final SolutionInfo solutionInfo,
      final long startingValue) {

    StringBuilder stringBuilder = new StringBuilder()
        .append("class Solution").append(solutionInfo.number)
        .append(" {").append(NEWLINE);

    stringBuilder.append(
        String.format("private static void printOut(long value) " +
            "{System.out.println(value + %d);}", startingValue))
        .append(NEWLINE);

    final int factorizationSize = solutionInfo.factorization.size();

    LinkedList<Long> multipliersList = new LinkedList<>(
        IntStream.range(0, factorizationSize)
            .mapToObj(index ->
                solutionInfo.factorization.subList(index, factorizationSize)
                    .stream().reduce(1L, (a, b) -> a * b))
            .collect(Collectors.toList()));
    multipliersList.add(1L);
    final long maxNumber = multipliersList.pollFirst();

    int methodIndex = 0;
    Iterator<Long> multiplierIterator = multipliersList.iterator();
    Iterator<Long> factorsIterator = solutionInfo.factorization.iterator();

    while (true) {
      long multiplier = multiplierIterator.next();
      long factor = factorsIterator.next();
      if (multiplierIterator.hasNext()) {
        final String nextMethodName =
            String.format("print%d", methodIndex + 1);
        stringBuilder.append(
            getCall(methodIndex, factor, multiplier, nextMethodName));
      } else {
        final String nextMethodName = "printOut";
        stringBuilder.append(
            getCall(methodIndex, factor, multiplier, nextMethodName));
        break;
      }
      methodIndex++;
    }

    stringBuilder
        .append("public static void main(String[] argv) {")
        .append(NEWLINE)
        .append("print0(0);")
        .append(NEWLINE);

    for(long remaining = maxNumber;
        remaining < solutionInfo.number;
        remaining++) {
      stringBuilder.append(String.format("printOut(%d);", remaining))
          .append(NEWLINE);
    }

    stringBuilder.append("}").append(NEWLINE);
    return stringBuilder.append("}").append(NEWLINE).toString();
  }
```

You probably saw, by now, that that `NEWLINE` constant is used a lot
in our code builder.

The code generator will call print0 with a seed of 0. Then it will print
the remaining operations which are not part of the factorization (e.g.
printing 101 separately from the factorization of 100).

`print0` is the first factorization interval. The `seed` value is
available in every print method, and represents the cardinality multiplication
of the previous value to be added to our partial computation of the 
inverse function.

If we hit the last interval, then the calls of the last `print` method
are made to the printOut method to do the actual printing. That's where
we also add the offset.

And now we can finally add:

```java
}
```

Ok, let's now consider adding a main method to generate code (you
should probably do this before the closing curly brace above).

```java
public static void main(String[] argv) {
  final long max = 101L;
  final long startingValue = 1L;
  optimizedSolutions(max).entrySet().stream()
        .map(Map.Entry::getValue)
        .map(si -> generateSolution(si, startingValue))
        .forEach(System.out::println);
```

This will generate solutions (a class for each) for all the numbers
between 1 and 101. All the solutions will start printing from the 
`startingValue` (which is 1).

Let's see how the code looks for our 100, and 101 cases from the
beginning:

```java
class Solution100 {
private static void printOut(long value) {System.out.println(value + 5);}
private static void print0(long seed) {
print1(seed + 0);
print1(seed + 25);
print1(seed + 50);
print1(seed + 75);
}
private static void print1(long seed) {
print2(seed + 0);
print2(seed + 5);
print2(seed + 10);
print2(seed + 15);
print2(seed + 20);
}
private static void print2(long seed) {
printOut(seed + 0);
printOut(seed + 1);
printOut(seed + 2);
printOut(seed + 3);
printOut(seed + 4);
}
public static void main(String[] argv) {
print0(0);
}
}
```

14? 14!

```java
class Solution101 {
private static void printOut(long value) {System.out.println(value + 5);}
private static void print0(long seed) {
print1(seed + 0);
print1(seed + 25);
print1(seed + 50);
print1(seed + 75);
}
private static void print1(long seed) {
print2(seed + 0);
print2(seed + 5);
print2(seed + 10);
print2(seed + 15);
print2(seed + 20);
}
private static void print2(long seed) {
printOut(seed + 0);
printOut(seed + 1);
printOut(seed + 2);
printOut(seed + 3);
printOut(seed + 4);
}
public static void main(String[] argv) {
print0(0);
printOut(100);
}
}

```

15? 15! I hope you don't act surprised, you already knew that. Or did you?

Apparently, not everything needs to work this way. Some factorizations are
better than others, and if the numbers are close enough you should choose
the one that has a smaller representation, not the one that has an exact
factorization.

Here's an example:

```text
36 = 2 x 2 x 3 x 3 = 4 x 3 x 3 (10 operations)
38 = 2 x 19 (21 operations) or 36 and 2 (12 operations).
```

Clearly, we would prefer to write 12 operations instead of 21.

To make it clearer, you can find the locally optimized vs. the 
optimized result operations count for the same numbers below.

I hope you enjoyed this. If you got here and you realized it was a waste
of time keep in mind that I did not force you to read it.

### Locally Optimized

```text
{number: 1, factorization:[1], extraAddition: 0, operations: 1}
{number: 2, factorization:[2], extraAddition: 0, operations: 2}
{number: 3, factorization:[3], extraAddition: 0, operations: 3}
{number: 4, factorization:[4], extraAddition: 0, operations: 4}
{number: 5, factorization:[5], extraAddition: 0, operations: 5}
{number: 6, factorization:[2,3], extraAddition: 0, operations: 5}
{number: 7, factorization:[7], extraAddition: 0, operations: 7}
{number: 8, factorization:[2,4], extraAddition: 0, operations: 6}
{number: 9, factorization:[3,3], extraAddition: 0, operations: 6}
{number: 10, factorization:[2,5], extraAddition: 0, operations: 7}
{number: 11, factorization:[11], extraAddition: 0, operations: 11}
{number: 12, factorization:[3,4], extraAddition: 0, operations: 7}
{number: 13, factorization:[13], extraAddition: 0, operations: 13}
{number: 14, factorization:[2,7], extraAddition: 0, operations: 9}
{number: 15, factorization:[3,5], extraAddition: 0, operations: 8}
{number: 16, factorization:[4,4], extraAddition: 0, operations: 8}
{number: 17, factorization:[17], extraAddition: 0, operations: 17}
{number: 18, factorization:[2,3,3], extraAddition: 0, operations: 8}
{number: 19, factorization:[19], extraAddition: 0, operations: 19}
{number: 20, factorization:[4,5], extraAddition: 0, operations: 9}
{number: 21, factorization:[3,7], extraAddition: 0, operations: 10}
{number: 22, factorization:[2,11], extraAddition: 0, operations: 13}
{number: 23, factorization:[23], extraAddition: 0, operations: 23}
{number: 24, factorization:[2,3,4], extraAddition: 0, operations: 9}
{number: 25, factorization:[5,5], extraAddition: 0, operations: 10}
{number: 26, factorization:[2,13], extraAddition: 0, operations: 15}
{number: 27, factorization:[3,3,3], extraAddition: 0, operations: 9}
{number: 28, factorization:[4,7], extraAddition: 0, operations: 11}
{number: 29, factorization:[29], extraAddition: 0, operations: 29}
{number: 30, factorization:[2,3,5], extraAddition: 0, operations: 10}
{number: 31, factorization:[31], extraAddition: 0, operations: 31}
{number: 32, factorization:[2,4,4], extraAddition: 0, operations: 10}
{number: 33, factorization:[3,11], extraAddition: 0, operations: 14}
{number: 34, factorization:[2,17], extraAddition: 0, operations: 19}
{number: 35, factorization:[5,7], extraAddition: 0, operations: 12}
{number: 36, factorization:[3,3,4], extraAddition: 0, operations: 10}
{number: 37, factorization:[37], extraAddition: 0, operations: 37}
{number: 38, factorization:[2,19], extraAddition: 0, operations: 21}
{number: 39, factorization:[3,13], extraAddition: 0, operations: 16}
{number: 40, factorization:[2,4,5], extraAddition: 0, operations: 11}
{number: 41, factorization:[41], extraAddition: 0, operations: 41}
{number: 42, factorization:[2,3,7], extraAddition: 0, operations: 12}
{number: 43, factorization:[43], extraAddition: 0, operations: 43}
{number: 44, factorization:[4,11], extraAddition: 0, operations: 15}
{number: 45, factorization:[3,3,5], extraAddition: 0, operations: 11}
{number: 46, factorization:[2,23], extraAddition: 0, operations: 25}
{number: 47, factorization:[47], extraAddition: 0, operations: 47}
{number: 48, factorization:[3,4,4], extraAddition: 0, operations: 11}
{number: 49, factorization:[7,7], extraAddition: 0, operations: 14}
{number: 50, factorization:[2,5,5], extraAddition: 0, operations: 12}
{number: 51, factorization:[3,17], extraAddition: 0, operations: 20}
{number: 52, factorization:[4,13], extraAddition: 0, operations: 17}
{number: 53, factorization:[53], extraAddition: 0, operations: 53}
{number: 54, factorization:[2,3,3,3], extraAddition: 0, operations: 11}
{number: 55, factorization:[5,11], extraAddition: 0, operations: 16}
{number: 56, factorization:[2,4,7], extraAddition: 0, operations: 13}
{number: 57, factorization:[3,19], extraAddition: 0, operations: 22}
{number: 58, factorization:[2,29], extraAddition: 0, operations: 31}
{number: 59, factorization:[59], extraAddition: 0, operations: 59}
{number: 60, factorization:[3,4,5], extraAddition: 0, operations: 12}
{number: 61, factorization:[61], extraAddition: 0, operations: 61}
{number: 62, factorization:[2,31], extraAddition: 0, operations: 33}
{number: 63, factorization:[3,3,7], extraAddition: 0, operations: 13}
{number: 64, factorization:[4,4,4], extraAddition: 0, operations: 12}
{number: 65, factorization:[5,13], extraAddition: 0, operations: 18}
{number: 66, factorization:[2,3,11], extraAddition: 0, operations: 16}
{number: 67, factorization:[67], extraAddition: 0, operations: 67}
{number: 68, factorization:[4,17], extraAddition: 0, operations: 21}
{number: 69, factorization:[3,23], extraAddition: 0, operations: 26}
{number: 70, factorization:[2,5,7], extraAddition: 0, operations: 14}
{number: 71, factorization:[71], extraAddition: 0, operations: 71}
{number: 72, factorization:[2,3,3,4], extraAddition: 0, operations: 12}
{number: 73, factorization:[73], extraAddition: 0, operations: 73}
{number: 74, factorization:[2,37], extraAddition: 0, operations: 39}
{number: 75, factorization:[3,5,5], extraAddition: 0, operations: 13}
{number: 76, factorization:[4,19], extraAddition: 0, operations: 23}
{number: 77, factorization:[7,11], extraAddition: 0, operations: 18}
{number: 78, factorization:[2,3,13], extraAddition: 0, operations: 18}
{number: 79, factorization:[79], extraAddition: 0, operations: 79}
{number: 80, factorization:[4,4,5], extraAddition: 0, operations: 13}
{number: 81, factorization:[3,3,3,3], extraAddition: 0, operations: 12}
{number: 82, factorization:[2,41], extraAddition: 0, operations: 43}
{number: 83, factorization:[83], extraAddition: 0, operations: 83}
{number: 84, factorization:[3,4,7], extraAddition: 0, operations: 14}
{number: 85, factorization:[5,17], extraAddition: 0, operations: 22}
{number: 86, factorization:[2,43], extraAddition: 0, operations: 45}
{number: 87, factorization:[3,29], extraAddition: 0, operations: 32}
{number: 88, factorization:[2,4,11], extraAddition: 0, operations: 17}
{number: 89, factorization:[89], extraAddition: 0, operations: 89}
{number: 90, factorization:[2,3,3,5], extraAddition: 0, operations: 13}
{number: 91, factorization:[7,13], extraAddition: 0, operations: 20}
{number: 92, factorization:[4,23], extraAddition: 0, operations: 27}
{number: 93, factorization:[3,31], extraAddition: 0, operations: 34}
{number: 94, factorization:[2,47], extraAddition: 0, operations: 49}
{number: 95, factorization:[5,19], extraAddition: 0, operations: 24}
{number: 96, factorization:[2,3,4,4], extraAddition: 0, operations: 13}
{number: 97, factorization:[97], extraAddition: 0, operations: 97}
{number: 98, factorization:[2,7,7], extraAddition: 0, operations: 16}
{number: 99, factorization:[3,3,11], extraAddition: 0, operations: 17}
{number: 100, factorization:[4,5,5], extraAddition: 0, operations: 14}
{number: 101, factorization:[101], extraAddition: 0, operations: 101}

```

### Optimized Solutions

```text
{number: 1, factorization:[1], extraAddition: 0, operations: 1}
{number: 2, factorization:[2], extraAddition: 0, operations: 2}
{number: 3, factorization:[3], extraAddition: 0, operations: 3}
{number: 4, factorization:[4], extraAddition: 0, operations: 4}
{number: 5, factorization:[5], extraAddition: 0, operations: 5}
{number: 6, factorization:[2,3], extraAddition: 0, operations: 5}
{number: 7, factorization:[2,3], extraAddition: 1, operations: 6}
{number: 8, factorization:[2,4], extraAddition: 0, operations: 6}
{number: 9, factorization:[3,3], extraAddition: 0, operations: 6}
{number: 10, factorization:[2,5], extraAddition: 0, operations: 7}
{number: 11, factorization:[3,3], extraAddition: 2, operations: 8}
{number: 12, factorization:[3,4], extraAddition: 0, operations: 7}
{number: 13, factorization:[3,4], extraAddition: 1, operations: 8}
{number: 14, factorization:[2,7], extraAddition: 0, operations: 9}
{number: 15, factorization:[3,5], extraAddition: 0, operations: 8}
{number: 16, factorization:[4,4], extraAddition: 0, operations: 8}
{number: 17, factorization:[4,4], extraAddition: 1, operations: 9}
{number: 18, factorization:[2,3,3], extraAddition: 0, operations: 8}
{number: 19, factorization:[2,3,3], extraAddition: 1, operations: 9}
{number: 20, factorization:[4,5], extraAddition: 0, operations: 9}
{number: 21, factorization:[3,7], extraAddition: 0, operations: 10}
{number: 22, factorization:[4,5], extraAddition: 2, operations: 11}
{number: 23, factorization:[4,5], extraAddition: 3, operations: 12}
{number: 24, factorization:[2,3,4], extraAddition: 0, operations: 9}
{number: 25, factorization:[5,5], extraAddition: 0, operations: 10}
{number: 26, factorization:[2,3,4], extraAddition: 2, operations: 11}
{number: 27, factorization:[3,3,3], extraAddition: 0, operations: 9}
{number: 28, factorization:[3,3,3], extraAddition: 1, operations: 10}
{number: 29, factorization:[3,3,3], extraAddition: 2, operations: 11}
{number: 30, factorization:[2,3,5], extraAddition: 0, operations: 10}
{number: 31, factorization:[2,3,5], extraAddition: 1, operations: 11}
{number: 32, factorization:[2,4,4], extraAddition: 0, operations: 10}
{number: 33, factorization:[2,4,4], extraAddition: 1, operations: 11}
{number: 34, factorization:[2,4,4], extraAddition: 2, operations: 12}
{number: 35, factorization:[5,7], extraAddition: 0, operations: 12}
{number: 36, factorization:[3,3,4], extraAddition: 0, operations: 10}
{number: 37, factorization:[3,3,4], extraAddition: 1, operations: 11}
{number: 38, factorization:[3,3,4], extraAddition: 2, operations: 12}
{number: 39, factorization:[3,3,4], extraAddition: 3, operations: 13}
{number: 40, factorization:[2,4,5], extraAddition: 0, operations: 11}
{number: 41, factorization:[2,4,5], extraAddition: 1, operations: 12}
{number: 42, factorization:[2,3,7], extraAddition: 0, operations: 12}
{number: 43, factorization:[2,3,7], extraAddition: 1, operations: 13}
{number: 44, factorization:[2,3,7], extraAddition: 2, operations: 14}
{number: 45, factorization:[3,3,5], extraAddition: 0, operations: 11}
{number: 46, factorization:[3,3,5], extraAddition: 1, operations: 12}
{number: 47, factorization:[3,3,5], extraAddition: 2, operations: 13}
{number: 48, factorization:[3,4,4], extraAddition: 0, operations: 11}
{number: 49, factorization:[3,4,4], extraAddition: 1, operations: 12}
{number: 50, factorization:[2,5,5], extraAddition: 0, operations: 12}
{number: 51, factorization:[2,5,5], extraAddition: 1, operations: 13}
{number: 52, factorization:[2,5,5], extraAddition: 2, operations: 14}
{number: 53, factorization:[2,5,5], extraAddition: 3, operations: 15}
{number: 54, factorization:[2,3,3,3], extraAddition: 0, operations: 11}
{number: 55, factorization:[2,3,3,3], extraAddition: 1, operations: 12}
{number: 56, factorization:[2,4,7], extraAddition: 0, operations: 13}
{number: 57, factorization:[2,3,3,3], extraAddition: 3, operations: 14}
{number: 58, factorization:[2,3,3,3], extraAddition: 4, operations: 15}
{number: 59, factorization:[2,3,3,3], extraAddition: 5, operations: 16}
{number: 60, factorization:[3,4,5], extraAddition: 0, operations: 12}
{number: 61, factorization:[3,4,5], extraAddition: 1, operations: 13}
{number: 62, factorization:[3,4,5], extraAddition: 2, operations: 14}
{number: 63, factorization:[3,3,7], extraAddition: 0, operations: 13}
{number: 64, factorization:[4,4,4], extraAddition: 0, operations: 12}
{number: 65, factorization:[4,4,4], extraAddition: 1, operations: 13}
{number: 66, factorization:[4,4,4], extraAddition: 2, operations: 14}
{number: 67, factorization:[4,4,4], extraAddition: 3, operations: 15}
{number: 68, factorization:[4,4,4], extraAddition: 4, operations: 16}
{number: 69, factorization:[4,4,4], extraAddition: 5, operations: 17}
{number: 70, factorization:[2,5,7], extraAddition: 0, operations: 14}
{number: 71, factorization:[2,5,7], extraAddition: 1, operations: 15}
{number: 72, factorization:[2,3,3,4], extraAddition: 0, operations: 12}
{number: 73, factorization:[2,3,3,4], extraAddition: 1, operations: 13}
{number: 74, factorization:[2,3,3,4], extraAddition: 2, operations: 14}
{number: 75, factorization:[3,5,5], extraAddition: 0, operations: 13}
{number: 76, factorization:[3,5,5], extraAddition: 1, operations: 14}
{number: 77, factorization:[3,5,5], extraAddition: 2, operations: 15}
{number: 78, factorization:[3,5,5], extraAddition: 3, operations: 16}
{number: 79, factorization:[3,5,5], extraAddition: 4, operations: 17}
{number: 80, factorization:[4,4,5], extraAddition: 0, operations: 13}
{number: 81, factorization:[3,3,3,3], extraAddition: 0, operations: 12}
{number: 82, factorization:[3,3,3,3], extraAddition: 1, operations: 13}
{number: 83, factorization:[3,3,3,3], extraAddition: 2, operations: 14}
{number: 84, factorization:[3,4,7], extraAddition: 0, operations: 14}
{number: 85, factorization:[3,4,7], extraAddition: 1, operations: 15}
{number: 86, factorization:[3,4,7], extraAddition: 2, operations: 16}
{number: 87, factorization:[3,4,7], extraAddition: 3, operations: 17}
{number: 88, factorization:[2,4,11], extraAddition: 0, operations: 17}
{number: 89, factorization:[2,4,11], extraAddition: 1, operations: 18}
{number: 90, factorization:[2,3,3,5], extraAddition: 0, operations: 13}
{number: 91, factorization:[2,3,3,5], extraAddition: 1, operations: 14}
{number: 92, factorization:[2,3,3,5], extraAddition: 2, operations: 15}
{number: 93, factorization:[2,3,3,5], extraAddition: 3, operations: 16}
{number: 94, factorization:[2,3,3,5], extraAddition: 4, operations: 17}
{number: 95, factorization:[2,3,3,5], extraAddition: 5, operations: 18}
{number: 96, factorization:[2,3,4,4], extraAddition: 0, operations: 13}
{number: 97, factorization:[2,3,4,4], extraAddition: 1, operations: 14}
{number: 98, factorization:[2,3,4,4], extraAddition: 2, operations: 15}
{number: 99, factorization:[2,3,4,4], extraAddition: 3, operations: 16}
{number: 100, factorization:[4,5,5], extraAddition: 0, operations: 14}
{number: 101, factorization:[4,5,5], extraAddition: 1, operations: 15}
```

