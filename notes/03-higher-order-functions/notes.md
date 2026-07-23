# 03 - Higher-Order Functions

Reading: Composing Programs 1.6

## Core Idea

Functions are values. They can be:

- bound to names
- passed as arguments
- returned from functions
- created by lambda expressions

A higher-order function takes a function as an argument or returns a function.

```py
def apply_twice(f, x):
    return f(f(x))
```

Higher-order functions abstract over **processes**, not only values.

## Function Values vs Calls

```py
def square(x):
    return x * x

f = square
f(5)  # 25
```

```py
square     # function value
square(5)  # return value from calling it
```

Pass a function without parentheses when another function should call it later.

## Functions as Arguments

Several functions may share one computational pattern and differ only in one operation.

```py
def summation(n, term):
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), k + 1
    return total
```

```py
def identity(x):
    return x


def cube(x):
    return x * x * x

summation(5, identity)  # 15
summation(3, cube)      # 36
```

`summation` captures the repeated process. `term` supplies the part that varies.

The function argument must have a compatible signature. `summation` calls `term(k)`, so `term` must accept one argument.

## Passing vs Calling

```py
summation(3, cube)     # pass the cube function
summation(3, cube(3))  # pass 27: wrong type for term
```

In the second call, `cube(3)` is evaluated before `summation` starts. Later, `term(k)` attempts to call an integer.

## General Computational Methods

A higher-order function can express a reusable method.

```py
def improve(update, close, guess=1):
    while not close(guess):
        guess = update(guess)
    return guess
```

- `update` produces a better guess.
- `close` decides when the guess is good enough.
- `improve` controls the repeated process.

Problem-specific behavior is supplied as functions rather than built into the loop.

## Nested Function Definitions

A function can define local helper functions.

```py
def sqrt(a):
    def update(x):
        return (x + a / x) / 2

    def close(x):
        return abs(x * x - a) < 1e-12

    return improve(update, close)
```

Benefits:

- helper names stay local
- helpers can use names from the enclosing function
- helpers can adapt a general method to a specific problem

The environment mechanics behind this behavior are covered in chapter 04.

## Functions as Return Values

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

```py
add_three = make_adder(3)
add_three(4)      # 7
make_adder(3)(4)  # 7
```

The returned function keeps access to the environment where it was defined. This function-plus-environment behavior is a closure.

## Function Composition

Composition creates a function that applies one function after another.

```py
def compose1(f, g):
    def composed(x):
        return f(g(x))
    return composed
```

```py
def square(x):
    return x * x


def successor(x):
    return x + 1

compose1(square, successor)(4)  # square(5) -> 25
compose1(successor, square)(4)  # successor(16) -> 17
```

Composition order matters.

## Currying

Currying converts a multi-argument function into a chain of one-argument functions.

```py
def curry2(f):
    def first(x):
        def second(y):
            return f(x, y)
        return second
    return first
```

```py
curried_pow = curry2(pow)
curried_pow(2)(5)  # 32
```

Uncurrying reverses the transformation.

```py
def uncurry2(g):
    def f(x, y):
        return g(x)(y)
    return f
```

Currying is useful when an interface expects a one-argument function and one argument should be fixed in advance.

## Lambda Expressions

A lambda expression creates an unnamed function.

```py
lambda x: x * x
```

General form:

```text
lambda <parameters>: <expression>
```

```py
square = lambda x: x * x
summation(5, lambda k: k * k)
(lambda x: x + 1)(4)  # 5
```

A lambda body is one expression. Statements such as assignment and `return` are not allowed.

Use `def` for named, documented, or multi-step functions. Use `lambda` for short behavior that is clearer inline.

## Decorators

A decorator transforms a function when its `def` statement executes.

```py
def trace(fn):
    def wrapped(x):
        print('calling', fn.__name__, 'with', x)
        return fn(x)
    return wrapped


@trace
def square(x):
    return x * x
```

This is equivalent to:

```py
def square(x):
    return x * x

square = trace(square)
```

The name `square` becomes bound to the returned wrapper function.

## Designing with Higher-Order Functions

Use a higher-order function when:

- several functions share the same control structure
- only one operation or decision varies
- a function needs to delay or customize behavior
- a function should construct a specialized function

Do not introduce higher-order structure if ordinary direct code is clearer.

## Common Traps

- Calling a function when it should be passed as a value.
- Passing a function with an incompatible number of parameters.
- Forgetting that a returned value may itself be a function.
- Reversing function-composition order.
- Writing unreadable nested lambdas instead of using `def`.
- Assuming an inner helper is globally available.
- Forgetting that different outer calls create different closures.
- Reimplementing a general loop instead of parameterizing the changing behavior.
- Adding abstraction that does not make the program clearer or more reusable.
