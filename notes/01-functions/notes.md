# 01 - Functions

Reading: Composing Programs 1.1, 1.2, 1.3

## Core Idea

Programs combine values and functions into expressions, bind names to results, and package repeated computations as functions.

The main theme is abstraction: give a complex process a name so it can be used as one unit.

## Expressions and Statements

An **expression** evaluates to a value.

```py
42
2 + 3
max(4, 9)
```

A **statement** performs an action.

```py
x = 3
from math import sqrt

def square(x):
    return x * x
```

Expressions can appear inside larger expressions. Statements generally cannot.

## Call Expressions

```py
max(7, 2 + 3)
```

- `max` is the operator expression.
- `7` and `2 + 3` are operand expressions.
- Evaluated operand values are arguments.
- The function produces a return value.

Evaluation rule:

1. Evaluate the operator.
2. Evaluate operands from left to right.
3. Apply the function to the argument values.

```py
from operator import add, mul

add(2, mul(3, 4))  # 14
```

Nested calls are evaluated from the inside as required by this rule.

## Names, Bindings, and Assignment

A name evaluates to its current binding in the environment.

```py
radius = 10
area = 3.14159 * radius * radius
```

Assignment evaluates the entire right side first, then binds the result.

```py
x = 2
x = x + 1  # evaluate old x + 1, then rebind x to 3
```

A name is not the value itself. Rebinding a name changes what it refers to.

```py
f = max
max = 5
f(2, 8)  # 8
```

`f` still refers to the original function value.

Avoid rebinding built-in names such as `max`, `list`, or `sum`; later calls may fail or become confusing.

## Multiple Assignment

All right-side expressions are evaluated before any left-side names are rebound.

```py
x, y = 3, 4
x, y = y, x
# x is 4; y is 3
```

This is useful for state updates that depend on old values.

```py
previous, current = current, previous + current
```

## Pure Functions and Side Effects

A pure function only returns a value and gives the same result for the same arguments.

```py
abs(-3)  # 3
```

A non-pure function also has a side effect, such as displaying output.

```py
result = print(3)
# displays 3; result is None
```

Keep returned values separate from printed output.

```py
print(print(1), print(2))
```

Output:

```text
1
2
None None
```

The inner calls print and return `None`; the outer call prints those two return values.

## Defining Functions

```py
def square(x):
    return x * x
```

- `square` is the function name.
- `x` is a formal parameter.
- `return x * x` is the body.

Executing a `def` statement:

1. creates a function value
2. records its parameters, body, and parent environment
3. binds the function name
4. does not execute the body

Calling and referring to a function are different.

```py
square     # function value
square(5)  # call; evaluates to 25
```

## Calling a User-Defined Function

For a call such as `square(5)`:

1. Evaluate the operator and operands.
2. Create a fresh local frame.
3. Bind formal parameters to argument values.
4. Execute the body in that environment.
5. Return the first executed `return` value.

```text
Global:
  square -> function square(x)

square call frame:
  x -> 5
  return value -> 25
```

Every call gets a separate frame, even when the same function is called repeatedly.

## Local Names and Scope

Formal parameters and assignments inside a function are local to that call.

```py
x = 10

def add_one(x):
    result = x + 1
    return result

add_one(3)  # 4
global x    # still 10
```

A local binding can shadow a global binding with the same name.

After a call finishes, its frame is no longer active. It normally becomes inaccessible, but it can remain reachable when a closure refers to it.

## Return Statements

`return`:

1. evaluates its expression
2. ends the current function call immediately
3. supplies the value to the caller

A function that reaches the end without executing `return` returns `None`.

```py
def broken_square(x):
    x * x

broken_square(4) is None  # True
```

An expression by itself is evaluated and discarded in a function body.

## Functional Abstraction

A good function lets callers depend on **what it does**, not **how it does it**.

```py
def square(x):
    return x * x


def sum_squares(x, y):
    return square(x) + square(y)
```

`sum_squares` depends on the behavior of `square`, not its implementation.

A functional abstraction has:

- **domain**: valid inputs
- **range**: possible outputs
- **intent**: relationship between input and output

Good functions usually have one clear job, a descriptive name, and a short contract.

## Operators

Infix operators are expression forms with precedence rules.

```py
2 + 3 * 4      # 14
(2 + 3) * 4    # 20
```

Important arithmetic operators:

```text
+   addition
-   subtraction
*   multiplication
/   true division
//  floor division
%   remainder
**  exponentiation
```

```py
5 / 4    # 1.25
5 // 4   # 1
-5 // 4  # -2: floor means round down
```

## Common Traps

- Confusing a function value with a function call.
- Forgetting `return` and receiving `None`.
- Confusing printed output with a return value.
- Rebinding a built-in function name.
- Expecting an earlier computed value to update automatically after another name changes.
- Forgetting that each call has a fresh local frame.
- Assuming assignment changes a value rather than rebinding a name.
- Ignoring left-to-right operand evaluation when calls have side effects.
