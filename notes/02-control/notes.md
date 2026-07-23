# 02 - Control

Reading: Composing Programs 1.4, 1.5

## Core Idea

Control statements decide what executes next.

They provide:

- choice with conditionals
- repetition with loops
- redirection with `return`

## Designing Functions

A good function should:

- do one clear job
- have a precise contract
- use descriptive names
- avoid duplicated logic
- be general enough to reuse
- separate computation from unrelated side effects

```py
def is_passing(score):
    """Return whether score is at least 70."""
    return score >= 70
```

A docstring states the function's behavior. Comments explain implementation details that are not obvious from the code.

## Default Arguments

Default values make trailing arguments optional.

```py
def pressure(volume, temperature, particles=6.022e23):
    k = 1.38e-23
    return particles * k * temperature / volume
```

```py
pressure(1, 273.15)           # uses default particles
pressure(1, 273.15, 1e20)     # overrides it
```

Default expressions are evaluated when the `def` statement executes, not on every call.

## Statements and Suites

A compound statement has a header ending in `:` and an indented suite.

```py
if x > 0:
    print('positive')
```

Indentation is part of Python syntax.

Statements normally execute in order until:

- no statements remain
- a conditional skips a suite
- a loop repeats or finishes
- `return` ends the current function call

## Return Redirects Control

```py
def absolute_value(x):
    if x < 0:
        return -x
    return x
```

Once a `return` executes, later statements in that call are not reached.

```py
def f(x):
    return x
    print('never runs')
```

## Conditional Statements

```py
if condition_1:
    suite_1
elif condition_2:
    suite_2
else:
    suite_3
```

Conditions are checked from top to bottom. Only the first true clause runs. `else` runs only if no earlier condition is true.

Order conditions from specific to general when they overlap.

```py
def sign(x):
    if x > 0:
        return 1
    elif x == 0:
        return 0
    else:
        return -1
```

## Truth Values

Common false values:

```py
False
None
0
0.0
''
[]
()
{}
```

Most other values are true in a boolean context.

Comparison operators return booleans.

```text
==  equal
!=  not equal
<   less than
<=  less than or equal
>   greater than
>=  greater than or equal
```

Do not confuse assignment `=` with equality comparison `==`.

## Boolean Operators

Boolean operators short-circuit: later expressions may not be evaluated.

### `and`

```py
left and right
```

- If `left` is false, return it without evaluating `right`.
- Otherwise evaluate and return `right`.

```py
0 and print('no')  # 0; nothing printed
3 and 5            # 5
```

### `or`

```py
left or right
```

- If `left` is true, return it without evaluating `right`.
- Otherwise evaluate and return `right`.

```py
3 or print('no')  # 3; nothing printed
0 or 5            # 5
```

### `not`

`not value` always returns `True` or `False`.

`and` and `or` return operand values, not necessarily booleans.

Short-circuiting is useful for guarding unsafe expressions.

```py
x != 0 and 10 / x > 2
```

## Predicate Functions

A predicate answers a yes/no question.

```py
def is_even(n):
    return n % 2 == 0
```

Predicate names should make their boolean intent obvious.

## While Loops

```py
while condition:
    suite
```

Execution:

1. Evaluate the condition.
2. If false, finish the loop.
3. If true, execute the suite.
4. Return to step 1.

A useful loop needs:

- state that represents current progress
- a condition based on that state
- an update that eventually makes the condition false

```py
def sum_naturals(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total
```

Loop invariant:

> Before each iteration, `total` is the sum of the integers already processed.

An invariant explains why the loop is correct.

## Iterative Fibonacci

```py
def fib(n):
    previous, current = 0, 1
    k = 0
    while k < n:
        previous, current = current, previous + current
        k += 1
    return previous
```

Simultaneous assignment is important: both right sides use the old bindings.

## Testing

Tests check behavior and document expectations.

```py
assert sum_naturals(4) == 10
```

Good tests include:

- the smallest valid input
- ordinary inputs
- boundaries between conditional cases
- cases likely to expose mistakes

Doctests place examples in a docstring.

```py
def square(x):
    """Return x squared.

    >>> square(4)
    16
    """
    return x * x
```

Run file doctests with:

```sh
python3 -m doctest file.py
```

## Common Traps

- Forgetting `return` and getting `None`.
- Placing code after an unconditional `return`.
- Using `=` instead of `==` in a condition.
- Expecting more than one clause in an `if`/`elif`/`else` chain to run.
- Forgetting that `and` and `or` return operand values.
- Relying on a short-circuited expression to run.
- Writing a loop whose relevant state never changes.
- Updating loop variables in the wrong order.
- Testing only ordinary cases and missing boundaries.
- Mixing computation and printing when a pure return value is expected.
