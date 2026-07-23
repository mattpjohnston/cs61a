# 05 - Recursion

Reading: Composing Programs 1.7

## Core Idea

A recursive function solves a problem by calling itself on a smaller version of the same problem.

Every correct recursive function needs:

1. A **base case** that returns without recursing.
2. A **recursive case** that makes progress toward the base case.
3. A way to use the smaller answer to solve the current problem.

```py
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

The recursive leap of faith:

> Assume the recursive call already works for the smaller input, then solve only the current step.

For `factorial(n)`, assume `factorial(n - 1)` returns `(n - 1)!`; multiplying by `n` gives `n!`.

## How Calls Execute

Each call gets its own frame and local bindings.

```text
factorial(3)
= 3 * factorial(2)
= 3 * (2 * factorial(1))
= 3 * (2 * (1 * factorial(0)))
= 3 * (2 * (1 * 1))
= 6
```

Calls first **descend** toward the base case, then **unwind** as values return.

The waiting caller is not the lexical parent of the recursive frame. For a globally defined function, every call frame has `Global` as its parent.

## Designing a Recursive Function

Ask:

1. What exactly does the function return?
2. What is the simplest input?
3. What smaller input represents the same problem?
4. What does the recursive call return?
5. How do I combine that answer with the current part?
6. Does every recursive path reach a base case?

Test the base case and one case immediately above it first.

## Recursion vs Iteration

```py
def factorial_iter(n):
    total = 1
    while n > 0:
        total, n = total * n, n - 1
    return total
```

- Iteration stores progress in changing local variables.
- Recursion stores progress in arguments and waiting call frames.
- Linear repetition is often better expressed with a loop in Python.
- Recursion is most natural for branching problems and recursive data such as trees.
- Python does not optimize tail recursion; deep recursion can raise `RecursionError`.

## Digit Recursion

For a non-negative integer:

```py
last = n % 10
rest = n // 10
```

The base case is often a single digit.

```py
def sum_digits(n):
    if n < 10:
        return n
    return sum_digits(n // 10) + n % 10
```

```py
def count_eights(n):
    if n < 10:
        return 1 if n == 8 else 0
    return count_eights(n // 10) + (1 if n % 10 == 8 else 0)
```

Choose the base value to match the question: zero has one digit when counting digits, but contributes zero when summing digits.

## Mutual Recursion

Mutually recursive functions call one another.

```py
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)


def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)
```

Use mutual recursion when the problem alternates between rules, turns, or states. Every function in the cycle still needs progress toward a base case.

## Printing and Call Order

Work before a recursive call happens while descending. Work after it happens while unwinding.

```py
def cascade(n):
    print(n)
    if n >= 10:
        cascade(n // 10)
        print(n)
```

```text
cascade(123):
123
12
1
12
123
```

Python evaluates operands left to right. This matters when calls print, mutate, raise errors, or yield values.

Keep side effects separate from return values: `print(...)` returns `None`.

## Tree Recursion

A tree-recursive function can make more than one recursive call.

```py
def fib(n):
    if n == 0:
        return 0
    if n == 1:
        return 1
    return fib(n - 2) + fib(n - 1)
```

The call structure branches like a tree. Naive Fibonacci repeats subproblems, so a clear recursive definition is not necessarily an efficient implementation.

Tree recursion often represents choices. For counting problems:

```py
def count_stairs(n):
    if n == 0:
        return 1       # one complete way
    if n < 0:
        return 0       # invalid choice
    return count_stairs(n - 1) + count_stairs(n - 2)
```

Typical meanings:

- successful complete state → `1`
- dead end → `0`
- combine disjoint possibilities → `+`
- does any branch work? → `or`
- do all branches work? → `and`

## Counting Partitions

Split partitions of `n` using parts up to `m` into:

- those using at least one `m`
- those using no `m`

```py
def count_partitions(n, m):
    if n == 0:
        return 1
    if n < 0 or m == 0:
        return 0
    return count_partitions(n - m, m) + count_partitions(n, m - 1)
```

```text
use m:     count_partitions(n - m, m)
skip m:    count_partitions(n, m - 1)
```

The include branch keeps `m` because it may be used again. Check `n == 0` first so `count_partitions(0, 0)` is `1`.

## Recursion Over Trees

When the data is a tree, recurse on each branch.

```py
def size(t):
    return 1 + sum(size(b) for b in branches(t))
```

```py
def contains(t, target):
    if label(t) == target:
        return True
    return any(contains(b, target) for b in branches(t))
```

The recursive call handles one complete branch. The current call handles the root and combines branch answers.

## Common Traps

- Missing or unreachable base case.
- Recursing on the original input or moving away from the base case.
- Forgetting to `return` the recursive result.
- Choosing a base value that does not match the combination.
- Trying to mentally solve the smaller call instead of trusting its contract.
- Returning inside the first branch and skipping the rest.
- Using `or` when every branch must be visited.
- Assuming recursion is automatically efficient.
- Confusing printed output with returned values.
- Mutating path lists shared between recursive results; construct a new path instead.
