# 04 - Environments

Reading: Composing Programs 1.6

## Core Idea

An environment determines what names mean when expressions are evaluated.

An environment is a sequence of frames. A frame contains name-value bindings.

Environment diagrams track:

- bindings
- function values
- call frames
- parent links
- return values
- shared objects

## Global Frame

Top-level assignment, import, and `def` statements add bindings to the global frame.

```py
x = 3

def square(n):
    return n * n
```

```text
Global:
  x -> 3
  square -> func square(n) [parent=Global]
```

## Function Values and Parent Frames

Executing a `def` statement creates a function value with:

- intrinsic name
- formal parameters
- body
- parent frame

The parent is the frame where the function was defined.

```py
def square(n):
    return n * n
```

```text
func square(n) [parent=Global]
```

The body is not executed until the function is called.

## Calling a User-Defined Function

For each call:

1. Evaluate the operator and operands.
2. Create a fresh local frame.
3. Set its parent to the called function's parent.
4. Bind parameters to argument values.
5. Execute the body there.
6. Record the return value.

```py
result = square(4)
```

```text
Global:
  square -> func square(n) [parent=Global]
  result -> 16

f1: square [parent=Global]
  n -> 4
  return value -> 16
```

Built-in function calls normally do not get frames in CS61A environment diagrams.

## Name Lookup

To evaluate a name:

1. Search the current frame.
2. If absent, follow the parent link.
3. Continue to Global.
4. Python can then use the built-in namespace, though CS61A diagrams usually do not draw it.
5. If no binding exists, raise `NameError`.

The first matching binding wins.

## Local Bindings and Shadowing

```py
x = 10

def f(x):
    y = x + 1
    return y
```

Calling `f(3)` creates:

```text
f1: f [parent=Global]
  x -> 3
  y -> 4
  return value -> 4
```

The local `x` shadows global `x` while the body runs. Global `x` remains `10`.

Ordinary assignment inside a function binds in the current local frame; it does not follow parent links to replace an outer binding.

## Every Call Gets a Fresh Frame

```py
a = square(2)
b = square(5)
```

This creates two call frames with separate `n` bindings.

```text
f1: square
  n -> 2
  return value -> 4

f2: square
  n -> 5
  return value -> 25
```

Never reuse one frame for multiple calls.

## Parent Links vs Call Order

A frame's parent is determined by where the function was **defined**, not where it was **called**.

```py
def double(x):
    return x * 2


def apply_twice(f, x):
    return f(f(x))
```

Calls to `double` have `Global` as parent because `double` was defined globally. They do not have the `apply_twice` frame as parent.

Keep these separate:

- parent links control name lookup
- the call stack records which calls are waiting to finish

Recursive calls can wait for one another without being lexical parents of one another.

## Higher-Order Function Frames

```py
result = apply_twice(double, 3)
```

```text
apply_twice frame:
  f -> func double(x) [parent=Global]
  x -> 3
```

A parameter can be bound to a function value like any other value.

Passing `double(3)` instead would bind `f` to `6`, and later attempting `f(...)` would fail.

## Nested Definitions

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The `adder` function is created when `make_adder` runs. Its parent is that particular `make_adder` call frame.

```text
f1: make_adder [parent=Global]
  n -> 3
  adder -> func adder(k) [parent=f1]
```

## Lexical Scope and Closures

```py
add_three = make_adder(3)
result = add_three(4)
```

Calling `add_three(4)` creates:

```text
f2: adder [parent=f1]
  k -> 4
  return value -> 7
```

Lookup for `n + k`:

```text
k -> found in f2
n -> found in parent f1
```

The returned function keeps `f1` reachable after `make_adder` returns. A function together with the environment it needs is a closure.

Different outer calls create different closures.

```py
add_two = make_adder(2)
add_five = make_adder(5)
```

The returned functions have the same body but different parent frames and different `n` bindings.

## Lambda Functions

A lambda expression creates a function using the current frame as its parent.

```py
f = lambda x: x + 1
```

```text
Global:
  f -> func lambda(x) [parent=Global]
```

A lambda created inside a call uses that call frame as its parent and can therefore form a closure.

## Rebinding Function Names

```py
def square(x):
    return x * x

f = square
square = 10
result = f(4)
```

`f` still refers to the original function value. Rebinding `square` changes only that binding.

A function's intrinsic name also does not change when another name is bound to it.

## Drawing Procedure

For each line:

1. Identify the statement or outermost expression.
2. Evaluate subexpressions in order.
3. Add or update bindings.
4. Draw a frame for each user-defined call.
5. Set the frame's parent from the function value.
6. Bind parameters and local names.
7. Record the return value before continuing the caller.
8. Reuse one object when several names are aliases to that same object.

## Common Traps

- Executing a function body during its `def` statement.
- Reusing one local frame for several calls.
- Using the caller's frame as the callee's parent.
- Connecting recursive frames as lexical parents.
- Looking in Global before the current frame.
- Forgetting local shadowing.
- Binding assignment before evaluating the right side.
- Confusing a function's intrinsic name with a name bound to it.
- Forgetting that a lambda has a parent frame.
- Assuming a returned function loses access to its defining frame.
- Drawing two objects when two names are aliases to the same object.
