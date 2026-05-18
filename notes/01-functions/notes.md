# 01 - Functions

Reading: Composing Programs 1.1, 1.2, 1.3

## Main Ideas

Programming languages are used both to tell computers what to do and to organize ideas about computational processes.

The first major theme is abstraction: giving something complex a name so it can be used as a single unit.

Every powerful programming language needs:

- primitive expressions and statements
- ways to combine smaller pieces into larger ones
- ways to name and manipulate compound pieces as units

In early Python, the two main things being organized are data and functions.

- Data: values being manipulated
- Functions: rules for manipulating values

## Expressions and Statements

An expression evaluates to a value.

```py
42
2 + 3
max(4, 9)
```

A statement performs an action.

```py
x = 3
from math import sqrt
def square(x):
    return x * x
```

The distinction matters because expressions can be used inside larger expressions, but statements generally cannot.

```py
max(2 + 3, 10)
```

Here `2 + 3` is an expression inside another expression.

## Primitive Expressions

Numerals are primitive expressions.

```py
42
-1
3.14
```

A numeral evaluates to the number it names.

Names are also primitive expressions.

```py
radius
max
square
```

A name evaluates to the value bound to that name in the current environment.

## Call Expressions

A call expression applies a function to arguments.

```py
max(7.5, 9.5)
```

Parts of the call:

- `max` is the operator expression
- `7.5` and `9.5` are operand expressions
- the evaluated operand values are the arguments
- the result of applying the function is the return value

The order of arguments matters.

```py
pow(100, 2)   # 10000
pow(2, 100)   # 1267650600228229401496703205376
```

Function notation works well for nesting because the structure is explicit.

```py
max(min(1, -2), min(pow(3, 5), -4))
```

## Evaluating Call Expressions

Evaluation rule for call expressions:

1. Evaluate the operator expression.
2. Evaluate the operand expressions.
3. Apply the function value to the argument values.

Example:

```py
from operator import add, sub

sub(add(2, 3), add(10, 4))
```

Trace:

```text
sub(add(2, 3), add(10, 4))
sub(5, add(10, 4))
sub(5, 14)
-9
```

Evaluating a call expression can require evaluating smaller call expressions first. This is why the evaluation rule is recursive.

Expression trees show this structure: the full expression is the root, subexpressions are branches, and primitive expressions are leaves.

## Importing Functions

Python has many library functions, but their names are not all available by default. Import statements bind selected names from modules.

```py
from math import sqrt
sqrt(256)     # 16.0
```

The `operator` module contains function versions of common operators.

```py
from operator import add, sub, mul

add(14, 28)                       # 42
sub(100, mul(7, add(8, 4)))        # 16
```

Imported functions can be called like built-in functions.

## Names, Bindings, and Environments

A name is bound to a value when Python has associated that name with that value.

```py
radius = 10
```

After this assignment, `radius` is bound to `10`.

An environment stores name-value bindings. The meaning of a name depends on the environment where it is evaluated.

```py
2 * radius
```

This expression only has a value if `radius` has a binding in the current environment.

Names can refer to numbers, functions, and other values.

```py
max
```

This evaluates to the built-in `max` function, unless the name has been rebound.

## Assignment

Assignment is a basic form of abstraction. It gives a name to a value.

```py
area = 3.14159 * radius * radius
```

Python evaluates the expression on the right side first, then binds the name on the left.

```py
x = 2
x = x + 1
```

Trace:

```text
evaluate right side: x + 1 -> 2 + 1 -> 3
bind x to 3
```

Rebinding a name removes the old binding for that name.

```py
f = max
f(2, 3, 4)    # 4

f = 2
f             # 2
```

Names are not values themselves; names point to whatever value they are currently bound to.

## Rebinding Built-In Names

Built-in names can be rebound, which can cause confusing errors.

```py
max = 5
max
```

Now `max` is bound to `5`, not to the built-in function.

```py
max(2, 3, 4)
```

This causes a `TypeError` because an integer is being used as the operator in a call expression.

If another name was already bound to the original function, that name still works.

```py
f = max
max = 5
f(2, 3, 4)
```

Here `f` still refers to the function value that `max` referred to before `max` was rebound.

## Multiple Assignment

Python can bind multiple names in one assignment statement.

```py
area, circumference = pi * radius * radius, 2 * pi * radius
```

All right-side expressions are evaluated before any left-side names are bound.

This makes swapping possible:

```py
x, y = 3, 4.5
y, x = x, y
```

Final bindings:

```text
x -> 4.5
y -> 3
```

Changing one name does not automatically update another name.

```py
from math import pi

radius = 10
area = pi * radius * radius
radius = 11
area
```

`area` still has the value computed when `radius` was `10`.

## Pure and Non-Pure Functions

A pure function has no effects beyond returning a value. Calling it with the same arguments always returns the same result.

```py
abs(-2)      # 2
max(1, 5)    # 5
```

A non-pure function can produce side effects. A side effect changes something outside the return value, such as displaying output.

`print` is non-pure.

```py
print(1, 2, 3)
```

Output:

```text
1 2 3
```

The return value of `print` is always `None`.

```py
two = print(2)
print(two)
```

Output:

```text
2
None
```

The first line prints `2` as a side effect and binds `two` to `None`.

## Nested Print Example

Expression:

```py
print(print(1), print(2))
```

Evaluation:

```text
evaluate outer operator: print
evaluate first operand: print(1)
  side effect: displays 1
  return value: None
evaluate second operand: print(2)
  side effect: displays 2
  return value: None
apply outer print to None and None
  side effect: displays None None
  return value: None
```

Output:

```text
1
2
None None
```

The example shows why return values and side effects must be kept separate.

## Function Definitions

A function definition creates a user-defined function and binds a name to it.

```py
from operator import mul

def square(x):
    return mul(x, x)
```

Parts:

- `square`: function name
- `x`: formal parameter
- `return mul(x, x)`: function body / return expression

General form:

```py
def <name>(<formal parameters>):
    return <return expression>
```

The return expression is not evaluated when the function is defined. It is stored as part of the function and evaluated when the function is called.

```py
square(21)          # 441
square(square(3))   # 81
```

## Def Statements and Assignment

Both assignment statements and `def` statements bind names to values.

```py
def g():
    return 1

g()     # 1

g = 2
g       # 2

def g(h, i):
    return h + i

g(1, 2) # 3
```

Each new binding replaces the previous binding for that name.

## User-Defined Functions

User-defined functions are called the same way as built-in functions.

```py
from operator import add, mul

def square(x):
    return mul(x, x)

def sum_squares(x, y):
    return add(square(x), square(y))

sum_squares(3, 4)   # 25
```

The definition of `sum_squares` depends on the behavior of `square`, not on whether `square` is built in, imported, or user-defined.

## Function Signatures

A function signature describes the function's formal parameters.

```py
def square(x):
    return x * x
```

Signature:

```text
square(x)
```

`square` expects one argument.

```py
square(2, 3)   # too many arguments
square()       # too few arguments
```

Built-in functions are often written in diagrams with `...` because their formal parameter names are not shown.

```text
max(...)
mul(...)
```

## Frames

An environment is a sequence of frames.

A frame contains bindings from names to values.

There is a global frame. Assignment and import statements at the top level add bindings to the global frame.

```py
from math import pi
tau = 2 * pi
```

Global frame:

```text
pi -> 3.14159...
tau -> 6.28318...
```

Function calls create local frames.

## Calling User-Defined Functions

To apply a user-defined function:

1. Bind the argument values to the function's formal parameters in a new local frame.
2. Execute the body of the function in an environment beginning with that local frame.

Example:

```py
from operator import mul

def square(x):
    return mul(x, x)

square(-2)
```

Global frame:

```text
mul -> built-in multiplication function
square -> function square(x)
```

Local frame for `square(-2)`:

```text
x -> -2
return value -> 4
```

The top-level expression `square(-2)` is evaluated in the global environment. The return expression `mul(x, x)` is evaluated in the environment created by calling `square`.

## Name Lookup

A name evaluates to the value bound to that name in the earliest frame of the current environment where the name is found.

Inside this function:

```py
from operator import mul

def square(x):
    return mul(x, x)
```

Name lookup for `mul(x, x)`:

```text
x   -> found in the local square frame
mul -> found in the global frame
```

The order of frames matters. Local bindings are checked before global bindings.

## Multiple Function Calls and Frames

Each function call has its own local frame, even when the same function is called more than once.

```py
from operator import add, mul

def square(x):
    return mul(x, x)

def sum_squares(x, y):
    return add(square(x), square(y))

result = sum_squares(5, 12)
```

Evaluation sketch:

```text
call sum_squares(5, 12)
  local frame:
    x -> 5
    y -> 12

evaluate square(x)
  x is 5 in the sum_squares frame
  call square(5)
    new local frame:
      x -> 5
      return value -> 25

evaluate square(y)
  y is 12 in the sum_squares frame
  call square(12)
    new local frame:
      x -> 12
      return value -> 144

apply add to 25 and 144
sum_squares return value -> 169
```

The `x` in the `sum_squares` frame and the `x` in each `square` frame are different bindings.

## Local Names and Scope

Formal parameters are local to the body of the function.

These two definitions have the same behavior:

```py
def square(x):
    return x * x

def square(y):
    return y * y
```

The parameter name is an implementation detail as long as the function computes the same relationship between input and output.

The scope of a local name is limited to the function body where it is defined. Once the function call is finished, the local frame is gone.

## Bound Names and Intrinsic Names

A function has an intrinsic name from its definition. A frame can bind any name to that function.

```py
def square(x):
    return x * x

f = square
```

`square` and `f` both refer to the same function value, but the function's intrinsic name is still `square`.

During evaluation, Python uses the bound name that appears in the expression.

```py
f(4)
```

Python looks up `f` in the current environment and applies the function value it finds.

## Functional Abstraction

A function definition can hide implementation details.

```py
def square(x):
    return x * x

def sum_squares(x, y):
    return square(x) + square(y)
```

`sum_squares` relies on what `square` does, not on how `square` does it.

Different implementations can represent the same abstraction:

```py
def square(x):
    return x * x

def square(x):
    return x ** 2
```

Both implementations have the same intent for numeric inputs.

The main attributes of a functional abstraction:

- Domain: the set of valid inputs
- Range: the set of possible return values
- Intent: the relationship between inputs and output

For `square`:

```text
Domain: one real number
Range: a non-negative real number
Intent: return the input multiplied by itself
```

## Naming

Clear names make function definitions easier to read.

Common Python naming conventions:

- function names are lowercase
- words are separated by underscores
- parameter names are lowercase
- parameter names should describe the role of the argument
- single-letter names are acceptable when the role is obvious

Examples:

```py
def square(x):
    return x * x

def calculate_area(radius):
    return 3.14159 * radius * radius
```

Avoid names that are easy to confuse with numerals:

```text
l, O, I
```

## Operators

Infix operators such as `+`, `-`, `*`, and `/` are special expression forms, but they can often be understood as shorthand for function calls.

```py
2 + 3
```

Similar meaning:

```py
from operator import add
add(2, 3)
```

Operator precedence controls how expressions are grouped.

```py
2 + 3 * 4 + 5
```

Equivalent structure:

```py
add(add(2, mul(3, 4)), 5)
```

Parentheses override normal precedence.

```py
(2 + 3) * (4 + 5)
```

Equivalent structure:

```py
mul(add(2, 3), add(4, 5))
```

Idiomatic Python usually uses infix operators for simple arithmetic.

## Division

Python has two main division operators in this part of the course.

True division:

```py
5 / 4    # 1.25
8 / 4    # 2.0
```

Floor division:

```py
5 // 4     # 1
-5 // 4    # -2
```

`//` rounds down to the nearest integer, not toward zero.

Function equivalents:

```py
from operator import truediv, floordiv

truediv(5, 4)    # 1.25
floordiv(5, 4)   # 1
```

## Debugging Principles

Early debugging usually comes down to checking bindings, return values, and side effects.

Important habits:

- test small pieces independently
- isolate the smallest expression or function causing the issue
- check what each name is bound to
- separate printed output from returned values
- trace nested call expressions one call at a time

## Key Traces

### Assignment

```py
x = 2
x = x + 1
x
```

Trace:

```text
x -> 2
evaluate x + 1 -> 3
rebind x -> 3
```

Final value:

```text
3
```

### Multiple Assignment Swap

```py
x, y = 3, 4.5
y, x = x, y
```

Trace:

```text
evaluate right side of second assignment:
  x -> 3
  y -> 4.5

bind left side:
  y -> 3
  x -> 4.5
```

### Function Rebinding

```py
def double(x):
    return x + x

twice = double
double = 7
twice(5)
```

Trace:

```text
double -> function double(x)
twice -> same function
double -> 7
twice(5) -> 10
```

### User-Defined Function Call

```py
def square(x):
    return x * x

square(6)
```

Trace:

```text
look up square -> function square(x)
evaluate operand 6 -> 6
create local frame:
  x -> 6
evaluate return expression:
  x * x -> 36
return 36
```

### Nested Function Calls

```py
def square(x):
    return x * x

def sum_squares(x, y):
    return square(x) + square(y)

sum_squares(2, 5)
```

Trace:

```text
sum_squares frame:
  x -> 2
  y -> 5

square(2) frame:
  x -> 2
  return -> 4

square(5) frame:
  x -> 5
  return -> 25

sum_squares return -> 29
```

## Common Pitfalls

Confusing printed output with return values:

```py
x = print(3)
```

`x` is bound to `None`, not `3`.

Expecting old assignments to update automatically:

```py
radius = 10
area = 3.14 * radius * radius
radius = 20
```

`area` still has the old computed value.

Rebinding a function name to a non-function value:

```py
max = 5
max(1, 2)
```

The call fails because `max` is no longer bound to the built-in function.

Forgetting that each function call has a fresh local frame:

```py
def f(x):
    return x + 1

f(2)
f(10)
```

The `x` in the first call and the `x` in the second call are separate local bindings.
