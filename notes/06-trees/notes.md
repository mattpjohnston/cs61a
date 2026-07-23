# 06 - Sequences and Trees

Reading: Composing Programs 2.2, 2.3

## Data Abstraction

Data abstraction separates **what a value does** from **how it is represented**.

- A **constructor** creates an abstract value.
- A **selector** retrieves one of its parts.
- A **predicate** answers a question about it.

```py
tree(3, [tree(4), tree(5)])  # constructor
label(t)                      # selector
branches(t)                   # selector
is_leaf(t)                    # predicate
```

Client code should use the public operations, not the concrete representation. This is the abstraction barrier.

```py
label(t)       # good
t[0]           # representation-dependent
```

A representation can change without breaking client code that respects the barrier.

## Sequence Abstraction

Lists, tuples, ranges, and strings are ordered sequences. Common operations:

```py
len(s)
s[index]
s[start:stop]
x in s
for x in s: ...
```

Key rules:

- Indices start at `0`.
- Negative indices count from the end.
- Slice start is included; stop is excluded.
- Slicing creates a new sequence.
- `+` concatenates compatible sequences.
- `*` repeats a sequence.
- String membership searches for substrings; list and tuple membership test elements.

```py
s = [10, 20, 30, 40]
s[1]       # 20
s[-1]      # 40
s[1:3]     # [20, 30]
```

## Main Sequence Types

- `list`: mutable, general-purpose sequence.
- `tuple`: immutable sequence, often used for fixed groups.
- `range`: compact arithmetic sequence; stop is excluded.
- `str`: immutable sequence of characters.

```py
list(range(2, 7, 2))  # [2, 4, 6]
```

Sequence unpacking binds several names at once.

```py
x, y = (3, 4)
for key, value in pairs:
    ...
```

The number of names must match the number of unpacked elements.

## Sequence Processing

A `for` loop processes elements directly.

```py
def total(s):
    result = 0
    for x in s:
        result += x
    return result
```

A list comprehension constructs a new list.

```py
[x * x for x in range(6) if x % 2 == 0]
# [0, 4, 16]
```

Read it as:

```text
for each x in the iterable
if the condition is true
collect the mapped expression
```

Common operations:

- map: transform every element
- filter: keep selected elements
- aggregate: combine elements into one result

Nested list repetition can create aliases:

```py
rows = [[0] * 2] * 3  # same inner list repeated
rows = [[0] * 2 for _ in range(3)]  # independent rows
```

## Tree Vocabulary

A tree has:

- one root label
- zero or more branches
- branches that are themselves trees
- leaves: nodes with no branches
- paths: sequences of nodes connected parent-to-child
- depth: distance from the root
- height: maximum depth below a node

The recursive fact to remember:

> Every branch is a complete smaller tree.

## Tree ADT

```py
def tree(root_label, branches=[]):
    for branch in branches:
        assert is_tree(branch)
    return [root_label] + list(branches)


def label(t):
    return t[0]


def branches(t):
    return t[1:]


def is_leaf(t):
    return not branches(t)
```

The list implementation is below the abstraction barrier. Tree algorithms should use `tree`, `label`, `branches`, and `is_leaf`.

Although the representation is a mutable list, the course tree interface is treated as immutable: build and return a new tree rather than mutating `t`.

```py
def map_tree(fn, t):
    return tree(fn(label(t)), [map_tree(fn, b) for b in branches(t)])
```

## General Tree-Recursion Pattern

```py
def process_tree(t):
    branch_results = [process_tree(b) for b in branches(t)]
    return combine(label(t), branch_results)
```

Ask:

1. What does the function return for one whole tree?
2. What does one recursive call return for a branch?
3. How should the root and branch results be combined?
4. Does an empty branch list already give the correct leaf behavior?

## Essential Tree Patterns

### Size

```py
def size(t):
    return 1 + sum(size(b) for b in branches(t))
```

The `1` counts the current node. `sum([]) == 0`, so leaves work automatically.

### Search

```py
def contains(t, target):
    if label(t) == target:
        return True
    return any(contains(b, target) for b in branches(t))
```

Check the root, then every branch. Do not return `False` after only the first branch.

### Height

Using edge height, a leaf has height `0`.

```py
def height(t):
    if is_leaf(t):
        return 0
    return 1 + max(height(b) for b in branches(t))
```

The explicit leaf case avoids `max` on an empty sequence.

### Maximum Root-to-Leaf Sum

```py
def max_path_sum(t):
    if is_leaf(t):
        return label(t)
    return label(t) + max(max_path_sum(b) for b in branches(t))
```

Choose one branch with `max`; do not sum several different paths.

### Copy or Transform

```py
def copy_tree(t):
    return tree(label(t), [copy_tree(b) for b in branches(t)])
```

```py
def map_tree(fn, t):
    return tree(fn(label(t)), [map_tree(fn, b) for b in branches(t)])
```

### Paths

A recursive call returns paths relative to a branch. Prepend the current label.

```py
def paths(t, target):
    result = []
    if label(t) == target:
        result.append([label(t)])
    for branch in branches(t):
        for path in paths(branch, target):
            result.append([label(t)] + path)
    return result
```

Continue searching below a matching node because descendants may also match.

## HW03 `make_path` Idea

For a requested path `p`:

1. Assert `p[0] == label(t)`.
2. If only one label remains, return `t`.
3. Recurse into branches matching the next label.
4. If none match, append a newly constructed one-branch path.
5. Return a tree built with `tree`, preserving all existing branches.

Do not mutate or index the concrete tree representation directly.

## Common Traps

- Using `t[0]` or `t[1:]` instead of selectors.
- Constructing raw lists instead of calling `tree`.
- Forgetting that each branch is a tree, not a label.
- Returning inside a branch loop and checking only the first branch.
- Forgetting to count the root.
- Calling `max` on an empty branch sequence.
- Summing branches when a path should choose one branch.
- Transforming only the root instead of every subtree.
- Returning the wrong type from a base case.
- Mutating the list-backed representation of an abstract tree.
