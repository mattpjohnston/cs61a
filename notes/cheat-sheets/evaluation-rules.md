# Evaluation Rules

## Names and Literals

- A literal evaluates to its value.
- A name evaluates to its binding in the first frame where it is found.
- Lookup follows the current frame, then parent frames, then Global, and finally Python's built-in namespace. CS61A environment diagrams usually stop at the Global frame and do not draw the built-in namespace.

## Call Expression

```py
operator(operand_1, operand_2)
```

1. Evaluate the operator.
2. Evaluate operands left to right.
3. Apply the function to the argument values.
4. The call evaluates to the return value.

For a user-defined function:

1. Create a fresh frame.
2. Parent it to the function's parent frame.
3. Bind parameters to arguments.
4. Execute the body.

## Assignment

```py
name = expression
```

Evaluate the entire right side first, then bind the result in the current frame.

For multiple assignment, evaluate every right-side expression before changing any left-side binding.

For item assignment:

```py
s[index] = value
```

Evaluate the right side first, then the target object and index, then mutate the object.

## `def` and Lambda

A `def` statement creates a function, records the current frame as its parent, and binds its name. It does not execute the body.

A lambda expression creates a function with the current frame as parent but does not bind a name.

## Return

A `return` statement evaluates its expression, ends the current call immediately, and supplies the value to the caller.

A function that reaches the end without `return` returns `None`.

`print` displays a side effect and returns `None`.

## Conditionals

Evaluate conditions from top to bottom. Execute only the first true suite; otherwise execute `else` if present.

Common false values:

```py
False, None, 0, '', [], (), {}
```

## Boolean Operators

```py
left and right
```

Return `left` if it is false; otherwise evaluate and return `right`.

```py
left or right
```

Return `left` if it is true; otherwise evaluate and return `right`.

```py
not value
```

Always returns `True` or `False`.

`and` and `or` return operand values, not necessarily booleans.

## Loops

`while` repeatedly evaluates its condition and executes the suite while true.

A `for` loop:

1. evaluates the iterable once
2. calls `iter`
3. repeatedly calls `next`
4. stops on `StopIteration`

## Recursion

Recursion uses ordinary call rules. Every call gets a fresh frame.

Calls descend to a base case, then waiting expressions finish while calls unwind.

## Mutation

- Mutation changes an existing object.
- Assignment rebinds a name.
- `==` compares value; `is` compares identity.
- Most list mutation methods return `None`; `pop` returns the removed value.

## Iterators and Generators

- `iter(iterable)` returns an iterator.
- `next(iterator)` returns and consumes the next value.
- Exhaustion raises `StopIteration`.
- `iter(iterator) is iterator`.

Calling a generator function creates a generator without running its body.

Each `next` resumes execution, runs to `yield`, returns that value, and preserves the suspended state.
