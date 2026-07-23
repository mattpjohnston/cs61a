# Environment Diagrams

## Four Rules

### Assignment

Evaluate the right side, then bind the result in the current frame.

### `def`

Create a function value with:

- intrinsic name
- parameters and body
- current frame as parent

Then bind its name. Do not execute the body.

### User-Defined Call

1. Evaluate operator and operands.
2. Create a fresh local frame.
3. Set its parent to the function's parent.
4. Bind parameters to arguments.
5. Execute the body and record the return value.

### Name Lookup

Search the current frame, then follow parent links. Stop at the first matching binding.

## Basic Example

```py
def square(x):
    return x * x

result = square(4)
```

```text
Global:
  square -> func square(x) [parent=Global]
  result -> 16

f1: square [parent=Global]
  x -> 4
  return value -> 16
```

Every call gets a new frame.

## Parent vs Caller

A frame's parent is determined by **where the function was defined**, not where it was called.

The call stack records waiting calls; parent links control name lookup. Recursive frames are not automatically parents of one another.

## Shadowing

A local binding hides a parent binding with the same name.

```py
x = 10

def f(x):
    return x + 1

f(3)  # local x is 3
```

Assignment inside a function normally creates or changes a local binding, not a global one.

## Higher-Order Functions

A parameter can point to a function value.

```py
def apply_twice(f, x):
    return f(f(x))
```

Calls to `f` use the parent stored on that function value, not the `apply_twice` frame.

## Closures

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The returned `adder` keeps the `make_adder` frame as its parent, so it can still find `n`.

Different calls to `make_adder` create different parent frames and therefore different closures.

## Mutable Objects

Draw one object with multiple arrows when names alias it.

```py
s = [1, 2]
t = s
t.append(3)
```

```text
s --\
     -> list [1, 2, 3]
t --/
```

Mutation changes the shared object. Rebinding moves only one name's arrow.

```py
s = s + [4]
```

Now `s` points to a new list while `t` still points to the old one.

## Shallow Copies

A shallow copy creates a new outer object but reuses references to nested objects.

```py
inner = [1]
a = [inner]
b = a[:]
```

`a is not b`, but `a[0] is b[0]`.

## Quick Checklist

For every `def`:

- function created
- name bound
- parent recorded
- body not run

For every call:

- fresh frame
- correct parent
- parameters bound
- return value recorded

For mutable data:

- mutation changes the object
- rebinding moves one arrow
- aliases share one object
- equal objects are not necessarily identical
