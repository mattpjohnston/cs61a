# 03 - Higher-Order Functions

Reading: Composing Programs 1.6

## Main Ideas

Functions are values.

That means a function can be:

- bound to a name
- passed as an argument
- returned from another function
- stored in a data structure

A higher-order function is a function that takes another function as an argument or returns a function as its result.

Higher-order functions let us abstract over patterns of computation, not just over individual values.

Earlier abstraction:

```py
def square(x):
    return x * x
```

This abstracts over the number being squared.

Higher-order abstraction:

```py
def apply_twice(f, x):
    return f(f(x))
```

This abstracts over the operation being applied.

## Functions as Values

A function name evaluates to a function value.

```py
def square(x):
    return x * x

square
```

The name `square` is bound to a function.

That function can be assigned to another name.

```py
f = square
f(5)       # 25
```

Both `f` and `square` refer to the same function value.

Calling a function is different from referring to it.

```py
square       # the function value
square(5)    # the return value from calling the function
```

This distinction is especially important with higher-order functions.

## Higher-Order Functions

A higher-order function manipulates functions.

Function as argument:

```py
def apply_once(f, x):
    return f(x)
```

Function as return value:

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

Both are higher-order functions.

Higher-order functions are useful when several pieces of code share the same structure but differ in one operation.

## Functions as Arguments

Suppose several functions sum different terms.

```py
def sum_naturals(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total
```

```py
def sum_cubes(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + k * k * k, k + 1
    return total
```

These functions have the same control structure.

Only the term being added changes.

Common pattern:

```py
def <name>(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + <term>(k), k + 1
    return total
```

We can turn the changing part into a parameter.

```py
def summation(n, term):
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), k + 1
    return total
```

Here `term` is a formal parameter that will be bound to a function.

## Summation Pattern

Helper functions describe the individual terms.

```py
def identity(x):
    return x

def cube(x):
    return x * x * x
```

Then the specific summations become short.

```py
def sum_naturals(n):
    return summation(n, identity)

def sum_cubes(n):
    return summation(n, cube)
```

Calls:

```py
sum_naturals(5)    # 15
sum_cubes(3)       # 36
```

The same `summation` function can also be called directly.

```py
summation(5, cube)       # 225
summation(5, identity)   # 15
```

The key idea is that `summation` captures the repeated process:

```text
start total at 0
start k at 1
while k is in range:
  add the kth term
  advance k
return total
```

The `term` function supplies the part that changes.

## Passing Functions vs Calling Functions

To pass a function as an argument, use its name without parentheses.

```py
summation(3, cube)
```

This passes the function `cube`.

Inside `summation`, Python calls it later:

```py
term(k)
```

Using parentheses calls the function immediately.

```py
summation(3, cube(3))
```

This is wrong for `summation`.

Trace:

```text
cube(3) -> 27
summation(3, 27)
```

Now `term` is bound to `27`, not to a function.

When the body tries to evaluate `term(k)`, Python attempts to call an integer.

That causes a `TypeError`.

## Function Signatures

A higher-order function expects argument functions with certain signatures.

For `summation`:

```py
def summation(n, term):
    ...
    total = total + term(k)
```

The `term` function must take one argument.

This works:

```py
def square(x):
    return x * x

summation(4, square)
```

This does not work:

```py
def add(x, y):
    return x + y

summation(4, add)
```

When `summation` evaluates `term(k)`, it only supplies one argument.

`add` needs two arguments, so the call fails.

Higher-order functions are flexible, but the functions they receive still need compatible inputs and outputs.

## General Methods

Higher-order functions can represent general methods of computation.

A general method is not tied to one specific problem.

For example, iterative improvement has this shape:

```text
start with a guess
while the guess is not good enough:
  improve the guess
return the final guess
```

The structure is independent of what problem is being solved.

We can write it as a function.

```py
def improve(update, close, guess=1):
    while not close(guess):
        guess = update(guess)
    return guess
```

Parameters:

- `update`: function that produces a better guess
- `close`: predicate function that checks whether the guess is good enough
- `guess`: starting point

The body of `improve` does not know what equation is being solved.

That knowledge is supplied by the functions passed in.

## Predicate Functions

A predicate function returns a truth value.

```py
def is_positive(x):
    return x > 0
```

In `improve`, `close` is a predicate.

```py
while not close(guess):
```

The loop continues while the current guess is not close enough.

Predicate functions are common arguments to higher-order functions because they let callers customize decisions.

## Approximate Equality

Floating point computations often produce approximations.

Instead of checking exact equality, compare the size of the difference.

```py
def approx_eq(x, y, tolerance=1e-15):
    return abs(x - y) < tolerance
```

Examples:

```py
approx_eq(0.1 + 0.2, 0.3)       # True
approx_eq(1.0, 1.1)             # False
```

The optional `tolerance` controls how close the values need to be.

## Golden Ratio Example

The golden ratio can be computed by iterative improvement.

One update rule is:

```py
def golden_update(guess):
    return 1 / guess + 1
```

One close check is:

```py
def square_close_to_successor(guess):
    return approx_eq(guess * guess, guess + 1)
```

Then:

```py
phi = improve(golden_update, square_close_to_successor)
```

The important abstraction is that `improve` is reused unchanged.

Only the update and close functions change.

Trace with a loose tolerance:

```py
def approx_eq(x, y, tolerance=0.01):
    return abs(x - y) < tolerance
```

```text
guess -> 1
close(1) -> approx_eq(1, 2) -> False
guess -> golden_update(1) -> 2.0

close(2.0) -> approx_eq(4.0, 3.0) -> False
guess -> golden_update(2.0) -> 1.5

close(1.5) -> approx_eq(2.25, 2.5) -> False
guess -> golden_update(1.5) -> 1.666...

eventually close(guess) -> True
return guess
```

## Testing Higher-Order Functions

A higher-order function should be tested with small, predictable argument functions.

```py
def increment(x):
    return x + 1

def double(x):
    return 2 * x
```

Example tests:

```py
assert apply_once(increment, 3) == 4
assert apply_once(double, 3) == 6
assert apply_twice(increment, 3) == 5
```

Testing the general method separately from complicated argument functions makes bugs easier to locate.

For `summation`, start with simple terms.

```py
assert summation(4, identity) == 10
assert summation(3, cube) == 36
```

## Nested Definitions

A function can be defined inside another function.

```py
def sqrt(a):
    def sqrt_update(x):
        return (x + a / x) / 2

    def sqrt_close(x):
        return approx_eq(x * x, a)

    return improve(sqrt_update, sqrt_close)
```

The functions `sqrt_update` and `sqrt_close` are local to the call to `sqrt`.

They are not bound in the global frame.

```py
sqrt(16)         # 4.0
sqrt_update      # NameError
```

Nested definitions help when helper functions are only useful inside one larger function.

They also let helper functions refer to names from the enclosing function.

## Local Function Names

Local `def` statements behave like local assignment.

```py
def outer(x):
    def inner(y):
        return x + y
    return inner(4)
```

When `outer(3)` is called:

```text
create outer frame:
  x -> 3

execute def inner(y):
  inner -> function inner(y), parent outer frame

return inner(4)
```

The name `inner` exists in the local frame for `outer`.

It does not exist globally.

This keeps the global frame from being cluttered with helper names.

## Lexical Scope

Lexical scope means that an inner function can refer to names in the environment where it was defined.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The function `adder` refers to `n`.

`n` is not a parameter of `adder`.

`n` comes from the enclosing call to `make_adder`.

```py
add_three = make_adder(3)
add_three(10)       # 13
```

The name `n` is looked up in the parent environment of `adder`.

Lexical scope depends on where a function is defined, not where it is called.

## Parent Environments

Every user-defined function has a parent environment.

The parent environment is the environment in which the function was defined.

For functions defined in the global frame:

```py
def square(x):
    return x * x
```

The parent is the global frame.

For functions defined inside another function:

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The parent of `adder` is the frame created by a particular call to `make_adder`.

Function call rule with parent environments:

1. Create a new local frame for the call.
2. Bind formal parameters to argument values in that frame.
3. Set the parent of the new frame to the function's parent environment.
4. Evaluate the function body in the new environment.

The environment can be a chain of frames.

```text
adder frame:
  k -> 10
parent -> make_adder frame:
  n -> 3
parent -> global frame
```

Looking up a name searches the current frame first, then parent frames.

## Closures

A closure is a function together with the environment it needs.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

When `make_adder(3)` returns `adder`, the call to `make_adder` is finished.

But the returned function still needs the binding:

```text
n -> 3
```

The returned function keeps access to that environment.

That is why this works:

```py
add_three = make_adder(3)
add_three(4)      # 7
```

Different calls create different closures.

```py
add_three = make_adder(3)
add_ten = make_adder(10)

add_three(5)      # 8
add_ten(5)        # 15
```

The two returned functions have the same body but different parent environments.

## Square Root with Nested Helpers

Without nested definitions, an update helper might need two arguments.

```py
def sqrt_update(x, a):
    return (x + a / x) / 2
```

But `improve` expects an update function that takes one argument.

```py
guess = update(guess)
```

Nested definitions adapt the helper to the expected signature.

```py
def sqrt(a):
    def update(x):
        return (x + a / x) / 2

    def close(x):
        return approx_eq(x * x, a)

    return improve(update, close)
```

`update` takes one argument, as `improve` expects.

It still has access to `a` through lexical scope.

This is a common pattern:

```text
outer function receives problem-specific values
inner functions use those values
outer function passes inner functions to a general method
```

## Functions as Returned Values

A function can return another function.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

Call:

```py
add_three = make_adder(3)
```

Now `add_three` is a function.

```py
add_three(4)      # 7
```

The expression can also be called immediately.

```py
make_adder(3)(4)  # 7
```

Evaluation:

```text
make_adder(3) -> function adder(k) with n -> 3
that returned function is then called on 4
return 3 + 4 -> 7
```

## Function Composition

Function composition combines two functions into one.

If:

```text
h(x) = f(g(x))
```

then `h` is the composition of `f` after `g`.

Python version:

```py
def compose1(f, g):
    def h(x):
        return f(g(x))
    return h
```

The `1` in `compose1` is a naming convention meaning the composed functions take one argument.

Example:

```py
def square(x):
    return x * x

def successor(x):
    return x + 1

square_successor = compose1(square, successor)
square_successor(4)      # 25
```

Trace:

```text
square_successor(4)
h(4)
square(successor(4))
square(5)
25
```

The order matters.

```py
successor_square = compose1(successor, square)
successor_square(4)      # 17
```

This computes:

```text
successor(square(4))
successor(16)
17
```

## Returned Functions Keep Their Environment

The returned function from `compose1` needs access to `f` and `g`.

```py
def compose1(f, g):
    def h(x):
        return f(g(x))
    return h
```

After `compose1(square, successor)` returns, the returned function `h` still knows:

```text
f -> square
g -> successor
```

That is a closure.

This works even if global names change later.

```py
square_successor = compose1(square, successor)

square = 0
successor = 0

square_successor(4)      # 25
```

The returned function uses the bindings in its parent environment, where `f` and `g` were already bound to the original function values.

## Newton's Method

Newton's method is a general method for finding zeros of functions.

A zero of a function `f` is a value `x` where:

```text
f(x) = 0
```

For example, the square root of `a` can be found by solving:

```text
x * x - a = 0
```

Newton's method repeatedly improves a guess using the function and its derivative.

The update has this form:

```py
def newton_update(f, df):
    def update(x):
        return x - f(x) / df(x)
    return update
```

Parameters:

- `f`: function whose zero we want
- `df`: derivative of `f`

Return value:

- an update function that improves one guess

This is a higher-order function because it takes functions as arguments and returns a function.

## Finding Zeros

The general zero-finding function can reuse `improve`.

```py
def find_zero(f, df):
    def near_zero(x):
        return approx_eq(f(x), 0)
    return improve(newton_update(f, df), near_zero)
```

Pieces:

- `newton_update(f, df)` builds the update function
- `near_zero` checks whether the current guess is good enough
- `improve` handles the loop

The same improvement engine is used again.

Only the problem-specific functions change.

## Square Roots with Newton's Method

To compute a square root of `a`, find a zero of:

```text
f(x) = x * x - a
```

Its derivative is:

```text
df(x) = 2 * x
```

Implementation:

```py
def square_root_newton(a):
    def f(x):
        return x * x - a

    def df(x):
        return 2 * x

    return find_zero(f, df)
```

Example:

```py
square_root_newton(64)     # 8.0
```

The function `square_root_newton` does not contain a loop.

The loop is inside `improve`.

The update rule is built by `newton_update`.

The stopping condition is built by `find_zero`.

This is the point of the abstraction: each function has one job.

## Roots of Any Degree

An nth root can be computed by solving:

```text
x ** n - a = 0
```

For CS61A-style code before using `**` heavily, power can be written with a loop.

```py
def power(x, n):
    product, k = 1, 0
    while k < n:
        product, k = product * x, k + 1
    return product
```

Then:

```py
def nth_root(n, a):
    def f(x):
        return power(x, n) - a

    def df(x):
        return n * power(x, n - 1)

    return find_zero(f, df)
```

Examples:

```py
nth_root(2, 64)     # 8.0
nth_root(3, 64)     # 4.0
nth_root(6, 64)     # 2.0
```

Newton's method is powerful, but it is still an approximation method.

It may fail to converge for some functions or starting guesses.

## Currying

Currying converts a function that takes multiple arguments into a chain of functions that each take one argument.

Normal two-argument call:

```py
pow(2, 5)       # 32
```

Curried form:

```py
def curried_pow(x):
    def h(y):
        return pow(x, y)
    return h
```

Call:

```py
curried_pow(2)(5)      # 32
```

Evaluation:

```text
curried_pow(2) -> function h(y), with x -> 2
h(5) -> pow(2, 5) -> 32
```

Currying is useful when a function expects a one-argument function, but we want to specialize a two-argument function.

## Map Pattern

A simple map pattern applies a function to each value in a range.

```py
def map_to_range(start, end, f):
    while start < end:
        print(f(start))
        start = start + 1
```

`map_to_range` expects `f` to take one argument.

To print powers of two:

```py
map_to_range(0, 5, curried_pow(2))
```

Output:

```text
1
2
4
8
16
```

`curried_pow(2)` returns a one-argument function.

That returned function takes the exponent.

```text
f(0) -> pow(2, 0)
f(1) -> pow(2, 1)
f(2) -> pow(2, 2)
```

The same pattern can print powers of another base.

```py
map_to_range(0, 4, curried_pow(3))
```

Output:

```text
1
3
9
27
```

## Curry and Uncurry Helpers

Currying can be generalized for two-argument functions.

```py
def curry2(f):
    def g(x):
        def h(y):
            return f(x, y)
        return h
    return g
```

`curry2` takes a two-argument function and returns a curried version.

```py
pow_curried = curry2(pow)
pow_curried(2)(5)       # 32
```

Uncurrying reverses the transformation.

```py
def uncurry2(g):
    def f(x, y):
        return g(x)(y)
    return f
```

Example:

```py
pow_again = uncurry2(pow_curried)
pow_again(2, 5)         # 32
```

Equivalences:

```text
curry2(f)(x)(y) == f(x, y)
uncurry2(g)(x, y) == g(x)(y)
```

The returned functions rely on lexical scope to remember `f`, `g`, and `x`.

## Lambda Expressions

A lambda expression creates a function value.

```py
lambda x: x * x
```

This means:

```text
a function that takes x and returns x * x
```

It can be bound to a name.

```py
square = lambda x: x * x
square(5)       # 25
```

It can also be passed directly as an argument.

```py
summation(5, lambda x: x * x)
```

A lambda expression evaluates to a function.

Calling the function is a separate step.

```py
(lambda x: x + 1)(4)      # 5
```

## Lambda Syntax

General form:

```py
lambda <parameters>: <return expression>
```

Examples:

```py
lambda x: x + 1
lambda x, y: x + y
lambda: 42
```

The body of a lambda is a single expression.

Statements are not allowed inside the body.

Allowed:

```py
lambda x: x * x
```

Not allowed:

```py
lambda x: return x * x
```

Also not allowed:

```py
lambda x: total = total + x
```

`return` and assignment are statements, not expressions.

## Lambda vs Def

These two functions behave similarly:

```py
def square(x):
    return x * x
```

```py
square = lambda x: x * x
```

But a `def` statement gives the function an intrinsic name.

Lambda functions are unnamed.

Python displays their name as `<lambda>`.

```py
f = lambda x: x + 1
f
```

Output will look like:

```text
<function <lambda> at ...>
```

Use `def` for functions that need a clear name, multiple lines, a docstring, or complex logic.

Use `lambda` for short functions that are clearer inline.

Good use:

```py
summation(10, lambda k: k * k)
```

Harder to read:

```py
compose1 = lambda f, g: lambda x: f(g(x))
```

The second version works, but nested lambdas can become difficult to read.

## Lambda and Composition

`compose1` can be written with a nested `def`.

```py
def compose1(f, g):
    def h(x):
        return f(g(x))
    return h
```

It can also be written with a lambda.

```py
def compose1(f, g):
    return lambda x: f(g(x))
```

These are equivalent for this simple case.

The lambda version still creates a function with a parent environment.

That function still remembers `f` and `g`.

## First-Class Functions

A value has first-class status if the language lets it be used like other ordinary values.

In Python, functions are first-class.

They can be bound to names.

```py
f = abs
```

They can be passed as arguments.

```py
summation(5, abs)
```

They can be returned from functions.

```py
def choose_adder(n):
    def adder(k):
        return n + k
    return adder
```

They can be stored in data structures.

```py
funcs = [abs, square, cube]
```

Function values being first-class is what makes higher-order functions practical.

## Function Decorators

A decorator is syntax for applying a higher-order function to a newly defined function.

Example higher-order function:

```py
def trace(fn):
    def wrapped(x):
        print('->', fn, '(', x, ')')
        return fn(x)
    return wrapped
```

Use as a decorator:

```py
@trace
def triple(x):
    return 3 * x
```

Calling:

```py
triple(4)
```

Output:

```text
-> <function triple at ...> ( 4 )
```

Return value:

```text
12
```

The printed line is a side effect from the wrapper.

## Decorator Execution

This code:

```py
@trace
def triple(x):
    return 3 * x
```

is equivalent to:

```py
def triple(x):
    return 3 * x

triple = trace(triple)
```

Execution rule:

1. Create the original `triple` function.
2. Evaluate the decorator expression `trace`.
3. Call `trace` on the original `triple`.
4. Bind the name `triple` to the returned function.

After decoration, `triple` no longer refers directly to the original function.

It refers to the wrapper returned by `trace`.

The wrapper still has access to the original function through lexical scope.

## Designing with Higher-Order Functions

Higher-order functions are helpful when they make a repeated pattern explicit.

Good uses:

- repeated loop structure with different term functions
- repeated improvement process with different update and close functions
- building specialized functions from general functions
- adding behavior around a function call, as with decorators

They are less helpful when the abstraction hides simple logic or makes signatures hard to follow.

A useful higher-order function usually has:

- a clear name for the pattern
- simple expectations for its function arguments
- tests with small helper functions
- a body that is easier to understand than several duplicated versions

The goal is not to make code abstract for its own sake.

The goal is to express the important process once and reuse it clearly.

## Key Traces

### Passing a Function Argument

```py
def cube(x):
    return x * x * x

def apply_once(f, x):
    return f(x)

apply_once(cube, 3)
```

Trace:

```text
look up apply_once -> function apply_once(f, x)
evaluate cube -> function cube(x)
evaluate 3 -> 3

create apply_once frame:
  f -> function cube(x)
  x -> 3

evaluate return expression:
  f(x)
  cube(3)
  return 27
```

### Passing a Return Value by Mistake

```py
def cube(x):
    return x * x * x

apply_once(cube(3), 4)
```

Trace:

```text
evaluate operator apply_once -> function
evaluate first operand cube(3):
  cube(3) -> 27
evaluate second operand 4 -> 4

create apply_once frame:
  f -> 27
  x -> 4

evaluate f(x):
  27(4)
```

Result:

```text
TypeError
```

`f` is bound to an integer, not a function.

### Summation

```py
def summation(n, term):
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), k + 1
    return total

def cube(x):
    return x * x * x

summation(3, cube)
```

Trace:

```text
summation frame:
  n -> 3
  term -> function cube(x)

start:
  total -> 0
  k -> 1

k <= 3:
  term(k) -> cube(1) -> 1
  total, k -> 1, 2

k <= 3:
  term(k) -> cube(2) -> 8
  total, k -> 9, 3

k <= 3:
  term(k) -> cube(3) -> 27
  total, k -> 36, 4

k <= 3 is false
return 36
```

### Nested Definition and Lexical Scope

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder

add_three = make_adder(3)
add_three(10)
```

Trace:

```text
call make_adder(3)

make_adder frame:
  n -> 3

execute def adder(k):
  adder -> function adder(k), parent make_adder frame

return adder
bind add_three -> returned function

call add_three(10)

adder frame:
  k -> 10
parent make_adder frame:
  n -> 3

evaluate n + k:
  n found in parent frame -> 3
  k found in local frame -> 10
return 13
```

### Multiple Closures

```py
add_three = make_adder(3)
add_ten = make_adder(10)
```

Trace:

```text
make_adder(3) creates:
  function adder(k), parent frame with n -> 3

make_adder(10) creates:
  function adder(k), parent frame with n -> 10
```

Calls:

```text
add_three(5) -> 8
add_ten(5) -> 15
```

The body of both functions is the same.

Their parent environments are different.

### Composition

```py
def compose1(f, g):
    def h(x):
        return f(g(x))
    return h

def square(x):
    return x * x

def successor(x):
    return x + 1

square_successor = compose1(square, successor)
square_successor(4)
```

Trace:

```text
compose1 frame:
  f -> function square(x)
  g -> function successor(x)

return h with parent compose1 frame

call square_successor(4)

h frame:
  x -> 4
parent compose1 frame:
  f -> square
  g -> successor

evaluate f(g(x)):
  g(x) -> successor(4) -> 5
  f(5) -> square(5) -> 25
return 25
```

### Currying

```py
def curry2(f):
    def g(x):
        def h(y):
            return f(x, y)
        return h
    return g

pow_curried = curry2(pow)
pow_curried(2)(5)
```

Trace:

```text
curry2(pow) -> function g(x), parent frame with f -> pow
pow_curried -> g

pow_curried(2):
  g frame:
    x -> 2
  parent curry2 frame:
    f -> pow
  return h(y), parent g frame

h(5):
  y -> 5
  x found in parent g frame -> 2
  f found through parent chain -> pow
  return pow(2, 5) -> 32
```

### Lambda Expression

```py
f = lambda x: x + 1
f(4)
```

Trace:

```text
evaluate lambda x: x + 1 -> function <lambda>(x)
bind f -> function <lambda>(x)

call f(4):
  x -> 4
  return x + 1 -> 5
```

### Decorator

```py
def trace(fn):
    def wrapped(x):
        print('call', x)
        return fn(x)
    return wrapped

@trace
def triple(x):
    return 3 * x

triple(4)
```

Trace:

```text
create original triple function
call trace(original triple)

trace frame:
  fn -> original triple function
  wrapped -> function wrapped(x), parent trace frame
return wrapped

bind triple -> wrapped

call triple(4):
  call wrapped(4)
  print side effect: call 4
  fn(4) -> original triple(4) -> 12
  return 12
```

## Common Pitfalls

Calling a function when you meant to pass it:

```py
summation(3, cube(3))
```

`cube(3)` evaluates to `27`, so `summation` receives a number instead of a function.

Passing a function with the wrong number of parameters:

```py
def add(x, y):
    return x + y

summation(5, add)
```

`summation` calls `term(k)` with one argument, but `add` needs two.

Forgetting that returned functions can be called:

```py
make_adder(3, 4)
```

If `make_adder` returns a function, the correct call shape is:

```py
make_adder(3)(4)
```

Expecting local helper functions to exist globally:

```py
def outer(x):
    def inner(y):
        return x + y
    return inner(1)

inner(2)
```

`inner` is local to calls of `outer`.

Confusing where a name is looked up:

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder

n = 100
add_three = make_adder(3)
add_three(4)
```

The result is `7`, not `104`.

`adder` uses the `n` from the frame where it was defined.

Writing lambdas with statements:

```py
lambda x: return x + 1
```

The body of a lambda must be an expression.

Making lambdas too dense:

```py
f = lambda a: lambda b: lambda c: a(b(c))
```

This may work, but a nested `def` version is usually easier to read.

Forgetting that decorators rebind the function name:

```py
@trace
def triple(x):
    return 3 * x
```

After this definition, `triple` is bound to the function returned by `trace`, not directly to the original `triple`.

Over-abstracting too early:

```py
def do_thing(strategy, checker, updater, combiner):
    ...
```

Higher-order functions should make a real repeated pattern clearer.

If the argument functions are hard to name or the call is hard to read, the abstraction may not be helping yet.
