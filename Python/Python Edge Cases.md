# Python Edge Cases & Gotchas: Complete Reference

A comprehensive guide to Python's most surprising, counterintuitive, and tricky behaviors — with runnable examples for every case.

---

## Table of Contents

1. [Scope, Binding, and Variables](#1-scope-binding-and-variables)
2. [Mutability and Object References](#2-mutability-and-object-references)
3. [Data Structures and Iteration](#3-data-structures-and-iteration)
4. [Syntax Quirks and Core Execution](#4-syntax-quirks-and-core-execution)
5. [Operators and Comparisons](#5-operators-and-comparisons)
6. [Classes and OOP](#6-classes-and-oop)
7. [Strings and Encoding](#7-strings-and-encoding)
8. [Exception Handling](#8-exception-handling)
9. [Imports and Modules](#9-imports-and-modules)
10. [Type System Surprises](#10-type-system-surprises)

---

## 1. Scope, Binding, and Variables

### 1.1 Late Binding in Closures

Inner functions don't capture the *value* of a variable — they capture a *reference* to it. The variable is only looked up when the function is actually called.

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])  # [2, 2, 2] — NOT [0, 1, 2]

# Fix: capture the value at definition time using a default argument
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])  # [0, 1, 2]
```

### 1.2 The UnboundLocalError Trap

If Python sees *any* assignment to a variable inside a function, it treats that variable as local for the **entire** function — even lines before the assignment.

```python
x = 10

def f():
    print(x)  # UnboundLocalError: local variable 'x' referenced before assignment
    x = 20

f()

# Fix: use global keyword, or restructure to avoid reading before assigning
def f_fixed():
    global x
    print(x)  # 10
    x = 20
```

### 1.3 Walrus Operator (`:=`) Scope Leakage

List comprehension variables are scoped to the comprehension and don't leak. But variables assigned with `:=` inside a comprehension **do** leak into the enclosing scope.

```python
# Regular loop variable: does NOT leak
result = [z for z in range(3)]
print(z)  # NameError: name 'z' is not defined

# Walrus operator: DOES leak
result = [y := i for i in range(3)]
print(y)  # 2  — y leaked into the surrounding scope
```

### 1.4 Closure State Modification with `nonlocal`

To modify a variable in an enclosing (non-global) scope from inside a nested function, you must declare it `nonlocal`. Without it, assignment creates a new local variable.

```python
def make_counter():
    count = 0
    def increment():
        count += 1  # UnboundLocalError without nonlocal
        return count
    return increment

def make_counter_fixed():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter = make_counter_fixed()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

### 1.5 Global Variable Shadowing in Comprehensions

Comprehensions in Python 3 have their own scope. A variable inside one won't accidentally overwrite an outer variable with the same name.

```python
x = 99
result = [x for x in range(5)]  # This x is local to the comprehension
print(x)       # 99 — outer x is unchanged
print(result)  # [0, 1, 2, 3, 4]
```

---

## 2. Mutability and Object References

### 2.1 Mutable Default Arguments

Default argument values are evaluated **once** at function definition time. A mutable default (list, dict, set) retains changes across calls.

```python
def append_item(item, lst=[]):
    lst.append(item)
    return lst

print(append_item(1))  # [1]
print(append_item(2))  # [1, 2]  — NOT [2]!
print(append_item(3))  # [1, 2, 3]

# Fix: use None as the default and create a new object inside
def append_item_fixed(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### 2.2 Shallow Copying via List Multiplication

Multiplying a list of mutable objects copies the **reference**, not the object. All elements point to the same underlying object.

```python
a = [[]] * 3
a[0].append(1)
print(a)  # [[1], [1], [1]] — all three are the same list!

# Fix: use a list comprehension to create independent objects
b = [[] for _ in range(3)]
b[0].append(1)
print(b)  # [[1], [], []]
```

### 2.3 `dict.fromkeys()` Mutable Default Trap

Every key shares the **same** mutable default value object.

```python
d = dict.fromkeys(['a', 'b', 'c'], [])
d['a'].append(1)
print(d)  # {'a': [1], 'b': [1], 'c': [1]}

# Fix: use a dict comprehension
d_fixed = {k: [] for k in ['a', 'b', 'c']}
d_fixed['a'].append(1)
print(d_fixed)  # {'a': [1], 'b': [], 'c': []}
```

### 2.4 Mutating a List Inside a Tuple

You cannot reassign an element of a tuple, but you *can* mutate a mutable object stored inside one. The `+=` operator on a list-in-tuple is a special trap: it mutates the list **and** then raises a `TypeError`.

```python
t = (1, [2, 3])
t[1].append(4)    # Works fine — mutates in-place
print(t)          # (1, [2, 3, 4])

t[1] += [5]       # Mutates the list, THEN raises TypeError!
# TypeError: 'tuple' object does not support item assignment
print(t)          # (1, [2, 3, 4, 5]) — the mutation already happened!
```

### 2.5 Copy vs Deep Copy

Assigning an object creates another reference, not a copy. `copy.copy()` creates a shallow copy; `copy.deepcopy()` is needed for full independence.

```python
import copy

original = [[1, 2], [3, 4]]

reference = original          # Same object
shallow   = copy.copy(original)
deep      = copy.deepcopy(original)

original[0].append(99)

print(reference)  # [[1, 2, 99], [3, 4]]  — same object
print(shallow)    # [[1, 2, 99], [3, 4]]  — inner lists shared!
print(deep)       # [[1, 2], [3, 4]]      — fully independent
```

---

## 3. Data Structures and Iteration

### 3.1 Modifying a List During Iteration

Removing elements while iterating shifts the internal index, causing elements to be skipped.

```python
a = [1, 2, 3, 4]
for x in a:
    if x % 2 == 0:
        a.remove(x)
print(a)  # [1, 3, 4] — 4 was skipped!

# Fix: iterate over a copy
a = [1, 2, 3, 4]
for x in a[:]:
    if x % 2 == 0:
        a.remove(x)
print(a)  # [1, 3]

# Or use a list comprehension
a = [x for x in [1, 2, 3, 4] if x % 2 != 0]
print(a)  # [1, 3]
```

### 3.2 Generator Exhaustion

Generators are single-use. Once consumed, they yield nothing.

```python
g = (x * 2 for x in range(5))

print(list(g))  # [0, 2, 4, 6, 8]
print(list(g))  # []  — exhausted!

# Fix: use a list if you need to iterate multiple times
data = [x * 2 for x in range(5)]
print(data)  # [0, 2, 4, 6, 8]
print(data)  # [0, 2, 4, 6, 8]  — still there
```

### 3.3 Dictionary Key Equivalence (`1`, `True`, `1.0`)

Dictionary keys are based on hash and equality. Since `1 == True == 1.0` and they share the same hash, they are all treated as the **same** key.

```python
d = {1: 'int', True: 'bool', 1.0: 'float'}
print(d)        # {1: 'float'}  — only one entry, last value wins
print(len(d))   # 1

# The key is whatever was inserted first
d2 = {True: 'bool', 1: 'int'}
print(d2)  # {True: 'int'}  — key stays as True, value updated
```

### 3.4 Hashability of Tuples

A tuple is only hashable if *all* of its elements are hashable. A tuple containing a list cannot be used as a dict key or set member.

```python
good = (1, 2, 3)
bad  = (1, [2, 3])

s = {good}  # Works fine
s = {bad}   # TypeError: unhashable type: 'list'

d = {good: 'ok'}   # Works
d = {bad: 'fail'}  # TypeError
```

### 3.5 Out-of-Bounds Slicing vs Indexing

Direct index access raises `IndexError` on out-of-bounds. Slicing never raises — it just returns an empty list (or a partial one).

```python
a = [1, 2, 3]

print(a[10])     # IndexError: list index out of range
print(a[10:20])  # []   — no error
print(a[1:100])  # [2, 3] — truncates gracefully
```

### 3.6 Slice Assignment Resizing

Assigning to a slice doesn't require the replacement to be the same length. Python will resize the list.

```python
a = [1, 2, 3, 4, 5]
a[1:3] = [10, 20, 30, 40]  # Replace 2 elements with 4
print(a)  # [1, 10, 20, 30, 40, 4, 5]

a[1:5] = []  # Delete elements by assigning empty list
print(a)     # [1, 4, 5]
```

### 3.7 `extend()` Unpacks Strings Character by Character

`list.extend()` takes any iterable — and strings are iterables of characters.

```python
a = [1, 2]
a.extend('hello')
print(a)  # [1, 2, 'h', 'e', 'l', 'l', 'o']

# vs append, which adds the whole string as one element
b = [1, 2]
b.append('hello')
print(b)  # [1, 2, 'hello']
```

### 3.8 Set Operations on Mixed Types

Sets use hashing and equality just like dict keys. This means `{1, True, 1.0}` collapses to one element.

```python
s = {1, True, 1.0, 2}
print(s)      # {1, 2}  — only two elements
print(len(s)) # 2
```

### 3.9 Dictionary Ordering and `popitem()`

Since Python 3.7, dicts maintain insertion order. `popitem()` removes and returns the **last** inserted item (LIFO), not a random one.

```python
d = {'a': 1, 'b': 2, 'c': 3}
print(d.popitem())  # ('c', 3) — last inserted
print(d.popitem())  # ('b', 2)
```

---

## 4. Syntax Quirks and Core Execution

### 4.1 Chained Comparisons

Python chained comparisons use an implicit `and`. Each pair is evaluated independently and then combined.

```python
print(1 < 2 < 3)           # True  → (1 < 2) and (2 < 3)
print(False == False in [False])  # True → (False == False) and (False in [False])
print(1 == 1.0 == True)    # True  → all comparisons hold
print(1 < 3 > 2)           # True  → (1 < 3) and (3 > 2)
```

### 4.2 Small Integer Caching (Interning)

CPython caches integers from **-5 to 256**. Variables holding these values point to the same object in memory. Integers outside this range are not guaranteed to be the same object.

```python
a = 256
b = 256
print(a is b)  # True — cached

a = 257
b = 257
print(a is b)  # False — new objects (in most contexts)

# Note: always use == for value equality, never 'is'
print(a == b)  # True
```

### 4.3 `finally` Block Overrides `return`

If both a `try` block and a `finally` block contain `return` statements, the `finally` return always wins.

```python
def f():
    try:
        return 'try'
    finally:
        return 'finally'

print(f())  # 'finally'

# The same applies to exceptions — finally can suppress them!
def g():
    try:
        raise ValueError("oops")
    finally:
        return 'suppressed'  # Exception is silently swallowed!

print(g())  # 'suppressed' — no exception raised
```

### 4.4 Decorator Execution at Import Time

Decorators run the moment the `def` statement is executed (i.e., at import/definition time), not when the decorated function is called.

```python
def my_decorator(func):
    print(f"Decorating {func.__name__}")  # Runs at definition time
    return func

@my_decorator
def greet():
    print("Hello!")

# Output when the module is loaded:
# Decorating greet

greet()  # Now the function runs: "Hello!"
```

### 4.5 The `for...else` Construct

The `else` clause on a `for` loop runs only if the loop completes **without** hitting a `break`. It does NOT mean "if the loop ran zero iterations."

```python
for i in range(5):
    if i == 3:
        break
else:
    print("No break")  # NOT printed — break was hit

for i in range(5):
    pass
else:
    print("Completed!")  # Printed — loop finished normally

# Common use case: search
for item in [1, 2, 3]:
    if item == 99:
        break
else:
    print("Item not found")  # Printed — 99 wasn't in the list
```

### 4.6 Banker's Rounding (Round Half to Even)

Python 3's `round()` uses **banker's rounding**: values exactly halfway between two integers round to the nearest even number, not always up.

```python
print(round(0.5))   # 0  — rounds to even
print(round(1.5))   # 2  — rounds to even
print(round(2.5))   # 2  — rounds to even
print(round(3.5))   # 4  — rounds to even
print(round(4.5))   # 4  — rounds to even

# Floating point can introduce surprises here too:
print(round(2.675, 2))  # 2.67, not 2.68 — due to float representation
```

### 4.7 Floor Division Rounds Toward Negative Infinity

`//` always rounds **down** (toward negative infinity), not toward zero.

```python
print(7 // 2)    #  3  (rounds down from 3.5)
print(-7 // 2)   # -4  (rounds down from -3.5, NOT -3)
print(7 // -2)   # -4
print(-7 // -2)  #  3
```

### 4.8 Booleans Are Integers

`bool` is a subclass of `int`. `True == 1` and `False == 0`, which enables some surprising arithmetic and indexing behavior.

```python
print(True + True)    # 2
print(True * 5)       # 5
print(False + 1)      # 1
print(isinstance(True, int))  # True

# Booleans can index into lists!
a = ['no', 'yes']
print(a[True])   # 'yes'
print(a[False])  # 'no'

# And they count in sum()
flags = [True, False, True, True, False]
print(sum(flags))  # 3 — counts the True values
```

---

## 5. Operators and Comparisons

### 5.1 `is` vs `==`

`==` checks value equality. `is` checks **object identity** (same memory address). Never use `is` to compare values — only use it to check for `None`, `True`, or `False`.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)  # True  — same values
print(a is b)  # False — different objects

# The only reliable uses of 'is':
x = None
print(x is None)   # True  — correct way to check for None
print(x == None)   # True  — works but not idiomatic
```

### 5.2 Augmented Assignment Creates a New Object for Immutables

For immutable types (int, str, tuple), `+=` rebinds the variable to a new object. For mutable types (list), it modifies in-place.

```python
# Immutable (int): new object created
a = 5
original_id = id(a)
a += 1
print(id(a) == original_id)  # False — new object

# Mutable (list): modified in-place
b = [1, 2]
original_id = id(b)
b += [3]
print(id(b) == original_id)  # True — same object
```

### 5.3 Negative Indexing and Modulo

Negative indexing in Python works via modulo arithmetic internally. `-1` is the last element, `-2` second to last, etc.

```python
a = [10, 20, 30, 40]
print(a[-1])   # 40
print(a[-2])   # 30

# Modulo behaves consistently with floor division (toward -inf)
print(-1 % 5)   # 4  — NOT -1
print(-7 % 3)   # 2  — NOT -1
```

### 5.4 Short-Circuit Evaluation Returns Values, Not Booleans

`and` and `or` return the actual value that determined the outcome, not `True` or `False`.

```python
# 'or' returns first truthy value (or last if all falsy)
print(0 or 'hello')     # 'hello'
print('' or 0 or [])    # []
print('' or 0 or 42)    # 42

# 'and' returns first falsy value (or last if all truthy)
print(1 and 2 and 3)    # 3
print(1 and 0 and 3)    # 0

# Common idiom: default values
name = None
display = name or 'Anonymous'
print(display)  # 'Anonymous'
```

---

## 6. Classes and OOP

### 6.1 Mutable Class Attributes Are Shared Between Instances

Class-level attributes are shared. If it's mutable and you mutate it via one instance (without reassigning), it changes for all instances.

```python
class MyClass:
    data = []  # Class-level attribute

a = MyClass()
b = MyClass()

a.data.append(1)   # Mutates the shared list
print(b.data)      # [1] — b sees the change!

# Fix: initialize in __init__
class MyClass:
    def __init__(self):
        self.data = []  # Instance-level attribute
```

### 6.2 `__del__` Is Unreliable

The `__del__` destructor is not called at a predictable time — it depends on the garbage collector, which may never run in some environments.

```python
class Resource:
    def __del__(self):
        print("Cleaned up!")  # May or may not print, and when is unpredictable

# Fix: use a context manager
class Resource:
    def __enter__(self):
        return self
    def __exit__(self, *args):
        print("Cleaned up!")  # Guaranteed to run

with Resource() as r:
    pass  # "Cleaned up!" is printed here
```

### 6.3 `super()` in Multiple Inheritance (MRO)

Python uses the C3 linearization algorithm (Method Resolution Order) for multiple inheritance. `super()` follows the MRO, not the class hierarchy directly.

```python
class A:
    def hello(self):
        print("A")

class B(A):
    def hello(self):
        super().hello()
        print("B")

class C(A):
    def hello(self):
        super().hello()
        print("C")

class D(B, C):
    pass

D().hello()
# Output:
# A
# C
# B
# MRO for D: [D, B, C, A]
print(D.__mro__)  # (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

### 6.4 `__slots__` Prevents Dynamic Attributes

Defining `__slots__` restricts instances to only those attributes — you cannot add arbitrary new ones.

```python
class Rigid:
    __slots__ = ['x', 'y']

r = Rigid()
r.x = 1      # Fine
r.z = 3      # AttributeError: 'Rigid' object has no attribute 'z'
```

---

## 7. Strings and Encoding

### 7.1 String Interning

CPython automatically interns short strings that look like identifiers. Two such string literals can end up as the same object, but this is an implementation detail — never rely on `is` for string comparison.

```python
a = 'hello'
b = 'hello'
print(a is b)  # True — interned (in CPython, not guaranteed)

a = 'hello world'
b = 'hello world'
print(a is b)  # May be True or False — not reliable

# Always use == for string comparison
print(a == b)  # True — always correct
```

### 7.2 String Multiplication Creates One Object

Multiplying a string produces a new string object, but the resulting characters are all the same.

```python
s = '-' * 10
print(s)  # ----------

# This is efficient because strings are immutable;
# concatenation in a loop is slow:
result = ''
for _ in range(1000):
    result += 'x'  # Creates 1000 new string objects — slow!

# Fix: use join
result = ''.join(['x'] * 1000)  # Much faster
```

### 7.3 `f-string` Expressions Are Evaluated Immediately

Unlike `str.format()` templates, f-strings evaluate their expressions at the point of creation, not later.

```python
name = 'Alice'
greeting = f'Hello, {name}!'
name = 'Bob'
print(greeting)  # 'Hello, Alice!' — captured at creation time
```

---

## 8. Exception Handling

### 8.1 Bare `except` Catches Everything Including `SystemExit`

A bare `except:` clause catches *everything*, including `KeyboardInterrupt` and `SystemExit`, which are not subclasses of `Exception`.

```python
# Dangerous — catches keyboard interrupt, system exit, etc.
try:
    pass
except:  # Too broad!
    pass

# Better — only catches actual errors
try:
    pass
except Exception:
    pass

# Or be even more specific
try:
    pass
except (ValueError, TypeError) as e:
    print(f"Error: {e}")
```

### 8.2 Exception Variable Deleted After `except` Block

The exception variable bound with `as e` is deleted after the `except` block ends, even if you try to use it afterward.

```python
try:
    raise ValueError("oops")
except ValueError as e:
    captured = str(e)  # Must save it before the block ends
    print(e)           # Works here: "oops"

print(e)        # NameError: name 'e' is not defined
print(captured) # "oops" — saved correctly
```

### 8.3 Chained Exceptions with `raise from`

When raising a new exception inside an `except` block, Python automatically chains it to the original. You can use `raise ... from None` to suppress the chain.

```python
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("Conversion failed") from e
    # Shows: RuntimeError: Conversion failed
    # AND: ValueError: invalid literal... (the cause)

# To suppress the context:
try:
    int("abc")
except ValueError:
    raise RuntimeError("Conversion failed") from None
    # Only shows: RuntimeError: Conversion failed
```

---

## 9. Imports and Modules

### 9.1 Circular Imports

If module A imports module B and module B imports module A, you get a circular import. Python handles this by partially loading modules, which can cause `ImportError` or `AttributeError` depending on what's been defined so far.

```python
# a.py
from b import greet   # ImportError if b.py also imports from a.py at top level

# Fix: move imports inside functions, or restructure modules
def use_greet():
    from b import greet   # Deferred import avoids circular dependency
    greet()
```

### 9.2 Module-Level Code Runs on Import

All top-level code in a module runs the moment the module is imported — not just when functions are called. Use `if __name__ == '__main__':` to guard executable code.

```python
# bad_module.py
print("This runs on every import!")   # Runs immediately on import

# good_module.py
def main():
    print("Only runs when called")

if __name__ == '__main__':
    main()  # Only runs when executed directly
```

### 9.3 `from module import *` Pollution

Importing everything with `*` dumps all names into the current namespace, potentially shadowing built-ins or other variables without warning.

```python
from os.path import *  # join, exists, etc. all dumped into local scope
from math import *     # Could overwrite things from os.path if names clash

# Always prefer explicit imports
from os.path import join, exists
from math import sqrt, pi
```

---

## 10. Type System Surprises

### 10.1 `None` in Arithmetic

`None` does not behave like `0` or `False` in arithmetic contexts — it raises a `TypeError`.

```python
print(None == False)  # False
print(None == 0)      # False
print(bool(None))     # False

total = None
total + 1             # TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'

# Fix: always check or default
total = total or 0
total += 1
```

### 10.2 `isinstance()` vs `type()` for Type Checking

`type()` checks for an exact type. `isinstance()` respects inheritance and is almost always the better choice.

```python
class Animal: pass
class Dog(Animal): pass

d = Dog()

print(type(d) == Animal)       # False — exact match
print(isinstance(d, Animal))   # True  — respects inheritance

# isinstance also supports multiple types
print(isinstance(42, (int, float)))  # True
```

### 10.3 `float('inf')` and `float('nan')` Behavior

Infinity and NaN have special comparison rules: NaN is not equal to anything, including itself.

```python
import math

inf = float('inf')
nan = float('nan')

print(inf > 999999999)   # True
print(inf + 1 == inf)    # True
print(nan == nan)        # False — NaN is never equal to itself!
print(nan != nan)        # True
print(math.isnan(nan))   # True — correct way to check

# NaN propagates through arithmetic
print(nan + 1)   # nan
print(nan * 0)   # nan
```

### 10.4 Integer Division Returns `float` with `/`

In Python 3, `/` always returns a `float`, even when both operands are integers and the result is a whole number.

```python
print(10 / 2)    # 5.0  — float, not 5
print(type(10 / 2))  # <class 'float'>

print(10 // 2)   # 5    — integer floor division
print(type(10 // 2))  # <class 'int'>
```

### 10.5 `NotImplemented` vs `NotImplementedError`

`NotImplemented` is a special singleton returned by comparison/arithmetic dunder methods to signal "I don't know how to handle this." `NotImplementedError` is an exception for abstract methods.

```python
class MyNum:
    def __add__(self, other):
        if not isinstance(other, MyNum):
            return NotImplemented  # Tells Python to try the other operand's __radd__
        return MyNum()

# NotImplementedError is for abstract/unimplemented methods
class Base:
    def do_thing(self):
        raise NotImplementedError("Subclasses must implement do_thing()")
```

---

## Quick Reference Cheat Sheet

| Gotcha | Short Rule |
|--------|------------|
| Late binding closures | Variables captured by reference, not by value |
| Mutable default args | Use `None`; create mutable objects inside the function |
| `is` vs `==` | `is` = same object; `==` = same value |
| `//` floor division | Rounds toward **−∞**, not toward zero |
| `round(0.5)` | Rounds to nearest **even**, not always up |
| `True` is `1` | `bool` is a subclass of `int` |
| Generator exhaustion | One-time use; convert to list if you need to re-iterate |
| `*` list multiplication | Copies reference, not nested objects |
| `finally` return | Always wins over `try` return |
| `for...else` | `else` runs only if no `break` occurred |
| `NaN != NaN` | Use `math.isnan()` to check for NaN |
| Dict key equality | `1`, `True`, `1.0` are the same key |
| Slicing out of bounds | Never raises; returns empty or partial list |
| String `extend()` | Strings unpack char-by-char |
| Exception `as e` | Variable deleted after `except` block exits |