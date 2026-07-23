# Recursion Patterns

## Core Checklist

For every recursive function:

1. State the contract.
2. Identify the base case.
3. Make the input smaller.
4. Trust the recursive call.
5. Combine its answer with the current part.
6. Confirm every path reaches a base case.

## Linear Numeric Recursion

```py
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

```py
def sum_to(n):
    if n == 0:
        return 0
    return n + sum_to(n - 1)
```

Choose the identity matching the combination: `0` for addition, `1` for multiplication.

## Digit Recursion

```py
last = n % 10
rest = n // 10
```

```py
def sum_digits(n):
    if n < 10:
        return n
    return sum_digits(n // 10) + n % 10
```

```py
def count_digit(n, target):
    if n < 10:
        return 1 if n == target else 0
    return count_digit(n // 10, target) + (n % 10 == target)
```

## Sequence Recursion

```py
def recursive_sum(s):
    if not s:
        return 0
    return s[0] + recursive_sum(s[1:])
```

Use an index helper when repeated slicing matters.

## Mutual Recursion

```py
def even(n):
    if n == 0:
        return True
    return odd(n - 1)


def odd(n):
    if n == 0:
        return False
    return even(n - 1)
```

Use when rules or states alternate.

## Choice / Tree Recursion

```py
def count_ways(state):
    if complete(state):
        return 1
    if invalid(state):
        return 0
    return sum(count_ways(next_state) for next_state in choices(state))
```

Meaning:

- `1`: one completed solution
- `0`: dead end
- `+`: combine disjoint sets of solutions
- `or`: any branch succeeds
- `and`: every branch succeeds

## Tree Data Recursion

```py
def size(t):
    return 1 + sum(size(b) for b in branches(t))
```

```py
def contains(t, target):
    return label(t) == target or any(
        contains(b, target) for b in branches(t)
    )
```

```py
def map_tree(fn, t):
    return tree(fn(label(t)), [map_tree(fn, b) for b in branches(t)])
```

Each branch is a complete smaller tree.

## Recursive Generator Paths

```py
def yield_paths(t, target):
    if label(t) == target:
        yield [label(t)]
    for branch in branches(t):
        for path in yield_paths(branch, target):
            yield [label(t)] + path
```

Iterate over or `yield from` recursive generators; do not return them.

## Side-Effect Order

```py
before()
recurse(smaller)
after()
```

- `before` happens while descending.
- `after` happens while unwinding.
- Operand and branch order matters for printing, mutation, errors, and yields.

## Common Failures

- No base case or no progress.
- Wrong base identity.
- Missing `return`.
- Returning inside the first branch.
- Solving the smaller problem again instead of trusting the recursive call.
- Using `or` when all branches must run.
- Mutating shared path results.
- Assuming elegant recursion is efficient.
