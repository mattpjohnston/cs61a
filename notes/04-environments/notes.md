# 04 - Environments

Reading: Composing Programs 1.6

## Main Ideas

An environment is the context in which expressions are evaluated.

Names do not have meaning by themselves. A name has meaning because an environment contains a binding for that name.

```py
x
```

This expression only has a value if `x` is bound in the environment where it is evaluated.

Environment diagrams are a way to track:

- which names are bound
- what values they are bound to
- which frames exist
- which frame is used for name lookup
- which parent frame a function or call uses

They are especially important for higher-order functions, nested functions, lambdas, and closures.

## Environment Diagrams

An environment diagram is a model of program execution.

It does not show every small arithmetic step. It focuses on names, values, functions, frames, and return values.

Example:

```py
x = 3
y = x + 2
```

Diagram idea:

```text
Global frame:
  x -> 3
  y -> 5
```

The diagram records the final bindings created by the assignment statements.

## Frames

A frame is a table of name-value bindings.

Each binding connects one name to one value.

```text
name -> value
```

Example:

```text
Global frame:
  pi -> 3.14159
  radius -> 10
  area -> 314.159
```

Frames are where names are looked up.

A name can appear in more than one frame. The value of the name depends on which environment is being used.

## Global Frame

Every environment diagram starts with the global frame.

Top-level assignment statements create bindings in the global frame.

```py
x = 1
y = 2
```

Diagram:

```text
Global frame:
  x -> 1
  y -> 2
```

Top-level `def` statements also create bindings in the global frame.

```py
def square(x):
    return x * x
```

Diagram:

```text
Global frame:
  square -> func square(x) [parent=Global]
```

The global frame does not come from calling a function. It is the starting frame for the program.

## What Changes an Environment Diagram

In early CS61A Python, the main things that change an environment diagram are:

- assignment statements
- `def` statements
- call expressions for user-defined functions
- return values from function calls

Arithmetic expressions by themselves do not add bindings.

```py
2 + 3
```

This evaluates to `5`, but it does not create a name.

Expression statements usually do not change the diagram unless they have side effects or call user-defined functions.

```py
square(4)
```

This creates a call frame for `square`, but if the result is not assigned to a name, the global frame does not get a new binding.

## Assignment Statements

Assignment evaluates the right side first, then binds the name on the left.

```py
x = 3
y = x + 4
```

Trace:

```text
x = 3:
  bind x -> 3 in Global

y = x + 4:
  look up x -> 3
  evaluate x + 4 -> 7
  bind y -> 7 in Global
```

Final diagram:

```text
Global frame:
  x -> 3
  y -> 7
```

Assignment updates the current frame.

If a name already exists in the current frame, assignment rebinds it.

```py
x = 3
x = x + 1
```

Final diagram:

```text
Global frame:
  x -> 4
```

## Multiple Assignment

In multiple assignment, all right-side expressions are evaluated before any left-side names are rebound.

```py
x, y = 1, 2
x, y = y, x + y
```

Trace for the second assignment:

```text
evaluate right side:
  y -> 2
  x + y -> 1 + 2 -> 3

bind left side:
  x -> 2
  y -> 3
```

Final diagram:

```text
Global frame:
  x -> 2
  y -> 3
```

This rule matters when tracing loops and state updates.

## Def Statements

A `def` statement creates a function value and binds a name to it.

```py
def square(x):
    return x * x
```

Diagram:

```text
Global frame:
  square -> func square(x) [parent=Global]
```

The function body is not executed when the `def` statement is executed.

Only the function value is created.

```py
def bad(x):
    return 1 / 0
```

This definition does not error immediately.

The error happens only if `bad` is called.

## Function Values

In environment diagrams, user-defined functions are shown as values.

They include:

- intrinsic name
- formal parameters
- parent frame

Example:

```text
func square(x) [parent=Global]
```

The intrinsic name is the name written in the `def` statement.

```py
def square(x):
    return x * x
```

If the function is rebound to another name, the intrinsic name does not change.

```py
f = square
```

Diagram:

```text
Global frame:
  square -> func square(x) [parent=Global]
  f ------^
```

Both names point to the same function value.

The function's intrinsic name is still `square`.

## Parent Frames

Every user-defined function has a parent frame.

The parent frame is the frame in which the function was defined.

For a top-level `def`, the parent is the global frame.

```py
def square(x):
    return x * x
```

Function value:

```text
func square(x) [parent=Global]
```

For a nested `def`, the parent is the local frame where the nested `def` statement was executed.

```py
def outer(x):
    def inner(y):
        return x + y
    return inner
```

When `outer(3)` is called, the `inner` function is created inside the `outer` frame.

```text
func inner(y) [parent=f1]
```

where `f1` is the frame for that call to `outer`.

## Call Expressions

A call expression applies a function to argument values.

```py
square(4)
```

Evaluation rule:

1. Evaluate the operator expression.
2. Evaluate the operand expressions.
3. Apply the function value to the argument values.

For a user-defined function, applying the function creates a new frame.

```py
def square(x):
    return x * x

square(4)
```

Call frame:

```text
f1: square [parent=Global]
  x -> 4
  return value -> 16
```

## Calling User-Defined Functions

When a user-defined function is called:

1. Create a new local frame.
2. Label it with the function's intrinsic name.
3. Set the frame's parent to the function's parent frame.
4. Bind formal parameters to argument values in the local frame.
5. Execute the function body in the environment that starts with the local frame.
6. Record the return value when a `return` statement is executed.

Example:

```py
def double(x):
    return x + x

result = double(5)
```

Diagram:

```text
Global frame:
  double -> func double(x) [parent=Global]
  result -> 10

f1: double [parent=Global]
  x -> 5
  return value -> 10
```

The call frame remains useful in the diagram even after the call has returned, because it explains how the result was produced.

## Built-In Functions

CS61A environment diagrams usually do not draw frames for built-in functions.

```py
abs(-3)
```

This returns `3`, but no `abs` frame is drawn.

User-defined function calls do create frames.

```py
def my_abs(x):
    if x < 0:
        return -x
    return x

my_abs(-3)
```

This creates a frame for `my_abs`.

The distinction keeps diagrams focused on the functions whose bodies we are tracing.

## Return Values

A return statement finishes the current function call and supplies a value to the call expression.

```py
def square(x):
    return x * x

y = square(4)
```

Trace:

```text
call square(4)
create frame:
  x -> 4
evaluate return expression:
  x * x -> 16
return 16
bind y -> 16 in Global
```

Diagram:

```text
f1: square [parent=Global]
  x -> 4
  return value -> 16
```

The return value is not automatically bound to a name.

It is only bound if the surrounding code assigns it.

```py
square(4)
```

This call returns `16`, but no global name receives it.

## Name Lookup

To evaluate a name, Python searches frames in order.

Name lookup rule:

1. Look in the current frame.
2. If the name is not found, look in the parent frame.
3. Continue through parent frames until reaching the global frame.
4. If the name is not found, raise a `NameError`.

Example:

```py
x = 10

def f(y):
    return x + y
```

Calling `f(3)`:

```text
f frame:
  y -> 3
parent -> Global:
  x -> 10
```

The name `y` is found locally.

The name `x` is found in the parent frame.

## Local Names

Formal parameters are local names.

```py
x = 10

def f(x):
    return x + 1

f(3)
```

Frame:

```text
Global frame:
  x -> 10
  f -> func f(x) [parent=Global]

f1: f [parent=Global]
  x -> 3
  return value -> 4
```

Inside `f`, the name `x` refers to the local parameter `x`, not the global `x`.

The local binding shadows the global binding.

The global `x` is unchanged.

## Same Name in Different Frames

The same name can have different values in different frames.

```py
def double(double):
    return double + double

result = double(2)
```

Diagram:

```text
Global frame:
  double -> func double(double) [parent=Global]
  result -> 4

f1: double [parent=Global]
  double -> 2
  return value -> 4
```

In the global frame, `double` is a function.

In the local frame, `double` is the argument value `2`.

The local name is found first when the body is evaluated.

This code is legal, but it is intentionally confusing.

## Assignment Inside Functions

Assignment inside a function binds names in the current local frame.

```py
x = 10

def f(y):
    x = y + 1
    return x

result = f(3)
```

Diagram:

```text
Global frame:
  x -> 10
  f -> func f(y) [parent=Global]
  result -> 4

f1: f [parent=Global]
  y -> 3
  x -> 4
  return value -> 4
```

The assignment `x = y + 1` creates or updates `x` in the local frame.

It does not change the global `x`.

## Multiple Calls Create Multiple Frames

Each call to a user-defined function creates a new frame.

```py
def square(x):
    return x * x

a = square(2)
b = square(5)
```

Diagram:

```text
Global frame:
  square -> func square(x) [parent=Global]
  a -> 4
  b -> 25

f1: square [parent=Global]
  x -> 2
  return value -> 4

f2: square [parent=Global]
  x -> 5
  return value -> 25
```

The two local `x` bindings are separate.

Changing or using `x` in one call does not affect `x` in another call.

## Nested Call Expressions

Nested call expressions are evaluated from the inside out as required by the call expression rule.

```py
def double(x):
    return x * 2

result = double(double(2))
```

Trace:

```text
evaluate outer operator:
  double -> function

evaluate outer operand:
  double(2)
```

The inner call happens first.

```text
f1: double [parent=Global]
  x -> 2
  return value -> 4
```

Then the outer call uses the returned value `4`.

```text
f2: double [parent=Global]
  x -> 4
  return value -> 8
```

Final global binding:

```text
result -> 8
```

Both frames have parent `Global` because the function `double` was defined in the global frame.

## Higher-Order Function Frames

Functions can be argument values.

```py
def apply_twice(f, x):
    return f(f(x))

def square(x):
    return x * x

result = apply_twice(square, 2)
```

When `apply_twice` is called:

```text
f1: apply_twice [parent=Global]
  f -> func square(x) [parent=Global]
  x -> 2
```

Inside the body:

```py
return f(f(x))
```

The inner call is:

```text
f(x) -> square(2) -> 4
```

The outer call is:

```text
f(4) -> square(4) -> 16
```

Final result:

```text
result -> 16
```

The parameter `f` is just a local name bound to a function value.

## Passing a Function vs Calling It

In an environment diagram, these create different bindings.

```py
apply_twice(square, 2)
```

The name `square` evaluates to the function value.

```text
f -> func square(x) [parent=Global]
```

But:

```py
apply_twice(square(2), 2)
```

evaluates `square(2)` first.

```text
square(2) -> 4
```

Then:

```text
f -> 4
```

When the body evaluates `f(f(x))`, Python tries to call `4`.

That causes a `TypeError`.

## Nested Def Statements

A `def` statement can appear inside another function body.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The nested `def adder(k)` is not executed when `make_adder` is defined.

It is executed when `make_adder` is called.

```py
add_three = make_adder(3)
```

Trace:

```text
create make_adder frame:
  n -> 3

execute def adder(k):
  create func adder(k) [parent=f1]
  bind adder -> that function in f1

return adder
bind add_three -> returned function in Global
```

The name `adder` is local to the `make_adder` frame.

The returned function value can still be bound globally.

## Lexical Scope

Lexical scope means that a function's parent is determined by where the function is defined.

It is not determined by where the function is called.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder

add_three = make_adder(3)
result = add_three(4)
```

When `adder` is called, its frame's parent is the frame where `adder` was defined.

```text
f2: adder [parent=f1]
  k -> 4

f1: make_adder [parent=Global]
  n -> 3
  adder -> func adder(k) [parent=f1]
  return value -> func adder(k) [parent=f1]
```

The body of `adder` evaluates:

```py
return n + k
```

Lookup:

```text
k found in f2 -> 4
n not in f2
n found in f1 -> 3
```

Return value:

```text
7
```

## Closures

A closure is a function value together with the environment it needs.

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder
```

The returned `adder` function needs the binding for `n`.

```py
add_three = make_adder(3)
```

The function bound to `add_three` carries its parent frame:

```text
func adder(k) [parent=f1]

f1:
  n -> 3
```

That is why this call works later:

```py
add_three(10)      # 13
```

The original call to `make_adder` has returned, but its frame remains part of the closure because the returned function still refers to it.

## Different Closures

Different calls to the same function can create different closures.

```py
add_three = make_adder(3)
add_five = make_adder(5)
```

Diagram idea:

```text
Global frame:
  add_three -> func adder(k) [parent=f1]
  add_five  -> func adder(k) [parent=f2]

f1: make_adder [parent=Global]
  n -> 3
  adder -> func adder(k) [parent=f1]

f2: make_adder [parent=Global]
  n -> 5
  adder -> func adder(k) [parent=f2]
```

Both returned functions have the same intrinsic name, `adder`.

They have different parent frames.

```py
add_three(2)      # 5
add_five(2)       # 7
```

The difference comes from the parent environment.

## Parent of Function vs Parent of Frame

There are two related parent labels to track.

Function value:

```text
func adder(k) [parent=f1]
```

Call frame:

```text
f2: adder [parent=f1]
```

The rule is:

```text
the parent of a call frame is the parent of the function being called
```

It is not necessarily the frame where the call expression appears.

Example:

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder

add_three = make_adder(3)
result = add_three(4)
```

The call `add_three(4)` appears in the global frame.

But the frame for `adder` has parent `f1`, not `Global`.

That is because the function value bound to `add_three` has parent `f1`.

## Name Lookup Uses the Current Environment

When a function body is executed, name lookup starts in the call frame and follows parent links.

```py
x = 100

def make_adder(n):
    def adder(k):
        return n + k
    return adder

add_three = make_adder(3)
result = add_three(4)
```

The name `n` is found in the `make_adder` frame, not in the global frame.

The global `x` is irrelevant because no expression asks for `x`.

Environment diagrams are about the names that expressions actually evaluate.

## Global Names in Functions

A function defined in the global frame can refer to global names.

```py
x = 3

def multiply_by_x(y):
    return x * y

result = multiply_by_x(4)
```

Call frame:

```text
f1: multiply_by_x [parent=Global]
  y -> 4
```

Lookup:

```text
y found in f1 -> 4
x found in Global -> 3
```

Result:

```text
12
```

If the global binding changes before the function is called, the function sees the new binding.

```py
x = 3

def multiply_by_x(y):
    return x * y

x = 5
result = multiply_by_x(4)
```

Now the result is `20`.

The parent is the global frame itself, not a frozen copy of old global bindings.

## Local Names in Closures

Closures preserve access to the parent frame where the function was defined.

```py
x = 100

def f(x):
    return lambda y: x + y

g = f(2)
result = g(3)
```

The lambda returned by `f(2)` has parent `f1`, where `x -> 2`.

Call to `g(3)`:

```text
f2: lambda [parent=f1]
  y -> 3

f1: f [parent=Global]
  x -> 2
  return value -> func lambda(y) [parent=f1]
```

The global `x -> 100` is shadowed by the `x -> 2` in the parent frame.

Result:

```text
5
```

## Lambda Expressions

A lambda expression creates a function value.

```py
lambda x: x + 1
```

In an environment diagram, a lambda function is usually shown as:

```text
func lambda(x) [parent=<current frame>]
```

or:

```text
func <lambda>(x) [parent=<current frame>]
```

Like a `def` statement, a lambda function has a parent frame.

Unlike a `def` statement, a lambda expression by itself does not bind a name.

```py
lambda x: x + 1
```

This creates a function value and then discards it if nothing uses it.

## Binding Lambda Functions

A lambda can be bound by assignment.

```py
inc = lambda x: x + 1
result = inc(4)
```

Diagram:

```text
Global frame:
  inc -> func lambda(x) [parent=Global]
  result -> 5

f1: lambda [parent=Global]
  x -> 4
  return value -> 5
```

The intrinsic name is `lambda`, not `inc`.

The name `inc` is just the global name bound to the function value.

## Lambda Parent Frames

A lambda expression uses the current frame as its parent.

```py
def make_multiplier(n):
    return lambda k: n * k

times_three = make_multiplier(3)
result = times_three(4)
```

When the lambda is evaluated, the current frame is the frame for `make_multiplier`.

```text
func lambda(k) [parent=f1]
```

Later, when `times_three(4)` is called:

```text
f2: lambda [parent=f1]
  k -> 4

f1: make_multiplier [parent=Global]
  n -> 3
```

The lambda body looks up `n` in its parent frame.

Result:

```text
12
```

## Lambda and Global Rebinding

A lambda defined in the global frame uses the global frame as its parent.

```py
x = 3
f = lambda y: x * y
x = 4
result = f(2)
```

The lambda's parent is `Global`.

When `f(2)` is called, the name `x` is looked up in the global frame.

At that time:

```text
x -> 4
```

So:

```text
result -> 8
```

The lambda did not store the old value `3`.

It stored its parent environment, which is the global frame.

## Zero-Argument Functions

Some functions take no arguments.

```py
def make_counter_message():
    return 'counting'
```

Call:

```py
message = make_counter_message()
```

Call frame:

```text
f1: make_counter_message [parent=Global]
  return value -> 'counting'
```

There are no formal parameters to bind.

The empty parentheses are still required to call the function.

```py
make_counter_message      # function value
make_counter_message()    # returned string
```

Zero-argument functions are useful when a function delays computation until it is called.

## Returned Functions

A function can return another function.

```py
def outer(x):
    def inner(y):
        return x * y
    return inner

twice = outer(2)
result = twice(5)
```

Diagram outline:

```text
Global frame:
  outer -> func outer(x) [parent=Global]
  twice -> func inner(y) [parent=f1]
  result -> 10

f1: outer [parent=Global]
  x -> 2
  inner -> func inner(y) [parent=f1]
  return value -> func inner(y) [parent=f1]

f2: inner [parent=f1]
  y -> 5
  return value -> 10
```

The returned function keeps the parent frame it had when it was created.

## Calling Returned Functions Directly

Returned functions can be called without first assigning them to a name.

```py
result = outer(2)(5)
```

Evaluation:

```text
outer(2) -> func inner(y) [parent=f1]
outer(2)(5) -> call that returned function on 5
```

Diagram still contains two frames:

```text
f1: outer [parent=Global]
  x -> 2
  inner -> func inner(y) [parent=f1]
  return value -> func inner(y) [parent=f1]

f2: inner [parent=f1]
  y -> 5
  return value -> 10
```

The only difference is that the returned function was not bound to a global name like `twice`.

## Currying in Environment Diagrams

Currying creates a chain of returned functions.

```py
def curry2(f):
    def g(x):
        def h(y):
            return f(x, y)
        return h
    return g
```

Call:

```py
pow_curried = curry2(pow)
result = pow_curried(2)(5)
```

Frames:

```text
f1: curry2 [parent=Global]
  f -> built-in pow
  g -> func g(x) [parent=f1]
  return value -> func g(x) [parent=f1]

f2: g [parent=f1]
  x -> 2
  h -> func h(y) [parent=f2]
  return value -> func h(y) [parent=f2]

f3: h [parent=f2]
  y -> 5
  return value -> 32
```

Lookup inside `h`:

```text
y found in f3 -> 5
x found in f2 -> 2
f found in f1 -> pow
```

Then:

```text
pow(2, 5) -> 32
```

## Reverse Function Example

Higher-order functions often create returned functions whose parents matter.

```py
def reverse(f):
    return lambda x, y: f(y, x)
```

Call:

```py
rev_pow = reverse(pow)
result = rev_pow(2, 3)
```

Frames:

```text
f1: reverse [parent=Global]
  f -> built-in pow
  return value -> func lambda(x, y) [parent=f1]

f2: lambda [parent=f1]
  x -> 2
  y -> 3
  return value -> 9
```

The lambda body evaluates:

```py
f(y, x)
```

Lookup:

```text
f found in f1 -> pow
y found in f2 -> 3
x found in f2 -> 2
```

So:

```text
pow(3, 2) -> 9
```

## Decorated Functions in Diagrams

A decorator rebinds a function name to the result of a higher-order function.

```py
def trace(fn):
    def wrapped(x):
        print('call', x)
        return fn(x)
    return wrapped

@trace
def triple(x):
    return 3 * x
```

Equivalent code:

```py
def triple(x):
    return 3 * x

triple = trace(triple)
```

Diagram idea:

```text
Global frame:
  trace -> func trace(fn) [parent=Global]
  triple -> func wrapped(x) [parent=f1]

f1: trace [parent=Global]
  fn -> original func triple(x) [parent=Global]
  wrapped -> func wrapped(x) [parent=f1]
  return value -> func wrapped(x) [parent=f1]
```

Calling `triple(4)` calls `wrapped(4)`.

Inside `wrapped`, the name `fn` is found in the `trace` frame and refers to the original `triple`.

## Drawing Rule for Def

When drawing a `def` statement:

1. Create a function value.
2. Record its intrinsic name and formal parameters.
3. Set its parent to the current frame.
4. Bind the function name to that function value in the current frame.

Example:

```py
def add_one(x):
    return x + 1
```

Diagram:

```text
current frame:
  add_one -> func add_one(x) [parent=current frame]
```

Do not execute the body yet.

The body is saved for later calls.

## Drawing Rule for Assignment

When drawing assignment:

1. Evaluate the expression on the right.
2. Bind the name on the left in the current frame.

Example:

```py
x = 2
y = x + 3
```

Diagram:

```text
current frame:
  x -> 2
  y -> 5
```

If the right side is a call expression, draw the call first.

```py
y = square(3)
```

The call to `square` creates a frame and returns a value.

Then `y` is bound to that return value.

## Drawing Rule for Calls

When drawing a user-defined function call:

1. Evaluate the operator to a function value.
2. Evaluate all operands to argument values.
3. Create a new frame.
4. Label the frame with the function's intrinsic name.
5. Copy the function's parent into the frame's parent label.
6. Bind parameters to argument values.
7. Execute the body in that environment.
8. Record the return value.

Example:

```py
def add(x, y):
    return x + y

result = add(2, 3)
```

Frame:

```text
f1: add [parent=Global]
  x -> 2
  y -> 3
  return value -> 5
```

## Diagram Checklist

Useful checklist when tracing code:

1. Start with the global frame.
2. Execute statements from top to bottom.
3. For assignment, evaluate right side first.
4. For `def`, create a function value with the current frame as parent.
5. For a user-defined call, create a new frame.
6. The call frame's parent is the function value's parent.
7. Bind parameters to arguments in the call frame.
8. Evaluate the body using normal name lookup.
9. Record return values.
10. Continue with the surrounding expression or statement.

Most mistakes come from skipping step 6 or from calling a function before evaluating its operands.

## Key Traces

### Assignment and Def

```py
x = 2

def square(x):
    return x * x

y = square(x)
```

Trace:

```text
Global:
  x -> 2

def square:
  create func square(x) [parent=Global]
  bind square -> function

y = square(x):
  look up square -> function
  look up x -> 2
  call square(2)

f1: square [parent=Global]
  x -> 2
  return value -> 4

Global:
  y -> 4
```

### Local Shadowing

```py
x = 10

def f(x):
    return x + 1

result = f(3)
```

Trace:

```text
Global:
  x -> 10
  f -> func f(x) [parent=Global]

f1: f [parent=Global]
  x -> 3
  return value -> 4

Global:
  result -> 4
```

The local `x` shadows the global `x` during the call.

### Nested Calls

```py
def double(x):
    return x * 2

result = double(double(2))
```

Trace:

```text
call inner double(2):
  f1: double [parent=Global]
    x -> 2
    return value -> 4

call outer double(4):
  f2: double [parent=Global]
    x -> 4
    return value -> 8

Global:
  result -> 8
```

### Higher-Order Function

```py
def apply_twice(f, x):
    return f(f(x))

def square(x):
    return x * x

result = apply_twice(square, 2)
```

Trace:

```text
f1: apply_twice [parent=Global]
  f -> func square(x) [parent=Global]
  x -> 2

inner f(x):
  square(2) -> 4

outer f(4):
  square(4) -> 16

return value -> 16
Global result -> 16
```

### Nested Def

```py
def outer(x):
    def inner(y):
        return x + y
    return inner(4)

result = outer(3)
```

Trace:

```text
f1: outer [parent=Global]
  x -> 3
  inner -> func inner(y) [parent=f1]

call inner(4):
  f2: inner [parent=f1]
    y -> 4
    x found in f1 -> 3
    return value -> 7

outer return value -> 7
Global result -> 7
```

### Returned Function

```py
def make_adder(n):
    def adder(k):
        return n + k
    return adder

add_three = make_adder(3)
result = add_three(4)
```

Trace:

```text
f1: make_adder [parent=Global]
  n -> 3
  adder -> func adder(k) [parent=f1]
  return value -> func adder(k) [parent=f1]

Global:
  add_three -> func adder(k) [parent=f1]

f2: adder [parent=f1]
  k -> 4
  n found in f1 -> 3
  return value -> 7

Global:
  result -> 7
```

### Different Closures

```py
add_three = make_adder(3)
add_five = make_adder(5)

a = add_three(10)
b = add_five(10)
```

Trace:

```text
add_three -> func adder(k) [parent=f1]
f1 has n -> 3

add_five -> func adder(k) [parent=f2]
f2 has n -> 5

add_three(10) -> 13
add_five(10) -> 15
```

The functions have the same code but different parent frames.

### Lambda Closure

```py
def make_multiplier(n):
    return lambda k: n * k

times_three = make_multiplier(3)
result = times_three(5)
```

Trace:

```text
f1: make_multiplier [parent=Global]
  n -> 3
  return value -> func lambda(k) [parent=f1]

Global:
  times_three -> func lambda(k) [parent=f1]

f2: lambda [parent=f1]
  k -> 5
  n found in f1 -> 3
  return value -> 15

Global:
  result -> 15
```

### Global Rebinding

```py
x = 3
f = lambda y: x * y
x = 4
result = f(2)
```

Trace:

```text
Global:
  x -> 3
  f -> func lambda(y) [parent=Global]
  x -> 4

f1: lambda [parent=Global]
  y -> 2
  x found in Global -> 4
  return value -> 8

Global:
  result -> 8
```

The lambda uses the global frame, where `x` is now `4`.

### Currying

```py
def curry2(f):
    def g(x):
        def h(y):
            return f(x, y)
        return h
    return g

pow_curried = curry2(pow)
result = pow_curried(2)(3)
```

Trace:

```text
f1: curry2 [parent=Global]
  f -> pow
  g -> func g(x) [parent=f1]
  return value -> func g(x) [parent=f1]

Global:
  pow_curried -> func g(x) [parent=f1]

f2: g [parent=f1]
  x -> 2
  h -> func h(y) [parent=f2]
  return value -> func h(y) [parent=f2]

f3: h [parent=f2]
  y -> 3
  f found in f1 -> pow
  x found in f2 -> 2
  return value -> 8

Global:
  result -> 8
```

## Common Pitfalls

Using the call site as the parent frame:

```py
add_three = make_adder(3)
result = add_three(4)
```

The call appears in the global frame, but the `adder` frame's parent is the `make_adder` frame where `adder` was defined.

Forgetting that `def` does not execute the body:

```py
def f(x):
    return 1 / 0
```

No error occurs until `f` is called.

Drawing frames for built-in functions:

```py
abs(-3)
pow(2, 5)
```

CS61A diagrams usually do not draw frames for built-ins.

Confusing a function value with a return value:

```py
f = square       # function value
y = square(4)    # return value
```

These bindings point to different kinds of values.

Assuming local assignment changes a global binding:

```py
x = 1

def f():
    x = 2
    return x
```

The `x = 2` assignment creates a local binding in the call frame.

Expecting a nested helper name to exist globally:

```py
def outer():
    def inner():
        return 1
    return inner()

inner()
```

`inner` is only bound in the frame for a call to `outer`.

Forgetting that lambda has a parent frame:

```py
def f(x):
    return lambda y: x + y
```

The returned lambda can still look up `x` because its parent is the frame for `f`.

Thinking closures store only values instead of environments:

```py
x = 3
f = lambda y: x + y
x = 10
f(1)
```

The result is `11`, because `x` is looked up in the global frame when the lambda is called.

Skipping operand evaluation in nested calls:

```py
double(double(2))
```

The inner call must return before the outer call can bind its parameter.

Mixing up intrinsic names and bound names:

```py
def square(x):
    return x * x

f = square
f(3)
```

The call frame is labeled with the function's intrinsic name, `square`, even though the function was called through the name `f`.
