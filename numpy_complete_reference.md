# NumPy Complete Reference Guide

A comprehensive guide to NumPy - from basics to advanced functions, explained for beginners.

---

## 1. NumPy Basics

### Import NumPy

```python
import numpy as np
```

### Creating Basic Arrays

**np.array()** - Create an array from a Python list or tuple

```python
# 1D array
arr = np.array([1, 2, 3, 4, 5])

# 2D array
arr2d = np.array([[1, 2, 3], [4, 5, 6]])

# Specify data type
arr_float = np.array([1, 2, 3], dtype=float)
```

### Key Array Properties

**dtype** - Data type of array elements

```python
arr = np.array([1, 2, 3])
print(arr.dtype)  # int32 or int64
```

**shape** - Dimensions of the array

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.shape)  # (2, 3) - 2 rows, 3 columns
```

**ndim** - Number of dimensions

```python
arr1d = np.array([1, 2, 3])
print(arr1d.ndim)  # 1

arr2d = np.array([[1, 2], [3, 4]])
print(arr2d.ndim)  # 2
```

**size** - Total number of elements

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.size)  # 6
```

### Type Conversion

**astype()** - Convert array to different data type

```python
arr = np.array([1, 2, 3])
arr_float = arr.astype(float)
arr_str = arr.astype(str)

# Careful! This truncates decimals
arr_float = np.array([1.7, 2.3, 3.9])
arr_int = arr_float.astype(int)  # [1, 2, 3]
```

### Copy vs View

**copy()** - Creates a deep copy (independent)

```python
arr = np.array([1, 2, 3])
arr_copy = arr.copy()
arr_copy[0] = 999
print(arr)  # [1, 2, 3] - unchanged
```

**view()** - Creates a shallow copy (shared data)

```python
arr = np.array([1, 2, 3])
arr_view = arr.view()
arr_view[0] = 999
print(arr)  # [999, 2, 3] - changed!
```

---

## 2. Array Creation

### Basic Constructors

**np.asarray()** - Convert input to array (won't copy if already an array)

```python
list_data = [1, 2, 3]
arr = np.asarray(list_data)

# Difference from np.array: won't copy if already array
existing = np.array([1, 2, 3])
same = np.asarray(existing)  # No copy made
```

### Arrays of Zeros, Ones, and Empty

**np.zeros()** - Array filled with zeros

```python
zeros_1d = np.zeros(5)  # [0. 0. 0. 0. 0.]
zeros_2d = np.zeros((3, 4))  # 3x4 array of zeros
zeros_int = np.zeros(5, dtype=int)
```

**np.ones()** - Array filled with ones

```python
ones_1d = np.ones(5)  # [1. 1. 1. 1. 1.]
ones_2d = np.ones((2, 3))  # 2x3 array of ones
```

**np.empty()** - Uninitialized array (faster, but random values)

```python
empty = np.empty(5)  # Contains whatever was in memory
empty_2d = np.empty((2, 3))
```

### Custom Fill Values

**np.full()** - Array filled with specified value

```python
sevens = np.full(5, 7)  # [7 7 7 7 7]
custom = np.full((2, 3), 3.14)
```

### Ranges and Sequences

**np.arange()** - Like Python's range() but returns array

```python
arr = np.arange(10)  # [0 1 2 3 4 5 6 7 8 9]
arr_step = np.arange(0, 10, 2)  # [0 2 4 6 8]
arr_float = np.arange(0, 1, 0.1)  # [0.  0.1 0.2 ... 0.9]
```

**np.linspace()** - Evenly spaced numbers over interval

```python
# 5 numbers from 0 to 1 (includes both endpoints)
arr = np.linspace(0, 1, 5)  # [0.   0.25 0.5  0.75 1.  ]

# Useful for plotting
x = np.linspace(0, 10, 100)  # 100 points from 0 to 10
```

**np.logspace()** - Evenly spaced on log scale

```python
# 10^0 to 10^3 with 4 points
arr = np.logspace(0, 3, 4)  # [1.  10.  100.  1000.]
```

### Identity and Diagonal Matrices

**np.eye()** - Identity matrix (2D)

```python
identity = np.eye(3)
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]

# Non-square with offset diagonal
arr = np.eye(3, 4, k=1)
```

**np.identity()** - Identity matrix (must be square)

```python
identity = np.identity(3)  # Same as np.eye(3)
```

**np.diag()** - Create diagonal matrix or extract diagonal

```python
# Create diagonal matrix
diag = np.diag([1, 2, 3])
# [[1 0 0]
#  [0 2 0]
#  [0 0 3]]

# Extract diagonal from matrix
matrix = np.array([[1, 2], [3, 4]])
diagonal = np.diag(matrix)  # [1, 4]

# Get offset diagonals
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
diag_above = np.diag(matrix, k=1)  # [2, 6]
```

### Triangular Matrices

**np.tri()** - Array with ones at and below diagonal

```python
tri = np.tri(3)
# [[1. 0. 0.]
#  [1. 1. 0.]
#  [1. 1. 1.]]

tri_offset = np.tri(3, k=1)  # Include one diagonal above
```

**np.tril()** - Lower triangle of array

```python
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
lower = np.tril(matrix)
# [[1 0 0]
#  [4 5 0]
#  [7 8 9]]
```

**np.triu()** - Upper triangle of array

```python
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
upper = np.triu(matrix)
# [[1 2 3]
#  [0 5 6]
#  [0 0 9]]
```

### Random Arrays

**np.random.rand()** - Uniform distribution [0, 1)

```python
random_1d = np.random.rand(5)
random_2d = np.random.rand(3, 4)  # 3x4 array
```

**np.random.randn()** - Standard normal distribution (mean=0, std=1)

```python
normal = np.random.randn(5)
normal_2d = np.random.randn(3, 4)
```

**np.random.randint()** - Random integers

```python
# Random integers from 0 to 9
arr = np.random.randint(0, 10, size=5)

# 2D array of random integers
arr_2d = np.random.randint(1, 100, size=(3, 4))
```

**np.random.choice()** - Random samples from array

```python
# Sample from array
arr = np.array([10, 20, 30, 40])
sample = np.random.choice(arr, size=3)

# Sample with replacement=False (no duplicates)
unique_sample = np.random.choice(arr, size=3, replace=False)

# With probabilities
biased = np.random.choice([0, 1], size=10, p=[0.7, 0.3])
```

### Advanced Creation

**np.fromfunction()** - Create array from function

```python
# Create multiplication table
def mult_table(i, j):
    return (i + 1) * (j + 1)

table = np.fromfunction(mult_table, (5, 5), dtype=int)
```

**np.fromiter()** - Create array from iterable

```python
# From generator
gen = (x**2 for x in range(5))
arr = np.fromiter(gen, dtype=int)  # [0, 1, 4, 9, 16]
```

**np.frombuffer()** - Create array from buffer

```python
# From bytes
byte_data = b'\x01\x02\x03\x04'
arr = np.frombuffer(byte_data, dtype=np.uint8)  # [1 2 3 4]
```

---

## 3. Array Properties & Inspection

### Core Properties

**shape** - Tuple of array dimensions

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.shape)  # (2, 3)

# Reshape by modifying shape
arr.shape = (3, 2)  # Now 3x2
```

**ndim** - Number of dimensions

```python
arr1d = np.array([1, 2, 3])
arr2d = np.array([[1, 2], [3, 4]])
arr3d = np.array([[[1, 2], [3, 4]]])

print(arr1d.ndim)  # 1
print(arr2d.ndim)  # 2
print(arr3d.ndim)  # 3
```

**size** - Total number of elements

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.size)  # 6
```

**dtype** - Data type of elements

```python
arr_int = np.array([1, 2, 3])
arr_float = np.array([1.0, 2.0, 3.0])

print(arr_int.dtype)    # int64 or int32
print(arr_float.dtype)  # float64
```

**itemsize** - Size of each element in bytes

```python
arr_int32 = np.array([1, 2, 3], dtype=np.int32)
arr_float64 = np.array([1.0, 2.0, 3.0], dtype=np.float64)

print(arr_int32.itemsize)    # 4 bytes
print(arr_float64.itemsize)  # 8 bytes
```

**nbytes** - Total bytes consumed by array

```python
arr = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.int32)
print(arr.nbytes)  # 24 bytes (6 elements × 4 bytes)
```

**T** - Transpose of array

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.T)
# [[1 4]
#  [2 5]
#  [3 6]]
```

**flags** - Information about memory layout

```python
arr = np.array([1, 2, 3])
print(arr.flags)
# Shows C_CONTIGUOUS, F_CONTIGUOUS, OWNDATA, WRITEABLE, etc.

print(arr.flags.c_contiguous)  # True
print(arr.flags.writeable)     # True
```

---

## 4. Indexing, Slicing & Masking

### Basic Slicing

```python
arr = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

# Single element
print(arr[0])    # 0
print(arr[-1])   # 9

# Slicing [start:stop:step]
print(arr[2:5])       # [2 3 4]
print(arr[:5])        # [0 1 2 3 4]
print(arr[5:])        # [5 6 7 8 9]
print(arr[::2])       # [0 2 4 6 8]
print(arr[::-1])      # [9 8 7 6 5 4 3 2 1 0] - reversed

# 2D slicing
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(arr2d[0, 1])        # 2 - row 0, col 1
print(arr2d[:2, :2])      # First 2 rows and 2 columns
print(arr2d[1:, 1:])      # From row 1, col 1 onwards
```

### Fancy Indexing

```python
arr = np.array([10, 20, 30, 40, 50])

# Index with array of integers
indices = [0, 2, 4]
print(arr[indices])  # [10 30 50]

# 2D fancy indexing
arr2d = np.array([[1, 2], [3, 4], [5, 6]])
rows = [0, 2]
cols = [1, 0]
print(arr2d[rows, cols])  # [2 5] - (0,1) and (2,0)
```

### Boolean Masking

```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Create boolean mask
mask = arr > 3
print(mask)  # [False False False True True True]

# Apply mask
print(arr[mask])  # [4 5 6]

# One-liner
print(arr[arr > 3])  # [4 5 6]

# Multiple conditions
print(arr[(arr > 2) & (arr < 5)])  # [3 4]
print(arr[(arr < 2) | (arr > 5)])  # [1 6]
```

### np.where()

**np.where()** - Return indices or select elements based on condition

```python
arr = np.array([1, 2, 3, 4, 5])

# Get indices where condition is True
indices = np.where(arr > 3)
print(indices)  # (array([3, 4]),)

# Conditional selection (like ternary operator)
result = np.where(arr > 3, 'big', 'small')
# ['small' 'small' 'small' 'big' 'big']

# Replace values
result = np.where(arr > 3, 999, arr)  # [1 2 3 999 999]
```

### np.take()

**np.take()** - Select elements along axis

```python
arr = np.array([10, 20, 30, 40, 50])
indices = [0, 2, 4]
result = np.take(arr, indices)  # [10 30 50]

# 2D array - along specific axis
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
result = np.take(arr2d, [0, 2], axis=0)  # Take rows 0 and 2
```

### np.put()

**np.put()** - Put values at specific indices (modifies in-place)

```python
arr = np.array([1, 2, 3, 4, 5])
np.put(arr, [0, 4], [999, 888])
print(arr)  # [999   2   3   4 888]
```

### np.compress()

**np.compress()** - Select elements using boolean array

```python
arr = np.array([10, 20, 30, 40, 50])
condition = [True, False, True, False, True]
result = np.compress(condition, arr)  # [10 30 50]

# Works like boolean masking but more explicit
```

---

## 5. Reshaping & Rearranging

### reshape()

**reshape()** - Give new shape without changing data

```python
arr = np.arange(12)  # [0 1 2 3 4 5 6 7 8 9 10 11]

# Reshape to 2D
arr2d = arr.reshape(3, 4)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Reshape to 3D
arr3d = arr.reshape(2, 3, 2)

# Use -1 for automatic dimension
arr2d = arr.reshape(3, -1)  # 3 rows, columns auto-calculated
arr2d = arr.reshape(-1, 4)  # 4 columns, rows auto-calculated
```

### resize()

**resize()** - Change shape (can add or remove elements)

```python
arr = np.array([1, 2, 3, 4])
arr.resize(6)  # [1 2 3 4 0 0] - pads with zeros

arr = np.array([1, 2, 3, 4])
arr.resize(2)  # [1 2] - truncates

# np.resize() repeats array to fill shape
arr = np.array([1, 2, 3])
result = np.resize(arr, 7)  # [1 2 3 1 2 3 1]
```

### ravel()

**ravel()** - Flatten to 1D (returns view if possible)

```python
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
flat = arr2d.ravel()  # [1 2 3 4 5 6]

# Modifying flat may affect original
flat[0] = 999
# Original might be changed (if it's a view)
```

### flatten()

**flatten()** - Flatten to 1D (always returns copy)

```python
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
flat = arr2d.flatten()  # [1 2 3 4 5 6]

# Modifying flat won't affect original
flat[0] = 999
# Original is unchanged
```

### squeeze()

**squeeze()** - Remove single-dimensional entries

```python
arr = np.array([[[1], [2], [3]]])
print(arr.shape)  # (1, 3, 1)

squeezed = arr.squeeze()
print(squeezed.shape)  # (3,)

# Squeeze specific axis
arr = np.array([[[1, 2, 3]]])  # Shape (1, 1, 3)
result = np.squeeze(arr, axis=0)  # Shape (1, 3)
```

### expand_dims()

**expand_dims()** - Add new axis

```python
arr = np.array([1, 2, 3])  # Shape (3,)

# Add axis at start
expanded = np.expand_dims(arr, axis=0)  # Shape (1, 3)

# Add axis at end
expanded = np.expand_dims(arr, axis=1)  # Shape (3, 1)

# Alternative syntax
expanded = arr[np.newaxis, :]  # (1, 3)
expanded = arr[:, np.newaxis]  # (3, 1)
```

### swapaxes()

**swapaxes()** - Swap two axes

```python
arr = np.arange(24).reshape(2, 3, 4)
print(arr.shape)  # (2, 3, 4)

swapped = arr.swapaxes(0, 2)
print(swapped.shape)  # (4, 3, 2)

# For 2D, same as transpose
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
result = arr2d.swapaxes(0, 1)  # Same as arr2d.T
```

### moveaxis()

**moveaxis()** - Move axis to new position

```python
arr = np.arange(24).reshape(2, 3, 4)
print(arr.shape)  # (2, 3, 4)

# Move axis 2 to position 0
moved = np.moveaxis(arr, 2, 0)
print(moved.shape)  # (4, 2, 3)

# Move multiple axes
moved = np.moveaxis(arr, [0, 2], [2, 0])
```

### rollaxis()

**rollaxis()** - Roll specified axis backwards (older, use moveaxis instead)

```python
arr = np.arange(24).reshape(2, 3, 4)
rolled = np.rollaxis(arr, 2, 0)  # Roll axis 2 to position 0
print(rolled.shape)  # (4, 2, 3)
```

### transpose()

**transpose()** - Permute dimensions

```python
arr = np.arange(24).reshape(2, 3, 4)
print(arr.shape)  # (2, 3, 4)

# Reverse all axes
transposed = arr.transpose()
print(transposed.shape)  # (4, 3, 2)

# Specify new order
transposed = arr.transpose(2, 0, 1)
print(transposed.shape)  # (4, 2, 3)

# For 2D, same as .T
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
result = arr2d.transpose()  # Same as arr2d.T
```

---

## 6. Joining & Splitting

### concatenate()

**concatenate()** - Join arrays along existing axis

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.concatenate([arr1, arr2])  # [1 2 3 4 5 6]

# 2D concatenation
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6]])

# Along axis 0 (rows)
result = np.concatenate([arr1, arr2], axis=0)
# [[1 2]
#  [3 4]
#  [5 6]]

# Along axis 1 (columns)
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5], [6]])
result = np.concatenate([arr1, arr2], axis=1)
# [[1 2 5]
#  [3 4 6]]
```

### stack()

**stack()** - Join arrays along NEW axis

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

# Stack along new axis 0
result = np.stack([arr1, arr2], axis=0)
# [[1 2 3]
#  [4 5 6]]

# Stack along new axis 1
result = np.stack([arr1, arr2], axis=1)
# [[1 4]
#  [2 5]
#  [3 6]]
```

**vstack()** - Stack vertically (row-wise)

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.vstack([arr1, arr2])
# [[1 2 3]
#  [4 5 6]]

# Works with 2D arrays
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])
result = np.vstack([arr1, arr2])
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]
```

**hstack()** - Stack horizontally (column-wise)

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.hstack([arr1, arr2])  # [1 2 3 4 5 6]

# With 2D arrays
arr1 = np.array([[1], [2], [3]])
arr2 = np.array([[4], [5], [6]])
result = np.hstack([arr1, arr2])
# [[1 4]
#  [2 5]
#  [3 6]]
```

**dstack()** - Stack along depth (3rd axis)

```python
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])
result = np.dstack([arr1, arr2])
# [[[1 5]
#   [2 6]]
#  [[3 7]
#   [4 8]]]
print(result.shape)  # (2, 2, 2)
```

### column_stack() and row_stack()

**column_stack()** - Stack 1D arrays as columns

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.column_stack([arr1, arr2])
# [[1 4]
#  [2 5]
#  [3 6]]
```

**row_stack()** - Same as vstack()

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.row_stack([arr1, arr2])
# [[1 2 3]
#  [4 5 6]]
```

### split()

**split()** - Split array into multiple sub-arrays

```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Split into 3 equal parts
result = np.split(arr, 3)  # [array([1, 2]), array([3, 4]), array([5, 6])]

# Split at specific indices
result = np.split(arr, [2, 4])  # [:2], [2:4], [4:]
# [array([1, 2]), array([3, 4]), array([5, 6])]

# 2D split
arr2d = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
result = np.split(arr2d, 2, axis=1)  # Split along columns
```

**array_split()** - Split into (possibly) unequal parts

```python
arr = np.array([1, 2, 3, 4, 5])

# Split into 3 parts (won't error even if uneven)
result = np.array_split(arr, 3)
# [array([1, 2]), array([3, 4]), array([5])]
```

**hsplit()** - Split horizontally

```python
arr = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
result = np.hsplit(arr, 2)
# [array([[1, 2], [5, 6]]), array([[3, 4], [7, 8]])]
```

**vsplit()** - Split vertically

```python
arr = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])
result = np.vsplit(arr, 2)
# [array([[1, 2], [3, 4]]), array([[5, 6], [7, 8]])]
```

**dsplit()** - Split along 3rd axis

```python
arr = np.arange(24).reshape(2, 3, 4)
result = np.dsplit(arr, 2)  # Split along depth
```

---

## 7. Mathematical Operations

### Basic Arithmetic

**add(), subtract(), multiply(), divide()** - Element-wise operations

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# Using operators (preferred)
print(a + b)   # [11 22 33 44]
print(a - b)   # [-9 -18 -27 -36]
print(a * b)   # [10 40 90 160]
print(a / b)   # [0.1 0.1 0.1 0.1]

# Using functions
result = np.add(a, b)
result = np.subtract(a, b)
result = np.multiply(a, b)
result = np.divide(a, b)

# Floor division and modulo
print(a // b)  # [0 0 0 0]
print(a % b)   # [1 2 3 4]
```

### Power and Exponential

**power()** - Raise to power

```python
arr = np.array([1, 2, 3, 4])
print(np.power(arr, 2))  # [1 4 9 16]
print(arr ** 2)          # Same thing

# Different power for each element
powers = np.array([1, 2, 3, 4])
print(np.power(arr, powers))  # [1 4 27 256]
```

**sqrt()** - Square root

```python
arr = np.array([1, 4, 9, 16, 25])
print(np.sqrt(arr))  # [1. 2. 3. 4. 5.]
```

**exp()** - e^x

```python
arr = np.array([0, 1, 2])
print(np.exp(arr))  # [1.  2.71828183  7.3890561]
```

**log(), log10(), log2()** - Logarithms

```python
arr = np.array([1, 10, 100, 1000])

print(np.log(arr))     # Natural log (ln)
print(np.log10(arr))   # [0. 1. 2. 3.]
print(np.log2(arr))    # Base-2 log
```

### Rounding

**floor(), ceil(), round()** - Rounding functions

```python
arr = np.array([1.2, 2.7, 3.5, -1.7])

print(np.floor(arr))   # [1. 2. 3. -2.]
print(np.ceil(arr))    # [2. 3. 4. -1.]
print(np.round(arr))   # [1. 3. 4. -2.]

# Round to specific decimals
arr = np.array([1.234, 2.567])
print(np.round(arr, 2))  # [1.23 2.57]
```

**mod(), remainder()** - Modulo operations

```python
a = np.array([10, 15, 20])
b = np.array([3, 4, 6])

print(np.mod(a, b))       # [1 3 2]
print(np.remainder(a, b)) # Same as mod
```

---

## 8. Aggregations & Statistics

### Basic Aggregations

**sum()** - Sum of array elements

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.sum(arr))  # 15
print(arr.sum())    # 15 (method version)

# Sum along axis
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2d.sum(axis=0))  # [5 7 9] - sum each column
print(arr2d.sum(axis=1))  # [6 15] - sum each row
```

**mean()** - Average

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.mean(arr))  # 3.0

arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2d.mean(axis=0))  # [2.5 3.5 4.5]
```

**median()** - Median value

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.median(arr))  # 3.0

arr = np.array([1, 2, 3, 4, 5, 6])
print(np.median(arr))  # 3.5 (average of middle two)
```

### Min and Max

**min(), max()** - Minimum and maximum

```python
arr = np.array([3, 1, 4, 1, 5, 9])
print(np.min(arr))  # 1
print(np.max(arr))  # 9

# Along axis
arr2d = np.array([[1, 5, 3], [2, 4, 6]])
print(arr2d.min(axis=1))  # [1, 2] - min of each row
```

**argmin(), argmax()** - Return indices of min/max

```python
arr = np.array([3, 1, 4, 1, 5, 9])
print(np.argmin(arr))  # 1 (first occurrence)
print(np.argmax(arr))  # 5

# In 2D
arr2d = np.array([[1, 5, 3], [2, 4, 6]])
print(arr2d.argmax(axis=1))  # [1, 2] - index of max in each row
```

### Variability

**std()** - Standard deviation

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.std(arr))  # 1.4142135623730951
```

**var()** - Variance

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.var(arr))  # 2.0 (std squared)
```

### Percentiles and Quantiles

**percentile()** - Percentile of data

```python
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
print(np.percentile(arr, 50))  # 5.5 (median)
print(np.percentile(arr, 25))  # 3.25 (1st quartile)
print(np.percentile(arr, 75))  # 7.75 (3rd quartile)

# Multiple percentiles at once
print(np.percentile(arr, [25, 50, 75]))
```

**quantile()** - Quantile (like percentile but 0-1 scale)

```python
arr = np.array([1, 2, 3, 4, 5])
print(np.quantile(arr, 0.5))   # 3.0 (median)
print(np.quantile(arr, 0.25))  # 2.0
print(np.quantile(arr, [0.25, 0.5, 0.75]))
```

### Other Aggregations

**ptp()** - Peak to peak (max - min)

```python
arr = np.array([1, 5, 2, 8, 3])
print(np.ptp(arr))  # 7 (8 - 1)
```

**bincount()** - Count occurrences of each value

```python
arr = np.array([0, 1, 1, 2, 2, 2, 3])
print(np.bincount(arr))  # [1 2 3 1]
# Index 0: 1 occurrence, Index 1: 2 occurrences, etc.

# Works only with non-negative integers
arr = np.array([1, 2, 2, 3, 3, 3])
print(np.bincount(arr))  # [0 1 2 3]
```

---

## 9. Broadcasting

### What is Broadcasting?

Broadcasting allows NumPy to perform operations on arrays of different shapes. The smaller array is "broadcast" across the larger array.

### Broadcasting Rules

1. If arrays have different number of dimensions, pad the smaller one with 1s on the left
2. Arrays are compatible if dimensions are equal OR one of them is 1
3. Arrays are broadcast together to the largest shape along each dimension

### Examples of Compatible Shapes

```python
# Scalar and array
arr = np.array([1, 2, 3])
result = arr + 10  # [11 12 13]
# 10 is broadcast to [10, 10, 10]

# 1D and 2D
arr2d = np.array([[1, 2, 3],
                  [4, 5, 6]])
arr1d = np.array([10, 20, 30])

result = arr2d + arr1d
# [[11 22 33]
#  [14 25 36]]
# arr1d is broadcast to each row

# Column vector and row
col = np.array([[1], [2], [3]])  # Shape (3, 1)
row = np.array([10, 20, 30])     # Shape (3,) -> (1, 3)

result = col + row
# [[11 21 31]
#  [12 22 32]
#  [13 23 33]]
```

### Shape Compatibility Examples

```python
# Compatible shapes
# (3, 4) and (3, 1) -> broadcasts to (3, 4)
# (3, 4) and (1, 4) -> broadcasts to (3, 4)
# (3, 4) and (4,)   -> broadcasts to (3, 4)
# (256, 256, 3) and (3,) -> broadcasts to (256, 256, 3)

# Example: Normalize each color channel
image = np.random.rand(256, 256, 3)  # RGB image
mean = np.array([0.5, 0.6, 0.4])     # Mean per channel
normalized = image - mean  # Broadcasting works!

# Example: Grid operations
x = np.arange(5).reshape(5, 1)  # [[0], [1], [2], [3], [4]]
y = np.arange(5).reshape(1, 5)  # [[0, 1, 2, 3, 4]]

# Create distance grid
distance = np.sqrt(x**2 + y**2)
```

### Common Broadcasting Mistakes

```python
# WRONG! Incompatible shapes
arr1 = np.ones((3, 4))
arr2 = np.ones((3, 2))
# result = arr1 + arr2  # ValueError!

# FIX: Reshape to make compatible
arr1 = np.ones((3, 4))
arr2 = np.ones((3, 1))
result = arr1 + arr2  # Works! (3, 4)

# WRONG! Can't align dimensions
arr1 = np.ones((3, 4))
arr2 = np.ones((5,))
# result = arr1 + arr2  # ValueError! 5 != 4

# Remember: broadcast rules are strict!
# Dimensions must be equal OR one must be 1
```

### Practical Broadcasting Examples

```python
# Standardize dataset (mean=0, std=1)
data = np.random.rand(100, 5)  # 100 samples, 5 features
mean = data.mean(axis=0)       # Shape (5,)
std = data.std(axis=0)         # Shape (5,)
standardized = (data - mean) / std

# Create coordinate grids
x = np.linspace(-5, 5, 100).reshape(-1, 1)  # (100, 1)
y = np.linspace(-5, 5, 100).reshape(1, -1)  # (1, 100)
z = np.sqrt(x**2 + y**2)  # (100, 100) - 2D grid!

# Add noise to multiple samples
samples = np.random.rand(1000, 10)
noise = np.random.randn(10)  # Same noise for all samples
noisy = samples + noise
```

---

## 10. Linear Algebra (np.linalg)

### Matrix Multiplication

**dot()** - Dot product

```python
# 1D dot product
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print(np.dot(a, b))  # 32 (1*4 + 2*5 + 3*6)

# 2D matrix multiplication
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(np.dot(A, B))
# [[19 22]
#  [43 50]]
```

**matmul()** - Matrix multiplication (@ operator)

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

result = np.matmul(A, B)
# OR
result = A @ B

# Note: matmul treats arrays differently than dot
# matmul doesn't allow scalar multiplication
```

**transpose()** - Matrix transpose

```python
A = np.array([[1, 2, 3], [4, 5, 6]])
print(A.T)
# OR
print(np.transpose(A))
# [[1 4]
#  [2 5]
#  [3 6]]
```

### Matrix Inversion

**inv()** - Matrix inverse

```python
A = np.array([[1, 2], [3, 4]])
A_inv = np.linalg.inv(A)

# Verify: A @ A_inv should be identity
print(A @ A_inv)
# [[1. 0.]
#  [0. 1.]]

# Note: Only works for square, non-singular matrices
```

**det()** - Determinant

```python
A = np.array([[1, 2], [3, 4]])
det = np.linalg.det(A)
print(det)  # -2.0

# Zero determinant means matrix is singular (no inverse)
B = np.array([[1, 2], [2, 4]])
print(np.linalg.det(B))  # 0.0
```

### Eigenvalues and Eigenvectors

**eig()** - Compute eigenvalues and eigenvectors

```python
A = np.array([[1, 2], [3, 4]])
eigenvalues, eigenvectors = np.linalg.eig(A)

print(eigenvalues)
# [-0.37228132  5.37228132]

print(eigenvectors)
# Each column is an eigenvector

# Verify: A @ v = λ * v
λ = eigenvalues[0]
v = eigenvectors[:, 0]
print(np.allclose(A @ v, λ * v))  # True
```

### Singular Value Decomposition

**svd()** - Singular Value Decomposition

```python
A = np.array([[1, 2], [3, 4], [5, 6]])

# A = U @ S @ Vt
U, s, Vt = np.linalg.svd(A)

print(U.shape)   # (3, 3) - left singular vectors
print(s.shape)   # (2,) - singular values
print(Vt.shape)  # (2, 2) - right singular vectors

# Reconstruct original matrix
S = np.zeros((3, 2))
S[:2, :2] = np.diag(s)
A_reconstructed = U @ S @ Vt
```

### Solving Linear Systems

**solve()** - Solve linear system Ax = b

```python
# Solve: 2x + 3y = 8
#        3x + 4y = 11
A = np.array([[2, 3], [3, 4]])
b = np.array([8, 11])

x = np.linalg.solve(A, b)
print(x)  # [1. 2.]

# Verify solution
print(A @ x)  # [8. 11.] ✓
```

**norm()** - Vector/matrix norm

```python
# Vector norms
v = np.array([3, 4])
print(np.linalg.norm(v))  # 5.0 (Euclidean/L2 norm)
print(np.linalg.norm(v, 1))  # 7.0 (L1 norm)
print(np.linalg.norm(v, np.inf))  # 4.0 (infinity norm)

# Matrix norms
A = np.array([[1, 2], [3, 4]])
print(np.linalg.norm(A))  # Frobenius norm
print(np.linalg.norm(A, 'fro'))  # Same thing
```

---

## 11. Sorting & Searching

### Sorting

**sort()** - Sort array

```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

# Returns sorted copy
sorted_arr = np.sort(arr)  # [1 1 2 3 4 5 6 9]

# Sort in-place
arr.sort()
print(arr)  # [1 1 2 3 4 5 6 9]

# Sort 2D along axis
arr2d = np.array([[3, 2, 1], [6, 5, 4]])
sorted_rows = np.sort(arr2d, axis=1)
# [[1 2 3]
#  [4 5 6]]

sorted_cols = np.sort(arr2d, axis=0)
# [[3 2 1]
#  [6 5 4]]
```

**argsort()** - Return indices that would sort array

```python
arr = np.array([3, 1, 4, 1, 5, 9])
indices = np.argsort(arr)
print(indices)  # [1 3 0 2 4 5]

# Use indices to sort
sorted_arr = arr[indices]  # [1 1 3 4 5 9]

# Useful for sorting multiple related arrays
names = np.array(['Charlie', 'Alice', 'Bob'])
ages = np.array([30, 25, 35])

sort_indices = np.argsort(names)
sorted_names = names[sort_indices]
sorted_ages = ages[sort_indices]
```

**lexsort()** - Sort by multiple keys

```python
# Sort by last name, then first name
first_names = np.array(['Alice', 'Bob', 'Alice', 'Charlie'])
last_names = np.array(['Smith', 'Smith', 'Jones', 'Brown'])

# lexsort sorts by LAST key first
indices = np.lexsort((first_names, last_names))
print(indices)  # [3 2 0 1]

# Result: Brown Charlie, Jones Alice, Smith Alice, Smith Bob
```

**searchsorted()** - Find insertion indices to maintain sorted order

```python
arr = np.array([1, 3, 5, 7, 9])

# Where should we insert 4 to keep array sorted?
idx = np.searchsorted(arr, 4)
print(idx)  # 2 (between 3 and 5)

# Insert multiple values
indices = np.searchsorted(arr, [0, 4, 10])
print(indices)  # [0 2 5]

# Side parameter
# 'left': insert before any equal elements (default)
# 'right': insert after any equal elements
arr = np.array([1, 2, 2, 2, 3])
print(np.searchsorted(arr, 2, side='left'))   # 1
print(np.searchsorted(arr, 2, side='right'))  # 4
```

**partition()** - Partial sort

```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

# Partition at index 3
# Elements before index 3 are smaller
# Element at index 3 is in correct position
# Elements after index 3 are larger
partitioned = np.partition(arr, 3)
print(partitioned)  # [1 1 2 3 4 5 9 6]

# Useful for finding k-th smallest/largest
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
k_smallest = np.partition(arr, 2)[2]  # 3rd smallest
k_largest = np.partition(arr, -3)[-3]  # 3rd largest
```

---

## 12. Random Module (np.random)

### Setting Seed

**seed()** - Set random seed for reproducibility

```python
np.random.seed(42)
arr1 = np.random.rand(3)

np.random.seed(42)
arr2 = np.random.rand(3)

print(np.array_equal(arr1, arr2))  # True - same results
```

### Random Arrays

**rand()** - Uniform distribution [0, 1)

```python
arr = np.random.rand(5)  # 5 random numbers
arr2d = np.random.rand(3, 4)  # 3x4 array
```

**randn()** - Standard normal distribution

```python
arr = np.random.randn(5)  # Mean=0, Std=1
arr2d = np.random.randn(3, 4)
```

**randint()** - Random integers

```python
# Random integers from 0 to 9
arr = np.random.randint(0, 10, size=5)

# From 1 to 100
arr = np.random.randint(1, 101, size=10)

# 2D array
arr2d = np.random.randint(0, 256, size=(10, 10))
```

### Distributions

**uniform()** - Uniform distribution over [low, high)

```python
# Random numbers between 5 and 10
arr = np.random.uniform(5, 10, size=5)
```

**normal()** - Normal (Gaussian) distribution

```python
# mean=0, std=1 (same as randn)
arr = np.random.normal(0, 1, size=5)

# mean=100, std=15 (like IQ scores)
iq_scores = np.random.normal(100, 15, size=1000)
```

### Shuffling and Permutations

**shuffle()** - Shuffle array in-place

```python
arr = np.array([1, 2, 3, 4, 5])
np.random.shuffle(arr)
print(arr)  # [3 1 5 2 4] (randomly shuffled)

# For multi-dimensional arrays, shuffles along first axis
arr2d = np.array([[1, 2], [3, 4], [5, 6]])
np.random.shuffle(arr2d)  # Shuffles rows
```

**permutation()** - Return shuffled copy

```python
arr = np.array([1, 2, 3, 4, 5])
shuffled = np.random.permutation(arr)
print(arr)       # [1 2 3 4 5] - unchanged
print(shuffled)  # [3 1 5 2 4] - shuffled copy

# Can also generate permutation of integers
perm = np.random.permutation(10)  # Random permutation of 0-9
```

**choice()** - Random sample from array

```python
arr = np.array([10, 20, 30, 40, 50])

# Sample 3 elements (with replacement)
sample = np.random.choice(arr, size=3)

# Sample without replacement
unique_sample = np.random.choice(arr, size=3, replace=False)

# With custom probabilities
biased = np.random.choice(arr, size=100, p=[0.5, 0.2, 0.2, 0.05, 0.05])
```

---

## 13. Logic & Comparison

### Boolean Operations

**all()** - Test if all elements are True

```python
arr = np.array([True, True, True])
print(np.all(arr))  # True

arr = np.array([True, False, True])
print(np.all(arr))  # False

# Check if all elements > 0
arr = np.array([1, 2, 3, 4])
print(np.all(arr > 0))  # True

# Along axis
arr2d = np.array([[True, True], [True, False]])
print(np.all(arr2d, axis=1))  # [True False]
```

**any()** - Test if any element is True

```python
arr = np.array([False, False, False])
print(np.any(arr))  # False

arr = np.array([False, True, False])
print(np.any(arr))  # True

# Check if any element > 10
arr = np.array([1, 5, 15, 3])
print(np.any(arr > 10))  # True
```

### Logical Operations

**logical_and()** - Element-wise AND

```python
a = np.array([True, True, False, False])
b = np.array([True, False, True, False])
print(np.logical_and(a, b))  # [True False False False]

# With conditions
arr = np.array([1, 5, 10, 15, 20])
result = np.logical_and(arr > 5, arr < 15)  # [False False False True False]
```

**logical_or()** - Element-wise OR

```python
a = np.array([True, True, False, False])
b = np.array([True, False, True, False])
print(np.logical_or(a, b))  # [True True True False]
```

**logical_not()** - Element-wise NOT

```python
arr = np.array([True, False, True])
print(np.logical_not(arr))  # [False True False]

# Invert condition
arr = np.array([1, 2, 3, 4, 5])
mask = arr > 3  # [False False False True True]
inverted = np.logical_not(mask)  # [True True True False False]
```

### Membership

**isin()** - Test element-wise membership

```python
arr = np.array([1, 2, 3, 4, 5])
test_values = [2, 4, 6]

result = np.isin(arr, test_values)
# [False True False True False]

# Get elements that ARE in test_values
print(arr[result])  # [2 4]

# Get elements that are NOT in test_values
print(arr[~result])  # [1 3 5]
```

### Comparison Operations

**equal(), not_equal()** - Element-wise equality

```python
a = np.array([1, 2, 3])
b = np.array([1, 5, 3])

print(np.equal(a, b))      # [True False True]
print(np.not_equal(a, b))  # [False True False]

# Same as using operators
print(a == b)  # [True False True]
print(a != b)  # [False True False]
```

**greater(), less()** - Comparisons

```python
a = np.array([1, 2, 3])
b = np.array([3, 2, 1])

print(np.greater(a, b))        # [False False True]
print(np.greater_equal(a, b))  # [False True True]
print(np.less(a, b))           # [True False False]
print(np.less_equal(a, b))     # [True True False]

# Same as operators
print(a > b)   # [False False True]
print(a >= b)  # [False True True]
print(a < b)   # [True False False]
print(a <= b)  # [True True False]
```

---

## 14. Set Operations

### Unique Values

**unique()** - Find unique elements

```python
arr = np.array([1, 2, 2, 3, 3, 3, 4, 4, 4, 4])
unique = np.unique(arr)
print(unique)  # [1 2 3 4]

# Return counts
unique, counts = np.unique(arr, return_counts=True)
print(unique)  # [1 2 3 4]
print(counts)  # [1 2 3 4]

# Return indices
arr = np.array([3, 1, 2, 1, 3])
unique, indices = np.unique(arr, return_index=True)
print(unique)   # [1 2 3]
print(indices)  # [1 2 0] - first occurrence of each
```

### Set Operations

**intersect1d()** - Intersection of two arrays

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([3, 4, 5, 6, 7])

intersection = np.intersect1d(a, b)
print(intersection)  # [3 4 5]
```

**union1d()** - Union of two arrays

```python
a = np.array([1, 2, 3])
b = np.array([3, 4, 5])

union = np.union1d(a, b)
print(union)  # [1 2 3 4 5]
```

**setdiff1d()** - Set difference (elements in a but not in b)

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([3, 4, 5, 6, 7])

diff = np.setdiff1d(a, b)
print(diff)  # [1 2]
```

**setxor1d()** - Symmetric difference (elements in either but not both)

```python
a = np.array([1, 2, 3, 4])
b = np.array([3, 4, 5, 6])

xor = np.setxor1d(a, b)
print(xor)  # [1 2 5 6]
```

---

## 15. Handling NaN & Inf

### Detection

**isnan()** - Check for NaN values

```python
arr = np.array([1, 2, np.nan, 4, np.nan])
print(np.isnan(arr))  # [False False True False True]

# Count NaNs
print(np.sum(np.isnan(arr)))  # 2

# Get non-NaN values
clean = arr[~np.isnan(arr)]  # [1. 2. 4.]
```

**isinf()** - Check for infinity

```python
arr = np.array([1, np.inf, -np.inf, 4])
print(np.isinf(arr))  # [False True True False]

# Positive infinity only
print(np.isposinf(arr))  # [False True False False]

# Negative infinity only
print(np.isneginf(arr))  # [False False True False]
```

**isfinite()** - Check for finite values (not NaN or Inf)

```python
arr = np.array([1, np.nan, np.inf, 4])
print(np.isfinite(arr))  # [True False False True]

# Get only finite values
finite = arr[np.isfinite(arr)]  # [1. 4.]
```

### NaN-Aware Statistics

**nanmean(), nansum()** - Ignore NaN in calculations

```python
arr = np.array([1, 2, np.nan, 4, 5])

print(np.mean(arr))     # nan
print(np.nanmean(arr))  # 3.0 (ignores NaN)

print(np.sum(arr))      # nan
print(np.nansum(arr))   # 12.0
```

**Other NaN-aware functions:**

```python
arr = np.array([1, 2, np.nan, 4, 5])

print(np.nanmedian(arr))  # 3.0
print(np.nanstd(arr))     # Standard deviation
print(np.nanvar(arr))     # Variance
print(np.nanmin(arr))     # 1.0
print(np.nanmax(arr))     # 5.0
print(np.nanargmin(arr))  # 0
print(np.nanargmax(arr))  # 4
```

### Replace NaN/Inf

**nan_to_num()** - Replace NaN/Inf with numbers

```python
arr = np.array([1, np.nan, np.inf, -np.inf, 5])

clean = np.nan_to_num(arr)
print(clean)
# [1.0e+00  0.0e+00  1.7976931e+308  -1.7976931e+308  5.0e+00]
# NaN -> 0, Inf -> very large number

# Custom replacements
clean = np.nan_to_num(arr, nan=0.0, posinf=999, neginf=-999)
print(clean)  # [1. 0. 999. -999. 5.]
```

---

## 16. File I/O

### Binary Format (NumPy specific)

**save()** - Save array to .npy file

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
np.save('my_array.npy', arr)

# Preserves dtype, shape, etc.
```

**load()** - Load array from .npy file

```python
loaded = np.load('my_array.npy')
print(loaded)
# [[1 2 3]
#  [4 5 6]]
```

**savez()** - Save multiple arrays to single .npz file

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

np.savez('arrays.npz', first=arr1, second=arr2)

# Load back
data = np.load('arrays.npz')
print(data['first'])   # [1 2 3]
print(data['second'])  # [4 5 6]
```

**savez_compressed()** - Compressed .npz file

```python
large_array = np.random.rand(1000, 1000)
np.savez_compressed('compressed.npz', data=large_array)
```

### Text Format

**savetxt()** - Save to text file

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
np.savetxt('data.txt', arr)

# With custom delimiter and format
np.savetxt('data.csv', arr, delimiter=',', fmt='%d')

# With header
np.savetxt('data.csv', arr, delimiter=',', 
           header='col1,col2,col3', comments='')
```

**loadtxt()** - Load from text file

```python
data = np.loadtxt('data.txt')
print(data)

# With delimiter
data = np.loadtxt('data.csv', delimiter=',')

# Skip header rows
data = np.loadtxt('data.csv', delimiter=',', skiprows=1)

# Select specific columns
data = np.loadtxt('data.csv', delimiter=',', usecols=(0, 2))
```

**genfromtxt()** - Load text with missing values

```python
# Handles missing data better than loadtxt
data = np.genfromtxt('data.csv', delimiter=',')

# Missing values become NaN
data = np.genfromtxt('data.csv', delimiter=',', 
                     filling_values=np.nan)

# With names (creates structured array)
data = np.genfromtxt('data.csv', delimiter=',', 
                     names=True, dtype=None)
```

### Memory-Mapped Files

**memmap()** - Memory-mapped array (for very large files)

```python
# Create memory-mapped file
mmap = np.memmap('large_array.dat', dtype='float32', 
                 mode='w+', shape=(1000000, 100))

# Use like regular array, but stays on disk
mmap[0, :] = np.arange(100)
mmap.flush()  # Write to disk

# Load existing memory-mapped file
mmap = np.memmap('large_array.dat', dtype='float32', 
                 mode='r', shape=(1000000, 100))
```

---

## 17. Performance & Memory

### Vectorization

Always prefer vectorized operations over loops!

```python
# BAD - Python loop (slow)
arr = np.arange(1000000)
result = np.zeros(1000000)
for i in range(len(arr)):
    result[i] = arr[i] ** 2

# GOOD - Vectorized (fast)
arr = np.arange(1000000)
result = arr ** 2

# Vectorization can be 100x faster!
```

### Data Type Optimization

Choose the smallest dtype that fits your data:

```python
# Default (often wastes memory)
arr = np.array([1, 2, 3])  # int64 or int32

# Optimized
arr = np.array([1, 2, 3], dtype=np.int8)  # Only need -128 to 127

# For image data (0-255)
image = np.random.randint(0, 256, (1000, 1000), dtype=np.uint8)
# 1 byte per pixel instead of 8!

# Check memory usage
print(arr.nbytes)
```

### Copy vs View

Understanding when arrays share memory:

```python
arr = np.array([1, 2, 3, 4, 5])

# VIEW (shares memory - fast, no copy)
view = arr[1:4]
view[0] = 999
print(arr)  # [1 999 3 4 5] - changed!

# COPY (independent - slower, uses more memory)
copy = arr[1:4].copy()
copy[0] = 777
print(arr)  # [1 999 3 4 5] - unchanged

# Check if it's a view or copy
print(view.base is arr)  # True - it's a view
print(copy.base is arr)  # False - it's a copy

# Reshape/transpose usually create views
arr2d = np.arange(6).reshape(2, 3)
transposed = arr2d.T
print(transposed.base is arr2d)  # True - view
```

### Memory Layout

**ascontiguousarray()** - Ensure C-contiguous memory layout

```python
arr = np.arange(12).reshape(3, 4).T

# Check if C-contiguous
print(arr.flags['C_CONTIGUOUS'])  # False

# Make it C-contiguous (may copy)
arr_c = np.ascontiguousarray(arr)
print(arr_c.flags['C_CONTIGUOUS'])  # True

# Why? Better cache performance for certain operations
```

**asfortranarray()** - Ensure Fortran-contiguous layout

```python
arr = np.arange(12).reshape(3, 4)
arr_f = np.asfortranarray(arr)
print(arr_f.flags['F_CONTIGUOUS'])  # True
```

### Performance Tips

```python
# 1. Pre-allocate arrays
# BAD
result = []
for i in range(10000):
    result.append(i ** 2)
result = np.array(result)

# GOOD
result = np.empty(10000)
for i in range(10000):
    result[i] = i ** 2

# BEST
result = np.arange(10000) ** 2

# 2. Use in-place operations
arr = np.arange(1000000)
arr = arr + 1    # Creates new array
arr += 1         # Modifies in-place (faster)

# 3. Use broadcasting instead of loops
A = np.random.rand(1000, 1000)
b = np.random.rand(1000)
# Add b to each row of A
result = A + b[np.newaxis, :]  # Broadcasting
# Instead of looping over rows

# 4. Use built-in functions
arr = np.random.rand(1000000)
mean = np.mean(arr)      # Fast (C implementation)
# vs sum(arr) / len(arr)  # Slow (Python loop)
```

---

## 18. Lesser-Known but Powerful Functions

### einsum()

**np.einsum()** - Einstein summation (powerful for tensor operations)

```python
# Matrix multiplication
A = np.random.rand(3, 4)
B = np.random.rand(4, 5)
C = np.einsum('ij,jk->ik', A, B)  # Same as A @ B

# Transpose
A = np.random.rand(3, 4)
At = np.einsum('ij->ji', A)  # Same as A.T

# Trace (diagonal sum)
A = np.random.rand(5, 5)
trace = np.einsum('ii->', A)  # Same as np.trace(A)

# Batch matrix multiplication
A = np.random.rand(10, 3, 4)
B = np.random.rand(10, 4, 5)
C = np.einsum('bij,bjk->bik', A, B)

# Element-wise product then sum
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
result = np.einsum('i,i->', a, b)  # 32 (dot product)
```

### tensordot()

**np.tensordot()** - Tensor dot product

```python
# Like dot but for higher dimensions
A = np.random.rand(3, 4, 5)
B = np.random.rand(5, 6, 7)

# Contract over axis 2 of A and axis 0 of B
C = np.tensordot(A, B, axes=([2], [0]))
print(C.shape)  # (3, 4, 6, 7)

# Same as einsum:
C = np.einsum('ijk,klm->ijlm', A, B)
```

### meshgrid()

**np.meshgrid()** - Create coordinate matrices

```python
# 1D arrays
x = np.array([1, 2, 3])
y = np.array([4, 5])

# Create 2D grids
X, Y = np.meshgrid(x, y)
print(X)
# [[1 2 3]
#  [1 2 3]]

print(Y)
# [[4 4 4]
#  [5 5 5]]

# Use for plotting or computations
X, Y = np.meshgrid(np.linspace(-5, 5, 100), 
                   np.linspace(-5, 5, 100))
Z = np.sqrt(X**2 + Y**2)  # Distance from origin
```

### select()

**np.select()** - Choose values based on multiple conditions

```python
arr = np.array([0, 5, 10, 15, 20])

# Multiple conditions
conditions = [
    arr < 5,
    (arr >= 5) & (arr < 15),
    arr >= 15
]

choices = [
    'small',
    'medium',
    'large'
]

result = np.select(conditions, choices, default='unknown')
print(result)
# ['small' 'medium' 'medium' 'large' 'large']

# More flexible than np.where for multiple conditions
```

### piecewise()

**np.piecewise()** - Piecewise function

```python
x = np.linspace(-2, 2, 100)

# Define piecewise function
# f(x) = x^2 if x < 0
#        x    if 0 <= x < 1
#        1    if x >= 1

y = np.piecewise(x, 
                 [x < 0, (x >= 0) & (x < 1), x >= 1],
                 [lambda x: x**2, lambda x: x, 1])
```

### gradient()

**np.gradient()** - Numerical gradient (derivative)

```python
# 1D gradient
x = np.array([1, 4, 9, 16, 25])  # x^2 values
grad = np.gradient(x)
print(grad)  # Approximates derivative (2x)

# 2D gradient (returns list of gradients)
Z = np.random.rand(10, 10)
grad_y, grad_x = np.gradient(Z)

# With spacing
x = np.linspace(0, 10, 100)
y = np.sin(x)
dy_dx = np.gradient(y, x)  # Derivative of sin(x)
```

### diff()

**np.diff()** - Discrete differences

```python
arr = np.array([1, 4, 9, 16, 25])

# First difference
diff1 = np.diff(arr)
print(diff1)  # [3 5 7 9]

# Second difference
diff2 = np.diff(arr, n=2)
print(diff2)  # [2 2 2]

# Along axis
arr2d = np.array([[1, 2, 4], [8, 16, 32]])
diff = np.diff(arr2d, axis=1)
# [[1 2]
#  [8 16]]
```

### unwrap()

**np.unwrap()** - Unwrap phase angles

```python
# Useful for signal processing
phases = np.array([0, 0.5, 1.0, 1.5, 2.0, -2.8, -2.3])

# Phases wrap around at π or -π
unwrapped = np.unwrap(phases)
# Removes discontinuities by adding multiples of 2π

# Example with angles in radians
angles = np.array([0, 1, 2, 3, 4, 5, 6])
wrapped = np.mod(angles, 2 * np.pi)  # Wrap to [0, 2π)
unwrapped = np.unwrap(wrapped)       # Recover original
```

### clip()

**np.clip()** - Limit values to range

```python
arr = np.array([1, 5, 10, 15, 20])

# Clip to [5, 15]
clipped = np.clip(arr, 5, 15)
print(clipped)  # [5 5 10 15 15]

# Useful for image processing
image = np.random.rand(100, 100) * 300
image_clipped = np.clip(image, 0, 255)
```

### digitize()

**np.digitize()** - Bin values into discrete intervals

```python
values = np.array([0.5, 1.5, 2.5, 3.5, 4.5])
bins = np.array([1, 2, 3, 4])

# Find which bin each value belongs to
indices = np.digitize(values, bins)
print(indices)  # [0 1 2 3 4]
# 0.5 is before bin 1, 1.5 is in bin [1,2), etc.

# Useful for histograms and data binning
```

### histogram()

**np.histogram()** - Compute histogram

```python
data = np.random.randn(1000)

# Get histogram
counts, bin_edges = np.histogram(data, bins=10)

print(counts)      # Number of values in each bin
print(bin_edges)   # Edges of bins (length = bins + 1)

# Specify bin ranges
counts, bins = np.histogram(data, bins=[-3, -2, -1, 0, 1, 2, 3])

# Normalized histogram (density)
density, bins = np.histogram(data, bins=10, density=True)
```

---

## Section Summary

### NumPy Core Strengths

1. **Fast operations** - C/Fortran under the hood
2. **Broadcasting** - Automatic shape matching
3. **Memory efficient** - Compact storage
4. **Vectorization** - Avoid Python loops
5. **Rich ecosystem** - Integrates with SciPy, Pandas, Matplotlib

### Most Commonly Used Functions

**Creation:**
- `np.array()`, `np.zeros()`, `np.ones()`, `np.arange()`, `np.linspace()`

**Operations:**
- `+, -, *, /`, `**`, `@` (matrix multiply)

**Aggregations:**
- `sum()`, `mean()`, `std()`, `min()`, `max()`

**Indexing:**
- Boolean masking: `arr[arr > 5]`
- Fancy indexing: `arr[[0, 2, 4]]`

**Reshaping:**
- `reshape()`, `flatten()`, `T` (transpose)

**Stacking:**
- `concatenate()`, `vstack()`, `hstack()`

---

## Key Takeaways

### 1. Vectorization is Your Friend
```python
# Avoid loops whenever possible
# BAD
for i in range(len(arr)):
    arr[i] = arr[i] ** 2

# GOOD
arr = arr ** 2
```

### 2. Understand Broadcasting
```python
# Broadcasting saves memory and code
matrix = np.random.rand(100, 10)
mean = matrix.mean(axis=0)  # Shape (10,)
centered = matrix - mean     # Broadcasting!
```

### 3. Choose the Right Data Type
```python
# Default int64 may be overkill
small_ints = np.array([1, 2, 3], dtype=np.int8)
# 1 byte vs 8 bytes per element
```

### 4. Copy vs View Matters
```python
# Slice = view (fast but shares memory)
view = arr[1:5]

# Always copy when you need independence
independent = arr[1:5].copy()
```

### 5. Use Built-in Functions
```python
# NumPy functions are optimized
np.sum(arr)      # Fast (C implementation)
sum(arr)         # Slow (Python loop)
```

---

## Common NumPy Pitfalls

### 1. Modifying Views Unexpectedly
```python
arr = np.array([1, 2, 3, 4, 5])
subset = arr[1:4]
subset[0] = 999
# Surprise! arr is now [1, 999, 3, 4, 5]

# Fix: Use .copy()
subset = arr[1:4].copy()
```

### 2. Broadcasting Mistakes
```python
# Shape mismatch
arr1 = np.ones((3, 4))
arr2 = np.ones((3, 2))
# result = arr1 + arr2  # ERROR!

# Check shapes before operating
print(arr1.shape, arr2.shape)
```

### 3. Integer Division Truncates
```python
a = np.array([1, 2, 3])
b = a / 2  # [0.5, 1.0, 1.5] - float result

a = np.array([1, 2, 3])
b = a // 2  # [0, 1, 1] - integer division
```

### 4. NaN Propagation
```python
arr = np.array([1, 2, np.nan, 4])
print(np.mean(arr))  # nan (one NaN ruins everything)

# Use NaN-aware functions
print(np.nanmean(arr))  # 2.333... (ignores NaN)
```

### 5. Mixing Python Lists and Arrays
```python
# Avoid mixing in operations
arr = np.array([1, 2, 3])
# result = arr + [1, 2, 3]  # Works but slow

# Convert to array first
arr2 = np.array([1, 2, 3])
result = arr + arr2  # Fast
```

### 6. Forgetting Axis Parameter
```python
arr2d = np.array([[1, 2, 3], [4, 5, 6]])

np.sum(arr2d)         # 21 (all elements)
np.sum(arr2d, axis=0) # [5 7 9] (column sums)
np.sum(arr2d, axis=1) # [6 15] (row sums)

# Remember: axis=0 is "down", axis=1 is "across"
```

---

## Practice Recommendations

### Phase 1: Basics (Week 1)
- Create arrays in different ways
- Practice indexing and slicing
- Use basic math operations
- Reshape and transpose arrays

### Phase 2: Intermediate (Week 2)
- Master boolean masking
- Practice broadcasting
- Learn aggregations (mean, sum, etc.)
- Work with random data

### Phase 3: Advanced (Week 3+)
- Linear algebra operations
- Performance optimization
- einsum for complex operations
- Memory-efficient techniques

### Learning Strategy
1. **Read** the syntax
2. **Type** the examples (don't copy-paste!)
3. **Modify** the examples
4. **Create** your own use cases
5. **Repeat** with real data

---

## Suggested Mini Exercises

### Exercise 1: Array Creation
Create a 5x5 array with random integers from 1 to 100, then:
- Find mean of each row
- Find max of each column
- Replace all values > 50 with 50

### Exercise 2: Boolean Masking
Given an array of 100 random numbers:
- Extract all values between 0.3 and 0.7
- Count how many are greater than 0.5
- Replace negatives with 0

### Exercise 3: Broadcasting
Create a 10x10 grid where each cell contains the sum of its row and column indices.

### Exercise 4: Matrix Operations
Create two random 3x3 matrices:
- Multiply them
- Find determinant of each
- Solve Ax = b where b = [1, 2, 3]

### Exercise 5: Data Cleaning
Create an array with some NaN values:
- Count NaN values
- Replace NaN with column mean
- Find rows with no NaN values

### Exercise 6: Performance
Generate 1 million random numbers:
- Time a loop that squares each element
- Time a vectorized operation doing the same
- Compare execution times

### Exercise 7: Real-World
Simulate test scores for 100 students across 5 subjects:
- Calculate average score per student
- Find the subject with highest average
- Identify students with all scores > 70
- Normalize scores (mean=0, std=1)

---

## Resources for Further Learning

- **Official NumPy Documentation**: numpy.org/doc
- **NumPy Tutorial**: numpy.org/numpy-tutorials
- **Practice**: Project Euler problems
- **Books**: "Python for Data Analysis" by Wes McKinney
- **Interactive**: Jupyter notebooks for experimentation

---

**Remember**: NumPy proficiency comes from practice. Start with simple operations and gradually work towards complex data manipulations. Don't try to memorize everything - understand the patterns and use this guide as a reference!

Good luck with your NumPy journey! 🚀
