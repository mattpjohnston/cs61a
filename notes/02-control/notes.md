# 02 - Control

Reading: Composing Programs 1.4, 1.5

## Main Ideas

Control is about deciding what happens next during program execution.

Before control statements, functions can mostly combine expressions and return results. Control statements add:

- choice: execute different code depending on a condition
- repetition: execute the same code multiple times
- redirection: stop a sequence early, usually with `return`

Statements do not evaluate to values in the same way expressions do. They are executed to change what the interpreter does next.

## Designing Functions

Functions are abstractions. A good function hides details while making the important behavior easy to use.

General qualities of a good function:

- one clear job
- short, descriptive name
- behavior that can be described in one line
- no repeated logic that should be factored out
- general enough to be reused

Example of repeated logic that should become a function:

```py
area1 = 3.14159 * radius1 * radius1
area2 = 3.14159 * radius2 * radius2
area3 = 3.14159 * radius3 * radius3
```

With functional abstraction:

```py
def area(radius):
    return 3.14159 * radius * radius

area1 = area(radius1)
area2 = area(radius2)
area3 = area(radius3)
```

The repeated process is named once and reused.

## One Job Per Function

A function with multiple jobs is harder to name, test, and reuse.

Less clear:

```py
def process_score(score):
    print(score)
    return score >= 70
```

This function both prints and checks a score.

More separated:

```py
def is_passing(score):
    return score >= 70

def print_score(score):
    print(score)
```

The separate functions have clearer purposes.

## General Functions

A function should usually describe a general idea rather than a one-off special case.

Special case:

```py
def square(x):
    return x * x
```

More general operation already exists:

```py
pow(x, 2)
pow(x, n)
```

General functions reduce repeated code because they cover more cases.

## Documentation

A docstring is a string at the beginning of a function body that describes the function.

```py
def pressure(v, t, n):
    """Compute the pressure in pascals of an ideal gas.

    v -- volume of gas, in cubic meters
    t -- absolute temperature in degrees kelvin
    n -- number of particles of gas
    """
    k = 1.38e-23
    return n * k * t / v
```

Docstrings are triple-quoted and indented with the function body.

The first line gives a short description. Extra lines can describe arguments, behavior, assumptions, or examples.

`help(function_name)` displays the docstring.

Comments use `#` and are ignored by Python.

```py
k = 1.38e-23  # Boltzmann's constant
```

Comments are for readers of the source code. Docstrings are attached to the function and can be shown by help tools.

## Default Argument Values

Default argument values make some arguments optional.

```py
def pressure(v, t, n=6.022e23):
    """Compute the pressure in pascals of an ideal gas."""
    k = 1.38e-23
    return n * k * t / v
```

Calling with two arguments uses the default for `n`.

```py
pressure(1, 273.15)
```

Calling with three arguments overrides the default.

```py
pressure(1, 273.15, 3 * 6.022e23)
```

The `=` in a function header is not an assignment statement. It marks a default value for a formal parameter.

```py
def pressure(v, t, n=6.022e23):
```

The `=` inside the body is assignment.

```py
k = 1.38e-23
```

Values that callers might reasonably change can be default arguments. Values that are fixed constants can be bound inside the function or globally.

## Statements

Statements are executed, not evaluated.

Examples already introduced:

```py
x = 3
def square(x):
    return x * x
return x * x
```

Statements can contain expressions.

```py
x = 2 + 3
```

The expression `2 + 3` is evaluated, then the assignment statement binds `x` to the result.

Expression statements are also valid.

```py
mul(x, x)
```

If this appears by itself in a function body, its value is discarded.

```py
def square(x):
    mul(x, x)
```

This function does not return the square. It calls `mul`, discards the result, and returns `None`.

Correct version:

```py
def square(x):
    return mul(x, x)
```

Expression statements are useful when the expression has a side effect.

```py
def print_square(x):
    print(square(x))
```

The value returned by `print` is still `None`, but the side effect is the point of the function.

## Simple and Compound Statements

A simple statement is usually one line and does not end in a colon.

```py
x = 3
return x
print(x)
```

A compound statement contains other statements.

```py
def square(x):
    return x * x
```

Compound statements have headers and suites.

```py
<header>:
    <statement>
    <statement>
```

The header controls whether, when, or how many times the suite is executed.

Examples of headers:

```py
def square(x):
if x > 0:
while k < n:
```

The indented block after the header is the suite.

Indentation is part of Python syntax. Lines in the same suite must be indented consistently.

## Executing a Sequence of Statements

A Python program is a sequence of statements.

Execution rule:

1. Execute the first statement.
2. If control has not been redirected, execute the next statement.
3. Continue until no statements remain.

Later statements may never run if an earlier statement redirects control.

```py
def f(x):
    return x
    print(x)
```

The `print` statement is unreachable because `return` ends the function call.

## Return Statements

A `return` statement redirects control out of the current function call.

```py
def square(x):
    return x * x
```

When `return` is executed:

- its expression is evaluated
- the function call receives that value
- the function body stops executing

Example:

```py
def f(x):
    return x + 1
    return x + 2
```

`f(10)` returns `11`. The second return statement is never executed.

If a function body finishes without executing a `return` statement, the function returns `None`.

```py
def f(x):
    x + 1

f(3)   # None
```

## Local Assignment

Assignment inside a function body binds names in the local frame for that function call.

```py
def percent_difference(x, y):
    difference = abs(x - y)
    return 100 * difference / x
```

Calling the function creates a local frame.

```py
percent_difference(40, 50)
```

Local frame:

```text
x -> 40
y -> 50
difference -> 10
return value -> 25.0
```

The local assignment to `difference` does not affect the global frame.

Local assignment often makes complex expressions easier to read.

```py
def percent_difference(x, y):
    return 100 * abs(x - y) / x
```

This version has the same behavior, but the intermediate meaning is less explicit.

## Conditional Statements

Conditional statements choose which suite to execute.

General form:

```py
if <expression>:
    <suite>
elif <expression>:
    <suite>
else:
    <suite>
```

The `if` clause is required. `elif` clauses are optional. The `else` clause is optional.

Example:

```py
def absolute_value(x):
    """Compute abs(x)."""
    if x > 0:
        return x
    elif x == 0:
        return 0
    else:
        return -x
```

Execution rule:

1. Evaluate the `if` header expression.
2. If it is true, execute that suite and skip the rest.
3. Otherwise, evaluate each `elif` header expression in order.
4. Execute the first suite whose header expression is true.
5. If none are true and there is an `else`, execute the `else` suite.

Only one suite in a single `if` / `elif` / `else` chain is executed.

## Boolean Contexts

The expression in an `if`, `elif`, or `while` header is in a boolean context.

In a boolean context, Python only cares whether the value is true or false.

False values in this part of the course:

```py
False
0
None
```

Most other values are true values.

```py
if 3:
    print('true')
```

This prints because `3` is a true value.

```py
if 0:
    print('not reached')
```

This does not print because `0` is false.

## Boolean Values and Comparisons

Python has two boolean values:

```py
True
False
```

Comparison operators return boolean values.

```py
4 < 2       # False
5 >= 5      # True
0 == -0     # True
3 != 4      # True
```

Common comparisons:

```text
>   greater than
<   less than
>=  greater than or equal to
<=  less than or equal to
==  equal to
!=  not equal to
```

Assignment and equality comparison are different.

```py
x = 3     # assignment
x == 3    # equality comparison
```

## Boolean Operators

Python has three basic boolean operators:

```py
and
or
not
```

Examples:

```py
True and False    # False
True or False     # True
not False         # True
```

Boolean operators use short-circuiting. They may avoid evaluating later subexpressions when the result is already determined.

## `and`

Evaluation rule:

```py
<left> and <right>
```

1. Evaluate `<left>`.
2. If `<left>` is false, the whole expression evaluates to that false value.
3. Otherwise, evaluate and return `<right>`.

Examples:

```py
0 and 5       # 0
False and 5   # False
3 and 5       # 5
True and 5    # 5
```

`and` does not necessarily return `True` or `False`; it returns one of the operand values.

Short-circuit example:

```py
0 and print('hi')
```

`print('hi')` is not evaluated.

## `or`

Evaluation rule:

```py
<left> or <right>
```

1. Evaluate `<left>`.
2. If `<left>` is true, the whole expression evaluates to that true value.
3. Otherwise, evaluate and return `<right>`.

Examples:

```py
0 or 5        # 5
False or 5    # 5
3 or 5        # 3
True or 5     # True
```

`or` also returns one of the operand values, not necessarily a boolean.

Short-circuit example:

```py
3 or print('hi')
```

`print('hi')` is not evaluated.

## `not`

Evaluation rule:

```py
not <expression>
```

1. Evaluate the expression.
2. Return `True` if the value is false.
3. Return `False` if the value is true.

Examples:

```py
not 0       # True
not 3       # False
not None    # True
not False   # True
```

Unlike `and` and `or`, `not` always returns a boolean value.

## Predicate Functions

Functions that test a condition and return a boolean are often called predicates.

Predicate names often start with `is`.

```py
def is_even(n):
    return n % 2 == 0
```

Examples from Python:

```py
isinstance(3, int)
```

The name signals that the function answers a true/false question.

## Iteration

Iteration repeats a suite of statements.

The main iterative control statement introduced here is `while`.

```py
while <expression>:
    <suite>
```

Execution rule:

1. Evaluate the header expression.
2. If it is true, execute the suite.
3. Return to step 1.
4. If the header expression is false, skip the suite and continue after the `while` statement.

The entire suite runs before the header expression is checked again.

## While Loops

A `while` loop needs:

- a condition
- state that changes over time
- an update that eventually makes the condition false

Example:

```py
def count_up_to(n):
    k = 1
    while k <= n:
        print(k)
        k = k + 1
```

For `count_up_to(3)`, the updates are:

```text
k -> 1   print 1
k -> 2   print 2
k -> 3   print 3
k -> 4   stop
```

If the loop body never changes a relevant binding, the loop may never stop.

```py
while True:
    print('forever')
```

A loop that does not terminate is an infinite loop.

## Fibonacci Example

The Fibonacci sequence starts:

```text
0, 1, 1, 2, 3, 5, 8, 13, 21, ...
```

Each number after the first two is the sum of the previous two.

Function:

```py
def fib(n):
    """Compute the nth Fibonacci number, for n >= 2."""
    pred, curr = 0, 1
    k = 2
    while k < n:
        pred, curr = curr, pred + curr
        k = k + 1
    return curr
```

State:

- `pred`: previous Fibonacci number
- `curr`: current Fibonacci number
- `k`: which Fibonacci number `curr` represents

Trace for `fib(8)`:

```text
start: pred -> 0, curr -> 1, k -> 2

k < 8:
  pred, curr = 1, 1
  k -> 3

k < 8:
  pred, curr = 1, 2
  k -> 4

k < 8:
  pred, curr = 2, 3
  k -> 5

k < 8:
  pred, curr = 3, 5
  k -> 6

k < 8:
  pred, curr = 5, 8
  k -> 7

k < 8:
  pred, curr = 8, 13
  k -> 8

k < 8 is false
return 13
```

The simultaneous assignment is important.

```py
pred, curr = curr, pred + curr
```

Both right-side expressions are evaluated before either left-side name is rebound.

If the updates were written separately in the wrong order, the result would change.

```py
pred = curr
curr = pred + curr
```

Here the second line uses the new `pred`, not the old one.

## Testing

Testing verifies that a function's behavior matches expectations.

A test usually calls a function with specific arguments and checks the returned value.

Tests also document intended behavior.

Example:

```py
assert fib(8) == 13, 'The 8th Fibonacci number should be 13'
```

If the assertion expression is true, nothing happens.

If it is false, Python raises an error and displays the message.

## Test Functions

A test function groups related assertions.

```py
def fib_test():
    assert fib(2) == 1, 'The 2nd Fibonacci number should be 1'
    assert fib(3) == 1, 'The 3rd Fibonacci number should be 1'
    assert fib(8) == 13, 'The 8th Fibonacci number should be 13'
    assert fib(50) == 7778742049, 'Error at the 50th Fibonacci number'
```

Good tests include normal cases and edge cases.

For `fib`, edge cases include the smallest allowed inputs.

## Doctests

Doctests place example interactions inside a function docstring.

```py
def sum_naturals(n):
    """Return the sum of the first n natural numbers.

    >>> sum_naturals(10)
    55
    >>> sum_naturals(100)
    5050
    """
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total
```

The examples show calls and expected outputs.

Running doctests for a file:

```sh
python3 -m doctest <python_source_file>
```

Running doctests for one function in the interpreter:

```py
from doctest import run_docstring_examples
run_docstring_examples(sum_naturals, globals(), True)
```

The `globals()` argument gives doctest access to the current global environment.

## Key Traces

### Missing Return

```py
def square(x):
    x * x

square(4)
```

Trace:

```text
call square(4)
evaluate expression statement x * x -> 16
discard 16
function ends without return
return None
```

### Return Redirects Control

```py
def f(x):
    if x > 0:
        return x
    return -x
```

Trace for `f(3)`:

```text
x -> 3
x > 0 -> True
return x -> 3
second return is not reached
```

Trace for `f(-3)`:

```text
x -> -3
x > 0 -> False
skip if suite
return -x -> 3
```

### Conditional Chain

```py
def sign(x):
    if x > 0:
        return 1
    elif x == 0:
        return 0
    else:
        return -1
```

Trace for `sign(0)`:

```text
x > 0 -> False
x == 0 -> True
return 0
skip else
```

### Short-Circuiting

```py
def positive_and_even(x):
    return x > 0 and x % 2 == 0
```

Trace for `positive_and_even(-3)`:

```text
x > 0 -> False
and short-circuits
return False
```

Trace for `positive_and_even(4)`:

```text
x > 0 -> True
evaluate x % 2 == 0 -> True
return True
```

### While Loop

```py
def sum_naturals(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total
```

Trace for `sum_naturals(4)`:

```text
start: total -> 0, k -> 1

k <= 4:
  total, k = 1, 2

k <= 4:
  total, k = 3, 3

k <= 4:
  total, k = 6, 4

k <= 4:
  total, k = 10, 5

k <= 4 is false
return 10
```

## Common Pitfalls

Forgetting `return`:

```py
def square(x):
    x * x
```

The expression value is discarded and the function returns `None`.

Using assignment instead of equality:

```py
if x = 3:
```

This is invalid syntax. Equality comparison uses `==`.

Expecting all conditional clauses to run:

```py
if x > 0:
    return 1
elif x > -10:
    return 2
else:
    return 3
```

Only the first true clause runs.

Writing a `while` loop without a useful update:

```py
k = 1
while k <= 5:
    print(k)
```

`k` never changes, so the loop does not terminate.

Confusing boolean values with truth values:

```py
3 and 5
```

This evaluates to `5`, not `True`.

```py
0 or 5
```

This evaluates to `5`, not `True`.

Placing tests only after a large amount of code:

```py
def complicated_function(...):
    ...
```

Small functions are easier to test as soon as they are written.
