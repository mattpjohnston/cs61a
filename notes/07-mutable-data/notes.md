# 07 - Mutable Data, Iterators, and Generators

Reading: Composing Programs 2.4, 4.2

## Mutation

A mutable object can change while keeping the same identity.

Mutable:

- lists
- dictionaries
- iterators and generators (their state advances)

Immutable:

- numbers
- booleans
- strings
- tuples
- ranges

```py
s = [1, 2]
s.append(3)       # changes the existing list

word = 'cat'
word = word + 's' # creates a new string and rebinds word
```

## Mutation vs Rebinding

Mutation changes an object. Assignment changes what a name refers to.

```py
x = [1, 2]
y = x
x.append(3)
# x and y both refer to [1, 2, 3]
```

```py
x = [1, 2]
y = x
x = x + [3]
# x refers to [1, 2, 3]; y still refers to [1, 2]
```

Function parameters are local names. Rebinding a parameter does not mutate the caller's object.

## Identity, Equality, and Aliasing

```py
x == y  # equal values or contents
x is y  # same object
```

```py
x = [1, 2]
y = [1, 2]
z = x

x == y  # True
x is y  # False
x is z  # True
```

Aliasing means several names or container positions refer to the same mutable object. Mutation through any alias is visible through all of them.

Use `is` mainly for identity questions such as `value is None`, not ordinary number or string comparison.

## List Mutation

```py
s[i] = value             # replace one element
s[a:b] = replacement     # replace a slice in place
s.append(x)              # add one element
s.extend(iterable)        # add every produced element
s.insert(i, x)            # insert before index i
s.remove(x)               # remove first equal x
s.pop(i)                  # remove and return element i
s.pop()                   # remove and return last element
```

Most mutation methods return `None`. `pop` is the important exception.

```py
s = [1, 2]
result = s.append(3)
# s is [1, 2, 3]; result is None
```

`append([3, 4])` adds one nested list. `extend([3, 4])` adds two elements.

Full-slice assignment changes contents while preserving identity.

```py
s = [1, 2]
alias = s
s[:] = [8, 9]
# alias is s and also sees [8, 9]
```

## Shallow Copies

```py
copy = s[:]
copy = list(s)
copy = s.copy()
```

These create a new outer list but share nested mutable elements.

```py
s = [[1], [2]]
t = s[:]
t[0].append(9)
# both see the shared inner list [1, 9]
```

## Mutating While Iterating

Changing a list while directly iterating over it can skip or revisit elements.

Safer options:

```py
for item in items[:]:
    pass  # mutate items safely here
```

```py
while target in items:
    items.remove(target)
```

Or use a carefully controlled index.

Two parameters may alias the same list. HW03's inventory and items can be the same object, so copy the traversal sequence before mutating the inventory.

```py
items_to_process = items[:]
for item in items_to_process:
    while item in inventory:
        inventory.remove(item)
    inventory.append(item)
```

An in-place result should preserve identity:

```py
result is inventory
```

## Dictionaries

A dictionary is a mutable mapping from unique, hashable keys to values.

```py
d = {'a': 1}
d['a'] = 2       # update
d['b'] = 3       # add
d.get('c', 0)    # 0
d.pop('a')       # remove and return 2
```

Iterating over a dictionary produces keys. Use `.items()` for key-value pairs.

```py
for key, value in d.items():
    ...
```

Do not add or remove keys while iterating over the same dictionary; iterate over `list(d)` if structural mutation is required.

## Iterable vs Iterator

An **iterable** can produce an iterator with `iter`.

An **iterator** produces values one at a time with `next` and remembers its position.

```py
values = [10, 20, 30]
it = iter(values)
next(it)  # 10
next(it)  # 20
```

Key rules:

- Lists, strings, tuples, ranges, and dictionaries are iterable.
- A list is not itself an iterator: `next([1, 2])` raises `TypeError`.
- Every iterator is iterable.
- `iter(iterator) is iterator`.
- Two iterators from the same list have independent positions.
- Two names for the same iterator share one position.
- Exhaustion raises `StopIteration`.
- Iterators are single-pass; an exhausted iterator stays exhausted.

```py
next(it, default)
```

The two-argument form returns `default` instead of raising `StopIteration`.

## How `for` Works

Conceptually:

```py
iterator = iter(iterable)
while True:
    try:
        item = next(iterator)
    except StopIteration:
        break
    process(item)
```

A loop over a list gets a fresh iterator. A loop over an already partly consumed iterator starts at its current position.

`map`, `filter`, `zip`, and `reversed` return lazy, single-pass iterators.

## Generator Functions

A function containing `yield` is a generator function.

```py
def countdown(n):
    while n >= 0:
        yield n
        n -= 1
```

Calling `countdown(3)` creates a generator without running the body.

Each `next`:

1. starts or resumes execution
2. runs until `yield`
3. returns the yielded value
4. pauses while preserving local state

Reaching the end or executing `return` raises `StopIteration` for the consumer.

Each call creates an independent generator. Aliases to one generator share its state. To restart, call the generator function again.

## `yield from`

```py
def generate(values):
    yield from values
```

For this course, it is equivalent to:

```py
def generate(values):
    for value in values:
        yield value
```

It forwards values lazily and is useful with recursive generators.

## Finite and Infinite Generators

Generators compute values only when requested, so they can represent infinite sequences.

```py
def naturals():
    n = 1
    while True:
        yield n
        n += 1
```

Never convert an infinite generator to a list. Request only finitely many values.

## Lazy Merge Pattern

To merge two strictly increasing iterables:

- keep one current value from each iterator
- yield the smaller one and advance only that iterator
- if equal, yield once and advance both
- handle exhaustion without materializing either input

This allows finite or infinite inputs and uses constant extra space.

## Recursive Generator Paths

```py
def yield_paths(t, target):
    if label(t) == target:
        yield [label(t)]
    for branch in branches(t):
        for path in yield_paths(branch, target):
            yield [label(t)] + path
```

The recursive call yields paths from a branch to matching nodes. The current call prepends its root label.

Continue searching below a match because descendants may also match.

## Common Traps

- Rebinding a name when the contract requires in-place mutation.
- Assuming assignment copies a list.
- Assuming a shallow copy duplicates nested lists.
- Writing `s = s.append(x)` and rebinding `s` to `None`.
- Confusing `append` with `extend`.
- Mutating a list while directly iterating over it.
- Forgetting that two arguments may be aliases.
- Using `==` when identity is the requirement.
- Calling `next` on an iterable rather than an iterator.
- Expecting `iter(iterator)` to reset it.
- Reusing an exhausted iterator.
- Forgetting that generator bodies do not run when called.
- Returning a recursive generator instead of yielding from it.
- Calling `list` on an infinite generator.
- Advancing both merge iterators when their values are different.
