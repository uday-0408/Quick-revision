# Pandas DataFrame Reference Guide

A complete reference and practice guide for working with Pandas DataFrames in Python.

---

## 1. Creating DataFrames

### pd.DataFrame()
Create a DataFrame from various data structures.

**Parameters:**
- `data`: dict, list, ndarray, Series, or another DataFrame
- `index`: row labels
- `columns`: column labels
- `dtype`: data type to force

```python
import pandas as pd

# From dictionary
data = {'name': ['Alice', 'Bob', 'Charlie'], 
        'age': [25, 30, 35], 
        'city': ['NY', 'LA', 'SF']}
df = pd.DataFrame(data)

# From list of lists
data = [['Alice', 25], ['Bob', 30], ['Charlie', 35]]
df = pd.DataFrame(data, columns=['name', 'age'])

# With custom index
df = pd.DataFrame(data, index=['row1', 'row2', 'row3'])
```

### pd.read_csv()
Read data from CSV files - one of the most commonly used Pandas functions.

**Key Parameters:**
- `filepath_or_buffer`: Path to file or file-like object
- `sep`: Delimiter character (default ','). Use '\t' for tab, '\s+' for whitespace
- `header`: Row number(s) to use as column names (default 0). Use None if no header
- `names`: List of column names to use (useful when header=None)
- `index_col`: Column(s) to use as row labels (int, str, or list)
- `usecols`: Subset of columns to read (list of str or int)
- `dtype`: Data type for columns (dict like {'col': type})
- `na_values`: Additional strings to recognize as NaN (list or dict)
- `skiprows`: Number of lines to skip at start (int or list of ints)
- `nrows`: Number of rows to read (useful for large files)
- `encoding`: File encoding (e.g., 'utf-8', 'latin1', 'cp1252')
- `parse_dates`: Parse date columns (list of column names or positions)
- `converters`: Dict of functions for converting values in columns
- `chunksize`: Return iterator for reading file in chunks

```python
import pandas as pd

# Basic usage
df = pd.read_csv('data.csv')

# No header row - specify column names
df = pd.read_csv('data.csv', header=None, names=['col1', 'col2', 'col3'])

# Tab-separated file
df = pd.read_csv('data.tsv', sep='\t')

# Custom delimiter (pipe)
df = pd.read_csv('data.txt', sep='|')

# Set specific column as index
df = pd.read_csv('data.csv', index_col='id')

# Multiple columns as index
df = pd.read_csv('data.csv', index_col=['date', 'city'])

# Read only specific columns
df = pd.read_csv('data.csv', usecols=['name', 'age', 'city'])
df = pd.read_csv('data.csv', usecols=[0, 2, 4])  # By position

# Skip rows at the beginning
df = pd.read_csv('data.csv', skiprows=5)  # Skip first 5 rows
df = pd.read_csv('data.csv', skiprows=[0, 2, 5])  # Skip specific rows

# Read only first N rows (useful for testing)
df = pd.read_csv('data.csv', nrows=1000)

# Specify data types for better performance
df = pd.read_csv('data.csv', dtype={'age': 'int32', 'name': 'string'})

# Handle missing values
df = pd.read_csv('data.csv', na_values=['NA', 'missing', '?', '-'])
# Different NA values per column
df = pd.read_csv('data.csv', na_values={'age': ['unknown'], 'salary': [0, -1]})

# Parse date columns
df = pd.read_csv('data.csv', parse_dates=['date'])
df = pd.read_csv('data.csv', parse_dates=[['year', 'month', 'day']])  # Combine columns

# Handle encoding issues
df = pd.read_csv('data.csv', encoding='utf-8')
df = pd.read_csv('data.csv', encoding='latin1')

# Apply custom converters
df = pd.read_csv('data.csv', converters={'price': lambda x: float(x.replace('$', ''))})

# Read large files in chunks
for chunk in pd.read_csv('large_file.csv', chunksize=10000):
    process(chunk)  # Process each chunk

# Handle files with comments
df = pd.read_csv('data.csv', comment='#')

# Read compressed files
df = pd.read_csv('data.csv.gz')  # Automatically detects compression
df = pd.read_csv('data.csv.zip')

# Read from URL
df = pd.read_csv('https://example.com/data.csv')

# Use thousands separator
df = pd.read_csv('data.csv', thousands=',')

# Handle decimal separator (European format)
df = pd.read_csv('data.csv', decimal=',')
```

### pd.read_excel()
Read data from Excel files.

```python
# Basic usage
df = pd.read_excel('data.xlsx')

# Specify sheet
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Multiple sheets
dfs = pd.read_excel('data.xlsx', sheet_name=['Sheet1', 'Sheet2'])
```

### pd.read_sql()
Read data from SQL database.

```python
import sqlite3

# Create connection
conn = sqlite3.connect('database.db')

# Read entire table
df = pd.read_sql('SELECT * FROM table_name', conn)

# With query
df = pd.read_sql('SELECT * FROM users WHERE age > 25', conn)

# Using table name directly
df = pd.read_sql_table('table_name', conn)
```

### pd.DataFrame.from_dict()
Create DataFrame from dictionary with orientation control.

```python
# Default: keys as columns
data = {'A': [1, 2, 3], 'B': [4, 5, 6]}
df = pd.DataFrame.from_dict(data)

# Keys as rows (orient='index')
data = {'row1': [1, 2, 3], 'row2': [4, 5, 6]}
df = pd.DataFrame.from_dict(data, orient='index', columns=['A', 'B', 'C'])

# Nested dictionary
data = {'row1': {'A': 1, 'B': 2}, 'row2': {'A': 3, 'B': 4}}
df = pd.DataFrame.from_dict(data, orient='index')
```

### pd.DataFrame.from_records()
Create DataFrame from structured ndarray, list of tuples, or dicts.

```python
# From list of tuples
records = [('Alice', 25, 'NY'), ('Bob', 30, 'LA')]
df = pd.DataFrame.from_records(records, columns=['name', 'age', 'city'])

# From list of dictionaries
records = [{'name': 'Alice', 'age': 25}, {'name': 'Bob', 'age': 30}]
df = pd.DataFrame.from_records(records)
```

---

## 2. Viewing and Inspecting Data

### df.head() and df.tail()
View first or last n rows.

```python
# First 5 rows (default)
df.head()

# First 10 rows
df.head(10)

# Last 5 rows
df.tail()

# Last 3 rows
df.tail(3)
```

### df.info()
Display concise summary of DataFrame.

```python
df.info()

# Output shows:
# - Index dtype and range
# - Column names, non-null counts, and dtypes
# - Memory usage
```

### df.describe()
Generate descriptive statistics - essential for data exploration.

**Key Parameters:**
- `percentiles`: List of percentiles to include (default [.25, .5, .75])
- `include`: 'all', list of dtypes, or None (default numeric only)
- `exclude`: List of dtypes to exclude
- `datetime_is_numeric`: Treat datetime as numeric (bool)

**Default Statistics:**
- Numeric: count, mean, std, min, 25%, 50%, 75%, max
- Object/Category: count, unique, top (most frequent), freq (frequency of top)

```python
import pandas as pd
import numpy as np

# Numeric columns only (default)
df.describe()
# Shows: count, mean, std, min, 25%, 50%, 75%, max

# All columns (numeric and categorical)
df.describe(include='all')
# Numeric: count, mean, std, min, 25%, 50%, 75%, max
# Object: count, unique, top, freq

# Only categorical/object columns
df.describe(include=['object'])
df.describe(include=['object', 'category'])
# Shows: count, unique, top, freq

# Only numeric columns (explicit)
df.describe(include=[np.number])
df.describe(include=['int64', 'float64'])

# Exclude certain types
df.describe(exclude=['object'])  # All except strings
df.describe(exclude=['float64'])  # All except floats

# Custom percentiles
df.describe(percentiles=[.1, .25, .5, .75, .9, .95, .99])
# Shows 10th, 25th, 50th, 75th, 90th, 95th, 99th percentiles

df.describe(percentiles=[.5])  # Only median
df.describe(percentiles=[])  # No percentiles

# Single column
df['age'].describe()
df['salary'].describe(percentiles=[.1, .5, .9])

# Transpose for easier reading
df.describe().T  # Columns become rows

# Format output
print(df.describe().round(2))  # Round to 2 decimals
print(df.describe().to_string())  # Full precision

# Save to DataFrame for further analysis
stats = df.describe()
print(f"Mean age: {stats.loc['mean', 'age']}")
print(f"Max salary: {stats.loc['max', 'salary']}")

# Describe specific columns
df[['age', 'salary', 'score']].describe()

# Describe after groupby
df.groupby('city')['age'].describe()
df.groupby('department')[['salary', 'age']].describe()

# Add custom statistics
def custom_describe(series):
    return pd.Series({
        'count': series.count(),
        'mean': series.mean(),
        'median': series.median(),
        'mode': series.mode()[0] if len(series.mode()) > 0 else np.nan,
        'std': series.std(),
        'var': series.var(),
        'min': series.min(),
        'max': series.max(),
        'range': series.max() - series.min(),
        'iqr': series.quantile(0.75) - series.quantile(0.25),
        'skew': series.skew(),
        'kurtosis': series.kurtosis()
    })

df['salary'].pipe(custom_describe)

# Categorical analysis
cat_desc = df.describe(include=['object'])
print(f"Most common city: {cat_desc.loc['top', 'city']}")
print(f"Frequency: {cat_desc.loc['freq', 'city']}")
print(f"Unique cities: {cat_desc.loc['unique', 'city']}")
```

### df.shape
Return tuple of (rows, columns).

```python
rows, cols = df.shape
print(f"DataFrame has {rows} rows and {cols} columns")
```

### df.dtypes
Return data types of each column.

```python
# View all column types
print(df.dtypes)

# Filter columns by type
numeric_cols = df.select_dtypes(include=['int64', 'float64']).columns
object_cols = df.select_dtypes(include=['object']).columns
```

### df.columns and df.index
Access column and row labels.

```python
# Column names
print(df.columns)
print(df.columns.tolist())

# Index
print(df.index)

# Convert to list
col_list = df.columns.tolist()
idx_list = df.index.tolist()
```

### Value Counts and Top/Bottom Values
Count occurrences and find extreme values - frequently used in analysis.

```python
import pandas as pd

# value_counts() - count unique values in a column
df['city'].value_counts()  # Count of each city
df['category'].value_counts()  # Frequency distribution

# Include NaN in counts
df['status'].value_counts(dropna=False)

# Normalize to get proportions/percentages
df['category'].value_counts(normalize=True)  # Returns proportions (0-1)
df['category'].value_counts(normalize=True) * 100  # Percentages

# Sort by index instead of counts
df['city'].value_counts(sort=False)  # Alphabetical order

# Sort ascending (least to most common)
df['city'].value_counts(ascending=True)

# Get top N most common values
df['product'].value_counts().head(10)  # Top 10 products
df['city'].value_counts().nlargest(5)  # Top 5 cities

# Bins for continuous data
df['age'].value_counts(bins=5)  # Create 5 bins
df['salary'].value_counts(bins=[0, 30000, 60000, 90000, 120000])  # Custom bins

# nlargest() - get rows with largest values
df.nlargest(10, 'salary')  # Top 10 highest salaries
df.nlargest(5, 'age')  # 5 oldest people
df.nlargest(3, 'score', keep='all')  # Include ties

# Multiple columns
df.nlargest(10, ['salary', 'age'])  # Top 10 by salary, then age

# nsmallest() - get rows with smallest values
df.nsmallest(10, 'age')  # 10 youngest people
df.nsmallest(5, 'price')  # 5 cheapest items
df.nsmallest(20, 'distance')  # 20 closest locations

# Keep parameter for ties
df.nlargest(5, 'score', keep='first')  # First occurrence (default)
df.nlargest(5, 'score', keep='last')  # Last occurrence
df.nlargest(5, 'score', keep='all')  # Keep all ties

# Combine with other operations
top_cities = df['city'].value_counts().head(10).index.tolist()
df_filtered = df[df['city'].isin(top_cities)]

# Value counts with multiple columns
df.groupby(['city', 'category']).size()  # Counts for combinations
df.value_counts(['city', 'category'])  # Same as above (pandas 1.1+)

# Practical examples
# Most common values
print(f"Most common category: {df['category'].value_counts().index[0]}")
print(f"Frequency: {df['category'].value_counts().iloc[0]}")

# Percentage distribution
print(df['status'].value_counts(normalize=True).mul(100).round(2))

# Find rare categories (less than 1%)
value_pcts = df['category'].value_counts(normalize=True)
rare = value_pcts[value_pcts < 0.01]
print(f"Rare categories: {rare.index.tolist()}")
```

### df.sample()
Return random sample of rows - useful for testing and sampling.

**Key Parameters:**
- `n`: Number of items to return (int)
- `frac`: Fraction of items to return (float, 0 to 1)
- `replace`: Sample with replacement (bool, default False)
- `weights`: Probability weights for sampling (str or array-like)
- `random_state`: Seed for reproducibility (int)
- `axis`: 0 for rows (default), 1 for columns

```python
import pandas as pd

# Random single row
df.sample()  # Same as df.sample(n=1)

# Multiple random rows
df.sample(n=5)
df.sample(n=100)

# Percentage of rows
df.sample(frac=0.1)  # 10% of rows
df.sample(frac=0.5)  # 50% of rows
df.sample(frac=1.0)  # All rows (shuffled)

# Sample with replacement (can get duplicates)
df.sample(n=10, replace=True)
df.sample(frac=1.5, replace=True)  # 150% of data (with duplicates)

# Reproducible sampling (same result each time)
df.sample(n=5, random_state=42)
df.sample(frac=0.2, random_state=123)

# Weighted sampling (higher weight = more likely)
weights = df['importance']  # Use column as weights
df.sample(n=10, weights=weights)

# Custom weights
weights = [0.1, 0.2, 0.3, 0.4]  # Must match DataFrame length
df.sample(n=2, weights=weights)

# Sample columns instead of rows
df.sample(n=3, axis=1)  # Random 3 columns

# Train/test split using sample
train = df.sample(frac=0.8, random_state=42)
test = df.drop(train.index)
print(f"Train size: {len(train)}, Test size: {len(test)}")

# Stratified sampling by group
def stratified_sample(group, n=5):
    return group.sample(n=min(len(group), n))

sampled = df.groupby('category').apply(stratified_sample).reset_index(drop=True)

# Bootstrap sampling (for statistical analysis)
bootstrap_samples = [df.sample(frac=1.0, replace=True) for _ in range(1000)]

# Random sample for quick data exploration
print(df.sample(n=10))  # Quick look at random records
```

---

## 3. Selection and Indexing

### Column Selection
Select single or multiple columns.

```python
# Single column (returns Series)
df['name']

# Multiple columns (returns DataFrame)
df[['name', 'age']]

# Using dot notation (if column name is valid identifier)
df.name
```

### df.loc[]
Label-based indexing - accesses rows and columns by their labels/names.

**Syntax:** `df.loc[row_indexer, column_indexer]`

**Row Indexer Options:**
- Single label: 'row1'
- List of labels: ['row1', 'row2']
- Slice: 'row1':'row5' (inclusive of endpoint)
- Boolean array: df['age'] > 25
- Callable: lambda df: df['age'] > 25

**Column Indexer Options:**
- Same as row indexer (single, list, slice, boolean)
- Use ':' to select all columns

```python
# Single row by label
row = df.loc['row1']  # Returns Series

# Multiple rows (specific labels)
df.loc[['row1', 'row2', 'row5']]  # Returns DataFrame

# Slice rows (INCLUSIVE endpoint)
df.loc['row1':'row5']  # Includes both row1 and row5

# Single value (row and column)
value = df.loc['row1', 'name']  # Returns scalar

# Multiple rows, single column
df.loc[['row1', 'row2'], 'name']  # Returns Series

# Multiple rows and columns
df.loc[['row1', 'row2'], ['name', 'age']]  # Returns DataFrame

# All rows, specific columns
df.loc[:, ['name', 'age']]

# Slice columns too
df.loc['row1':'row3', 'name':'age']

# Boolean indexing (most common use case)
df.loc[df['age'] > 25]  # All columns where age > 25
df.loc[df['age'] > 25, ['name', 'city']]  # Specific columns

# Multiple conditions
df.loc[(df['age'] > 25) & (df['city'] == 'NY')]
df.loc[(df['age'] < 20) | (df['age'] > 60), 'name']

# Set values using loc
df.loc['row1', 'age'] = 30
df.loc[df['age'] < 18, 'category'] = 'minor'
df.loc[df['city'] == 'NY', ['bonus', 'status']] = [1000, 'active']

# Using callable (function)
df.loc[lambda df: df['age'] > df['age'].mean()]

# Negate condition
df.loc[~df['city'].isin(['NY', 'LA'])]

# Select rows with string methods
df.loc[df['name'].str.startswith('A')]
df.loc[df['email'].str.contains('@gmail.com')]
```

### df.iloc[]
Integer position-based indexing - accesses rows and columns by their integer positions.

**Syntax:** `df.iloc[row_indexer, column_indexer]`

**Indexer Options:**
- Single integer: 0, -1 (negative indexing supported)
- List of integers: [0, 2, 4]
- Slice: 0:5 (exclusive of endpoint, like Python lists)
- Boolean array (must be same length)
- Callable function

```python
# Single row by position
row = df.iloc[0]  # First row (Returns Series)
row = df.iloc[-1]  # Last row

# Multiple rows (specific positions)
df.iloc[[0, 2, 4]]  # 1st, 3rd, 5th rows
df.iloc[[0, -1]]  # First and last rows

# Slice rows (EXCLUSIVE endpoint)
df.iloc[0:5]  # First 5 rows (0 through 4)
df.iloc[5:10]  # Rows 5 through 9
df.iloc[-5:]  # Last 5 rows
df.iloc[:10]  # First 10 rows
df.iloc[::2]  # Every other row

# Single value (row and column position)
value = df.iloc[0, 1]  # First row, second column
value = df.iloc[-1, -1]  # Last row, last column

# Multiple rows, single column
df.iloc[[0, 1, 2], 0]  # First 3 rows, first column
df.iloc[:5, -1]  # First 5 rows, last column

# Multiple rows and columns
df.iloc[0:3, 0:2]  # First 3 rows, first 2 columns
df.iloc[[0, 2], [1, 3]]  # Specific rows and columns

# All rows, specific columns
df.iloc[:, [0, 2, 4]]  # All rows, columns 0, 2, 4
df.iloc[:, 0:3]  # All rows, first 3 columns
df.iloc[:, -3:]  # All rows, last 3 columns

# Advanced slicing
df.iloc[::2, :]  # Every other row, all columns
df.iloc[:, ::2]  # All rows, every other column
df.iloc[1:10:2, :]  # Rows 1,3,5,7,9

# Set values using iloc
df.iloc[0, 1] = 100
df.iloc[0:5, 0] = 999  # Set first column for first 5 rows
df.iloc[-1, :] = df.iloc[-2, :]  # Copy second last row to last row

# Boolean indexing (array must match length)
import numpy as np
mask = np.array([True, False, True, False, True])
df.iloc[mask]  # Select rows where mask is True

# Combine with conditions (requires numpy)
df.iloc[np.where(df['age'] > 25)[0]]

# Get specific rows excluding some
df.iloc[df.index.difference([2, 5, 7])]  # All rows except 2, 5, 7

# Random sample by position
import random
positions = random.sample(range(len(df)), 5)
df.iloc[positions]
```

### df.at[] and df.iat[]
Fast scalar access.

```python
# at: label-based (faster than loc for single value)
value = df.at['row1', 'name']
df.at['row1', 'name'] = 'Alice'

# iat: position-based (faster than iloc for single value)
value = df.iat[0, 0]
df.iat[0, 0] = 'Alice'
```

---

## 4. Filtering and Querying

### Boolean Filtering
Filter rows using boolean conditions - one of the most common operations.

**Operators:**
- Comparison: `<`, `>`, `<=`, `>=`, `==`, `!=`
- Logical: `&` (AND), `|` (OR), `~` (NOT)
- Methods: `isin()`, `between()`, `str.contains()`, `str.startswith()`, etc.

**Important:** Use parentheses around each condition when combining with `&` or `|`

```python
import pandas as pd
import numpy as np

# Single condition
df[df['age'] > 25]
df[df['age'] >= 18]
df[df['city'] == 'NY']
df[df['status'] != 'inactive']

# Multiple conditions with AND (&)
df[(df['age'] > 25) & (df['city'] == 'NY')]
df[(df['age'] >= 18) & (df['age'] <= 65) & (df['status'] == 'active')]

# Multiple conditions with OR (|)
df[(df['age'] < 20) | (df['age'] > 60)]
df[(df['city'] == 'NY') | (df['city'] == 'LA') | (df['city'] == 'SF')]

# NOT condition (~)
df[~(df['city'] == 'NY')]  # Not equal to NY
df[~df['status'].isin(['deleted', 'archived'])]  # Not in list

# Combine AND, OR, NOT
df[(df['age'] > 18) & ((df['city'] == 'NY') | (df['city'] == 'LA'))]
df[((df['age'] < 25) | (df['age'] > 65)) & ~(df['status'] == 'inactive')]

# Using isin() - check if value in list
df[df['city'].isin(['NY', 'LA', 'SF'])]
df[df['age'].isin([18, 21, 25, 30])]
df[~df['status'].isin(['deleted', 'archived'])]  # Not in list

# Using between() - inclusive range
df[df['age'].between(25, 35)]  # 25 <= age <= 35
df[df['age'].between(18, 65, inclusive='neither')]  # 18 < age < 65
df[df['age'].between(18, 65, inclusive='left')]  # 18 <= age < 65
df[df['age'].between(18, 65, inclusive='right')]  # 18 < age <= 65

# String operations
df[df['name'].str.contains('John')]  # Contains substring
df[df['name'].str.contains('john', case=False)]  # Case insensitive
df[df['email'].str.contains('@gmail.com')]

df[df['name'].str.startswith('A')]  # Starts with
df[df['name'].str.endswith('son')]  # Ends with

df[df['name'].str.match(r'^[A-M]')]  # Regex match (starts with A-M)
df[df['phone'].str.match(r'^\d{3}-\d{3}-\d{4}$')]  # Phone format

# Handle NaN in string operations
df[df['name'].str.contains('John', na=False)]  # Treat NaN as False

# String length
df[df['name'].str.len() > 10]  # Name longer than 10 characters
df[df['description'].str.len() < 100]

# Null/missing value conditions
df[df['age'].isna()]  # Rows where age is NaN
df[df['age'].notna()]  # Rows where age is not NaN
df[df['salary'].isnull()]  # Alias for isna()
df[df['salary'].notnull()]  # Alias for notna()

# Multiple columns with missing values
df[df[['age', 'salary']].isna().any(axis=1)]  # Any column is NaN
df[df[['age', 'salary']].isna().all(axis=1)]  # All columns are NaN
df[df[['age', 'salary']].notna().all(axis=1)]  # No NaNs in these columns

# Numeric comparisons
df[df['salary'] > df['salary'].mean()]  # Above average salary
df[df['age'] > df['age'].median()]  # Above median age
df[df['score'] > df.groupby('category')['score'].transform('mean')]  # Above group mean

# Comparing two columns
df[df['salary'] > df['expected_salary']]
df[df['start_date'] < df['end_date']]
df[df['actual_amount'] != df['expected_amount']]

# Complex numeric conditions
df[df['salary'] > df['salary'].quantile(0.75)]  # Top 25%
df[(df['age'] >= 18) & (df['salary'] < 50000)]

# Date/time filtering
df['date'] = pd.to_datetime(df['date'])
df[df['date'] > '2023-01-01']
df[df['date'].dt.year == 2023]
df[df['date'].dt.month.isin([6, 7, 8])]  # Summer months
df[df['date'].dt.dayofweek < 5]  # Weekdays only (0=Mon, 6=Sun)

# Date ranges
df[df['date'].between('2023-01-01', '2023-12-31')]
df[(df['date'] >= '2023-01-01') & (df['date'] <= '2023-12-31')]

# Time-based conditions
df[df['timestamp'].dt.hour.between(9, 17)]  # Business hours
df[df['date'].dt.is_month_start]  # First day of month
df[df['date'].dt.is_quarter_end]  # Last day of quarter

# Using query() method (alternative syntax)
df.query('age > 25')
df.query('age > 25 and city == "NY"')
df.query('city in ["NY", "LA", "SF"]')

# Query with variables
min_age = 25
max_age = 65
df.query('age >= @min_age and age <= @max_age')

cities = ['NY', 'LA', 'SF']
df.query('city in @cities')

# Conditional column selection
df.loc[df['age'] > 25, ['name', 'age', 'city']]  # Filter rows, select columns
df.loc[df['status'] == 'active', 'salary']  # Returns Series

# Filter and create copy (avoid SettingWithCopyWarning)
filtered_df = df[df['age'] > 25].copy()

# Filter with custom function
def is_valid_email(email):
    return '@' in str(email) and '.' in str(email)

df[df['email'].apply(is_valid_email)]

# Filter using numpy
df[np.logical_and(df['age'] > 25, df['salary'] > 50000)]
df[np.logical_or(df['city'] == 'NY', df['city'] == 'LA')]

# Outlier detection
mean = df['salary'].mean()
std = df['salary'].std()
df[np.abs(df['salary'] - mean) < 2 * std]  # Within 2 standard deviations

# Using where() for conditional selection
df.where(df['age'] > 25)  # Keeps shape, fills with NaN
df.where(df['age'] > 25).dropna()  # Then drop NaN rows

# Multiple DataFrames - filter one based on another
active_ids = df2[df2['status'] == 'active']['id'].unique()
df[df['user_id'].isin(active_ids)]

# Filter with percentage/quantile
top_10_percent = df['score'].quantile(0.9)
df[df['score'] >= top_10_percent]

# Complex example: Multiple conditions
filtered = df[
    (df['age'].between(25, 65)) &
    (df['city'].isin(['NY', 'LA', 'SF'])) &
    (df['salary'] > 50000) &
    (df['status'] == 'active') &
    (df['name'].str.len() > 3) &
    (df['email'].notna())
]

# Save boolean mask for reuse
mask = (df['age'] > 25) & (df['city'] == 'NY')
ny_adults = df[mask]
count = mask.sum()  # Number of True values
```

### df.query()
Query using string expression.

```python
# Simple condition
df.query('age > 25')

# Multiple conditions
df.query('age > 25 and city == "NY"')
df.query('age > 25 or age < 20')

# Using @ for variables
min_age = 25
max_age = 35
df.query('age >= @min_age and age <= @max_age')

# IN expression
cities = ['NY', 'LA']
df.query('city in @cities')

# NOT IN
df.query('city not in @cities')

# String methods
df.query('name.str.startswith("A")', engine='python')
```

---

## 5. Modifying Data

### Adding/Updating Columns

```python
# Add new column with scalar value
df['country'] = 'USA'

# Add calculated column
df['age_squared'] = df['age'] ** 2

# Add column based on condition
df['is_adult'] = df['age'] >= 18

# Using assign (returns new DataFrame)
df = df.assign(
    age_doubled=lambda x: x['age'] * 2,
    name_upper=lambda x: x['name'].str.upper()
)

# Insert at specific position
df.insert(1, 'new_col', [1, 2, 3])
```

### df.drop()
Remove rows or columns - essential for data cleaning.

**Key Parameters:**
- `labels`: Index or column labels to drop (single label or list)
- `axis`: 0 or 'index' for rows, 1 or 'columns' for columns
- `index`: Alternative to using labels with axis=0
- `columns`: Alternative to using labels with axis=1
- `level`: For MultiIndex, level to use (int or level name)
- `inplace`: Modify DataFrame in place (bool)
- `errors`: 'raise' (default) or 'ignore' if label not found

```python
import pandas as pd

# Drop single column
df.drop('column_name', axis=1)
df.drop(columns='column_name')  # More explicit

# Drop multiple columns
df.drop(['col1', 'col2', 'col3'], axis=1)
df.drop(columns=['col1', 'col2', 'col3'])

# Drop columns by position (use iloc to get names first)
cols_to_drop = df.columns[[0, 2, 4]]  # Drop 1st, 3rd, 5th columns
df.drop(columns=cols_to_drop)

# Drop single row by label
df.drop('row1', axis=0)
df.drop(index='row1')  # More explicit

# Drop multiple rows by label
df.drop(['row1', 'row2', 'row5'], axis=0)
df.drop(index=['row1', 'row2', 'row5'])

# Drop rows by position
df.drop(df.index[0])  # Drop first row
df.drop(df.index[-1])  # Drop last row
df.drop(df.index[[0, 2, 4]])  # Drop specific positions
df.drop(df.index[0:5])  # Drop first 5 rows

# Drop rows and columns simultaneously
df.drop(index=['row1'], columns=['col1'])

# Inplace modification (changes original DataFrame)
df.drop('column_name', axis=1, inplace=True)
df.drop(columns=['col1', 'col2'], inplace=True)

# Ignore errors if label doesn't exist
df.drop(['col1', 'nonexistent_col'], axis=1, errors='ignore')
# Without errors='ignore', would raise KeyError

# Drop based on condition
# Drop rows where age < 18
df.drop(df[df['age'] < 18].index)

# Drop rows with missing values in specific column
df.drop(df[df['salary'].isna()].index)

# Drop duplicate rows (see drop_duplicates for more control)
df.drop_duplicates()

# Drop columns with all NaN
df.drop(columns=df.columns[df.isna().all()])

# Drop columns with any NaN
df.drop(columns=df.columns[df.isna().any()])

# Drop columns by data type
df.drop(columns=df.select_dtypes(include=['object']).columns)  # Drop string columns
df.drop(columns=df.select_dtypes(include=[np.number]).columns)  # Drop numeric columns

# Drop columns with specific string pattern
import re
cols_to_drop = [col for col in df.columns if re.search(r'temp|test', col, re.I)]
df.drop(columns=cols_to_drop)

# Drop columns that start with/end with specific text
df.drop(columns=[col for col in df.columns if col.startswith('tmp_')])
df.drop(columns=[col for col in df.columns if col.endswith('_old')])

# Drop rows with specific value
df.drop(df[df['status'] == 'deleted'].index)

# Drop rows outside a range
df.drop(df[(df['age'] < 18) | (df['age'] > 100)].index)

# Drop rows with outliers (using IQR method)
Q1 = df['salary'].quantile(0.25)
Q3 = df['salary'].quantile(0.75)
IQR = Q3 - Q1
outliers = df[(df['salary'] < Q1 - 1.5 * IQR) | (df['salary'] > Q3 + 1.5 * IQR)].index
df.drop(outliers)

# MultiIndex: Drop by level
df.drop('key1', level=0)  # Drop from first level of MultiIndex
df.drop('key2', level='level_name')  # Drop by level name

# Drop and reassign (common pattern)
df = df.drop(columns=['temp_col1', 'temp_col2'])

# Chain multiple drops
df = (df
    .drop(columns=['col1', 'col2'])
    .drop(df[df['age'] < 0].index)
    .drop_duplicates()
)

# Drop based on column value counts
# Drop rows where category appears less than 10 times
value_counts = df['category'].value_counts()
rare_categories = value_counts[value_counts < 10].index
df.drop(df[df['category'].isin(rare_categories)].index)

# Drop columns with low variance (almost constant)
variance = df.var()
low_var_cols = variance[variance < 0.01].index
df.drop(columns=low_var_cols)

# Drop highly correlated columns
corr_matrix = df.corr().abs()
upper_triangle = corr_matrix.where(
    np.triu(np.ones(corr_matrix.shape), k=1).astype(bool)
)
to_drop = [col for col in upper_triangle.columns if any(upper_triangle[col] > 0.95)]
df.drop(columns=to_drop)

# Store dropped data before removal
dropped_rows = df[df['age'] < 18].copy()
df = df.drop(df[df['age'] < 18].index)
```

### df.rename()
Rename columns or index labels.

```python
# Rename columns
df.rename(columns={'old_name': 'new_name'})
df.rename(columns={'name': 'full_name', 'age': 'years'})

# Rename index
df.rename(index={'row1': 'first_row'})

# Using function
df.rename(columns=str.upper)
df.rename(columns=lambda x: x.replace('_', ' '))

# Inplace
df.rename(columns={'old': 'new'}, inplace=True)
```

### set_index() and reset_index()
Manage DataFrame index.

```python
# Set column as index
df.set_index('name')

# Set multiple columns as index
df.set_index(['name', 'city'])

# Keep the column
df.set_index('name', drop=False)

# Reset index to default integer index
df.reset_index()

# Drop the old index
df.reset_index(drop=True)

# Inplace operations
df.set_index('name', inplace=True)
df.reset_index(inplace=True)
```

---

## 6. Aggregation and Statistics

### Basic Statistics

```python
# Column-wise statistics
df.mean()      # Mean of numeric columns
df.sum()       # Sum
df.min()       # Minimum
df.max()       # Maximum
df.median()    # Median
df.std()       # Standard deviation
df.var()       # Variance
df.count()     # Count non-null values

# Row-wise statistics
df.mean(axis=1)
df.sum(axis=1)

# Specific column
df['age'].mean()
df['age'].sum()
```

### df.groupby()
Group data and apply aggregations - extremely powerful for analysis.

**Key Parameters:**
- `by`: Column(s) to group by (str, list, or array)
- `axis`: 0 for index (default), 1 for columns
- `level`: Level(s) to group by if axis is MultiIndex
- `as_index`: Return group labels as index (default True)
- `sort`: Sort group keys (default True)
- `group_keys`: Add group keys to index when calling apply (default True)
- `dropna`: Drop NA values from group keys (default True)

```python
import pandas as pd
import numpy as np

# Simple groupby - single column
df.groupby('city').mean()  # Mean of all numeric columns
df.groupby('city')['age'].mean()  # Mean of specific column
df.groupby('city')['age'].sum()  # Sum by group

# Multiple grouping columns (hierarchical grouping)
df.groupby(['city', 'gender']).mean()
df.groupby(['city', 'department', 'year']).sum()

# Group without setting as index
df.groupby('city', as_index=False).mean()  # City becomes regular column

# Group without sorting (faster for large datasets)
df.groupby('city', sort=False).mean()

# Single aggregation function
df.groupby('city')['age'].mean()
df.groupby('city')['salary'].sum()
df.groupby('city')['name'].count()  # Count non-null values
df.groupby('city').size()  # Count including nulls

# Multiple aggregations on single column
df.groupby('city')['age'].agg(['mean', 'min', 'max', 'std', 'count'])
df.groupby('city')['salary'].agg(['sum', 'mean', 'median'])

# Named aggregations (cleaner column names)
df.groupby('city')['age'].agg(
    avg_age='mean',
    min_age='min',
    max_age='max',
    total_people='count'
)

# Different functions for different columns
df.groupby('city').agg({
    'age': ['mean', 'max', 'min'],
    'salary': ['sum', 'mean'],
    'name': ['count', 'nunique']  # nunique = number of unique values
})

# Named aggregations for multiple columns (cleaner)
df.groupby('city').agg(
    avg_age=('age', 'mean'),
    max_age=('age', 'max'),
    total_salary=('salary', 'sum'),
    avg_salary=('salary', 'mean'),
    num_employees=('name', 'count')
)

# Custom aggregation functions
df.groupby('city')['age'].agg(lambda x: x.max() - x.min())  # Range
df.groupby('city')['salary'].agg(lambda x: x.quantile(0.75))  # 75th percentile

# Multiple custom functions with names
df.groupby('city')['age'].agg([
    ('range', lambda x: x.max() - x.min()),
    ('median', 'median'),
    ('q75', lambda x: x.quantile(0.75))
])

# Using built-in functions
df.groupby('city')['age'].agg([np.mean, np.std, np.median])

# Transform (return same shape as input)
df['age_normalized'] = df.groupby('city')['age'].transform(lambda x: (x - x.mean()) / x.std())
df['pct_of_group_total'] = df.groupby('city')['salary'].transform(lambda x: x / x.sum())

# Filter groups based on group properties
df.groupby('city').filter(lambda x: len(x) > 10)  # Keep groups with >10 rows
df.groupby('city').filter(lambda x: x['age'].mean() > 30)  # Groups where avg age > 30

# Apply custom function to each group
def top_n_salaries(group, n=3):
    return group.nlargest(n, 'salary')

df.groupby('city').apply(top_n_salaries, n=2)  # Top 2 salaries per city

# Get descriptive statistics per group
df.groupby('city')['age'].describe()

# Iterate over groups
for group_name, group_data in df.groupby('city'):
    print(f"Group: {group_name}")
    print(f"Size: {len(group_data)}")
    print(f"Mean age: {group_data['age'].mean()}\n")

# Iterate over multiple grouping columns
for (city, gender), group in df.groupby(['city', 'gender']):
    print(f"City: {city}, Gender: {gender}")
    print(f"Count: {len(group)}\n")

# Get specific group
ny_data = df.groupby('city').get_group('NY')
male_ny = df.groupby(['city', 'gender']).get_group(('NY', 'Male'))

# Get group keys
groups_dict = dict(list(df.groupby('city')))
print(groups_dict.keys())  # See all group names

# Group by multiple columns and reset index
result = df.groupby(['city', 'gender'], as_index=False).agg({
    'age': 'mean',
    'salary': 'sum'
})

# Aggregate with value counts
df.groupby('city')['product'].value_counts()

# First and last records per group
df.groupby('city').first()  # First row of each group
df.groupby('city').last()   # Last row of each group
df.groupby('city').head(3)  # First 3 rows of each group
df.groupby('city').tail(2)  # Last 2 rows of each group

# Cumulative operations within groups
df['cumsum_salary'] = df.groupby('city')['salary'].cumsum()
df['rank_in_city'] = df.groupby('city')['salary'].rank(ascending=False)

# Handle missing values in groupby
df.groupby('city', dropna=False).mean()  # Include NaN in groups (pandas 1.1+)
```

### df.agg()
Aggregate using multiple functions.

```python
# Single column, multiple functions
df['age'].agg(['mean', 'median', 'std'])

# Multiple columns, same functions
df[['age', 'salary']].agg(['mean', 'median'])

# Different functions for different columns
df.agg({
    'age': ['min', 'max', 'mean'],
    'salary': ['sum', 'mean'],
    'name': ['count']
})

# Custom function
df.agg({
    'age': lambda x: x.max() - x.min(),
    'salary': 'sum'
})
```

### df.pivot_table()
Create spreadsheet-style pivot table - powerful for data summarization.

**Key Parameters:**
- `values`: Column(s) to aggregate (str or list)
- `index`: Column(s) to group by on rows (str, list, or array)
- `columns`: Column(s) to group by on columns (str, list, or array)
- `aggfunc`: Aggregation function(s) - 'mean', 'sum', 'count', list, or dict
- `fill_value`: Value to replace missing values (scalar)
- `margins`: Add row/column totals (bool, default False)
- `margins_name`: Name of margin row/column (str, default 'All')
- `dropna`: Exclude NaN from group keys (bool, default True)
- `sort`: Sort result (bool, default True)

```python
import pandas as pd
import numpy as np

# Sample data
df = pd.DataFrame({
    'date': ['2023-01', '2023-01', '2023-02', '2023-02'],
    'city': ['NY', 'LA', 'NY', 'LA'],
    'product': ['A', 'B', 'A', 'B'],
    'sales': [100, 150, 120, 180],
    'quantity': [10, 15, 12, 18]
})

# Basic pivot - single value, single index, single column
df.pivot_table(values='sales', index='city', columns='product')
# Result: rows=cities, columns=products, values=average sales

# Specify aggregation function explicitly
df.pivot_table(values='sales', index='city', columns='product', aggfunc='sum')
df.pivot_table(values='sales', index='city', columns='product', aggfunc='mean')
df.pivot_table(values='sales', index='city', columns='product', aggfunc='count')

# Multiple aggregation functions
df.pivot_table(values='sales', 
               index='city', 
               columns='product',
               aggfunc=['sum', 'mean', 'count', 'std'])
# Creates MultiIndex columns: (sum, A), (sum, B), (mean, A), etc.

# Different functions for different values
df.pivot_table(index='city',
               columns='product',
               aggfunc={
                   'sales': 'sum',
                   'quantity': ['sum', 'mean']
               })

# Multiple values with single function
df.pivot_table(values=['sales', 'quantity'], 
               index='city',
               columns='product',
               aggfunc='sum')

# Multiple index levels (hierarchical rows)
df.pivot_table(values='sales',
               index=['city', 'date'],
               columns='product',
               aggfunc='sum')

# Multiple column levels (hierarchical columns)
df.pivot_table(values='sales',
               index='city',
               columns=['date', 'product'],
               aggfunc='sum')

# With margins (add totals)
df.pivot_table(values='sales', 
               index='city',
               columns='product',
               aggfunc='sum',
               margins=True,
               margins_name='Total')
# Adds 'Total' row and columns showing sums

# Fill missing values
df.pivot_table(values='sales',
               index='city',
               columns='product',
               aggfunc='sum',
               fill_value=0)
# Replace NaN with 0

# Custom aggregation function
def range_func(x):
    return x.max() - x.min()

df.pivot_table(values='sales',
               index='city',
               aggfunc=range_func)

# Using numpy functions
df.pivot_table(values='sales',
               index='city',
               columns='product',
               aggfunc=[np.sum, np.mean, np.median])

# Count non-null values
df.pivot_table(values='sales',
               index='city',
               columns='product',
               aggfunc='count')

# Count including nulls (use 'size')
df.pivot_table(index='city',
               columns='product',
               aggfunc='size')

# No columns parameter - single dimensional summary
df.pivot_table(values='sales',
               index='city',
               aggfunc=['sum', 'mean', 'count'])

# Percentage calculations
pt = df.pivot_table(values='sales', index='city', columns='product', aggfunc='sum')
pct = pt.div(pt.sum(axis=0), axis=1) * 100  # Percentage of column total

# Complex example: Sales dashboard
sales_summary = df.pivot_table(
    values=['sales', 'quantity'],
    index=['city', 'date'],
    columns='product',
    aggfunc={
        'sales': ['sum', 'mean'],
        'quantity': 'sum'
    },
    fill_value=0,
    margins=True,
    margins_name='Grand Total'
)

# Pivot table with calculated column
df['revenue_per_item'] = df['sales'] / df['quantity']
df.pivot_table(values='revenue_per_item', 
               index='city',
               columns='product',
               aggfunc='mean')

# Flatten MultiIndex columns after pivot
pt = df.pivot_table(values='sales', 
                    index='city',
                    columns=['product', 'date'],
                    aggfunc='sum')
pt.columns = ['_'.join(map(str, col)) for col in pt.columns.values]
pt = pt.reset_index()

# Time-based pivot (dates as columns)
df['date'] = pd.to_datetime(df['date'])
df.pivot_table(values='sales',
               index='product',
               columns=df['date'].dt.month,
               aggfunc='sum')

# Dropna parameter
df.pivot_table(values='sales',
               index='city',
               columns='product',
               dropna=False)  # Include NaN in grouping keys

# Advanced: Custom aggregation with multiple inputs
def weighted_avg(values, weights):
    return (values * weights).sum() / weights.sum()

df.pivot_table(index='city',
               aggfunc=lambda x: weighted_avg(x['sales'], x['quantity']))
```

---

## 7. Sorting

### df.sort_values()
Sort by column values - essential for data organization.

**Key Parameters:**
- `by`: Column name(s) to sort by (str or list)
- `axis`: 0 for index (sort rows), 1 for columns (sort by row values)
- `ascending`: True (default) or False, can be list for multiple columns
- `inplace`: Modify DataFrame in place (bool)
- `kind`: Sorting algorithm - 'quicksort', 'mergesort', 'heapsort', 'stable'
- `na_position`: 'last' (default) or 'first' - where to put NaNs
- `ignore_index`: Reset index to default integer (bool)
- `key`: Function to apply before sorting (like key in Python's sorted())

```python
import pandas as pd

# Sort by single column (ascending, default)
df.sort_values('age')
df.sort_values('age', ascending=True)

# Sort descending
df.sort_values('age', ascending=False)
df.sort_values('salary', ascending=False)

# Sort by multiple columns (all same direction)
df.sort_values(['city', 'age'])
df.sort_values(['department', 'salary', 'name'])

# Sort by multiple columns (different directions)
df.sort_values(['city', 'age'], ascending=[True, False])
# Sort by city ascending, then age descending within each city

df.sort_values(['department', 'salary', 'hire_date'], 
               ascending=[True, False, True])
# Complex multi-level sort

# Inplace sorting (modifies original DataFrame)
df.sort_values('age', inplace=True)

# Reset index after sorting
df.sort_values('age', ignore_index=True)
# Creates new 0,1,2... index instead of keeping original

df_sorted = df.sort_values('age').reset_index(drop=True)
# Alternative: sort and reset separately

# Handle NaN values
df.sort_values('age', na_position='last')  # NaN at bottom (default)
df.sort_values('age', na_position='first')  # NaN at top

# Sort by absolute value
df.sort_values('temperature', key=lambda x: x.abs())
# Useful for data that can be positive or negative

# Case-insensitive string sorting
df.sort_values('name', key=lambda x: x.str.lower())

# Sort by string length
df.sort_values('description', key=lambda x: x.str.len())

# Custom sorting function
df.sort_values('category', key=lambda x: x.map({
    'high': 3,
    'medium': 2, 
    'low': 1
}))

# Sort by month names (custom order)
month_order = {'January': 1, 'February': 2, 'March': 3, 'April': 4,
               'May': 5, 'June': 6, 'July': 7, 'August': 8,
               'September': 9, 'October': 10, 'November': 11, 'December': 12}
df.sort_values('month', key=lambda x: x.map(month_order))

# Sort by datetime components
df['date'] = pd.to_datetime(df['date'])
df.sort_values('date')  # Chronological order

# Sort by extracted date parts
df.sort_values('date', key=lambda x: x.dt.month)  # By month only
df.sort_values('date', key=lambda x: x.dt.dayofweek)  # By day of week

# Sort by calculated values (without creating new column)
df.sort_values('price', key=lambda x: x * df['quantity'])
# Sort by total value (price * quantity)

# Sort and get top/bottom N
df.sort_values('salary', ascending=False).head(10)  # Top 10 salaries
df.sort_values('age').head(5)  # 5 youngest
df.sort_values('score', ascending=False).tail(10)  # Bottom 10 scores

# Sort and filter
df.sort_values('date').tail(100)  # Most recent 100 records

# Sort after groupby
df.sort_values(['department', 'salary'], ascending=[True, False])
# Within each department, sort by salary descending

# Sort by column not in final output
df_sorted = df.sort_values('timestamp')[['name', 'value']]
# Sort by timestamp but don't include it in result

# Stable sort (maintains relative order for equal values)
df.sort_values('category', kind='stable')
# Important when you've already done a sort and want to maintain sub-order

# Sort by multiple criteria with different methods
df = df.sort_values('date')  # First by date
df = df.sort_values('priority', ascending=False, kind='stable')  # Then by priority
# Maintains date order within same priority

# Sort by index (see sort_index), but using sort_values
# Not typical, but possible
df.sort_values(by=df.index.name, axis=0)

# Sort columns (axis=1)
df.sort_values(by=0, axis=1)  # Sort columns based on first row values
df.sort_values(by='row_name', axis=1)  # Sort columns by specific row

# Rank-based sorting
df['rank'] = df['score'].rank(method='dense')
df.sort_values('rank')

# Sort with chaining
result = (df
    .query('age > 18')
    .sort_values(['city', 'salary'], ascending=[True, False])
    .head(100)
)

# Sort for display but don't save
print(df.sort_values('age'))  # Just for viewing
# Original df unchanged

# Complex example: Multi-level sort with custom order
priority_map = {'critical': 1, 'high': 2, 'medium': 3, 'low': 4}
status_map = {'open': 1, 'in_progress': 2, 'closed': 3}

df_sorted = df.sort_values(
    ['priority', 'status', 'created_date'],
    ascending=[True, True, False],
    key=lambda x: x.map(priority_map) if x.name == 'priority'
                 else (x.map(status_map) if x.name == 'status'
                 else x)
)

# Sort by aggregated values
# Sort categories by their average value
category_means = df.groupby('category')['value'].mean()
df['category_mean'] = df['category'].map(category_means)
df = df.sort_values('category_mean', ascending=False).drop('category_mean', axis=1)

# Sort to identify duplicates
df.sort_values(['email', 'created_date'])  # Duplicates grouped, oldest first
```

### df.sort_index()
Sort by index labels.

```python
# Sort by index
df.sort_index()

# Descending
df.sort_index(ascending=False)

# Sort columns
df.sort_index(axis=1)

# Inplace
df.sort_index(inplace=True)
```

---

## 8. Handling Missing Data

### Detecting Missing Data

```python
# Check for missing values
df.isna()       # or df.isnull() (alias)
df.notna()      # or df.notnull()

# Count missing values per column
df.isna().sum()

# Any missing values in DataFrame?
df.isna().any().any()

# Rows with any missing values
df[df.isna().any(axis=1)]

# Rows with all missing values
df[df.isna().all(axis=1)]
```

### df.fillna()
Fill missing values - critical for data cleaning.

**Key Parameters:**
- `value`: Scalar, dict, Series, or DataFrame to fill with
- `method`: 'ffill'/'pad' (forward fill) or 'bfill'/'backfill' (backward fill)
- `axis`: 0 for index (down), 1 for columns (across)
- `inplace`: Modify DataFrame in place (bool)
- `limit`: Maximum number of consecutive NaNs to fill
- `downcast`: Dict of item->dtype to downcast if possible

```python
import pandas as pd
import numpy as np

# Fill with scalar value (all NaN become 0)
df.fillna(0)
df.fillna('Unknown')
df.fillna(-999)  # Sentinel value

# Different values for different columns
df.fillna({
    'age': 0,
    'name': 'Unknown',
    'salary': df['salary'].median(),
    'city': 'Not Specified'
})

# Fill with column statistics
df['age'].fillna(df['age'].mean())  # Mean
df['age'].fillna(df['age'].median())  # Median
df['salary'].fillna(df['salary'].mode()[0])  # Mode (most common)
df['price'].fillna(df['price'].quantile(0.25))  # 25th percentile

# Fill all numeric columns with their mean
numeric_cols = df.select_dtypes(include=[np.number]).columns
df[numeric_cols] = df[numeric_cols].fillna(df[numeric_cols].mean())

# Fill categorical columns with mode
categorical_cols = df.select_dtypes(include=['object']).columns
for col in categorical_cols:
    df[col].fillna(df[col].mode()[0], inplace=True)

# Forward fill - use previous valid value
df.fillna(method='ffill')  # or method='pad'
df['temperature'].fillna(method='ffill')  # Common for time series

# Backward fill - use next valid value
df.fillna(method='bfill')  # or method='backfill'
df['status'].fillna(method='bfill')

# Forward fill with limit
df.fillna(method='ffill', limit=1)  # Fill only 1 consecutive NaN
df.fillna(method='ffill', limit=3)  # Fill up to 3 consecutive NaNs

# Fill along different axis
df.fillna(method='ffill', axis=1)  # Fill across columns
df.fillna(method='bfill', axis=0)  # Fill down rows (default)

# Fill with value from another column
df['email'].fillna(df['alternative_email'])
df['phone'].fillna(df['mobile'].where(df['mobile'].notna(), df['home_phone']))

# Fill with group statistics (common in groupby)
df['salary'] = df.groupby('department')['salary'].transform(
    lambda x: x.fillna(x.mean())
)

df['age'] = df.groupby('city')['age'].transform(
    lambda x: x.fillna(x.median())
)

# Fill with interpolation (see interpolate section for more details)
df['value'].fillna(method='ffill').fillna(method='bfill')  # Fill from both sides

# Inplace modification (changes original DataFrame)
df.fillna(0, inplace=True)
df['age'].fillna(df['age'].mean(), inplace=True)

# Conditional filling
df['salary'] = df['salary'].fillna(
    df['years_experience'] * 50000  # Estimate based on experience
)

# Fill with a Series (matched by index)
fill_values = pd.Series({'age': 25, 'salary': 50000})
df.fillna(fill_values)

# Fill with forward fill, then backward fill for remaining NaNs
df.fillna(method='ffill').fillna(method='bfill')

# Fill with custom function
df['score'] = df.groupby('category')['score'].transform(
    lambda x: x.fillna(x.rolling(window=3, min_periods=1).mean())
)

# Fill with constant for specific condition
df.loc[df['type'] == 'A', 'value'] = df.loc[df['type'] == 'A', 'value'].fillna(100)

# Complex example: Multiple strategies
for col in df.columns:
    if df[col].dtype in [np.float64, np.int64]:
        # Numeric: fill with median
        df[col].fillna(df[col].median(), inplace=True)
    elif df[col].dtype == 'object':
        # Categorical: fill with mode or 'Unknown'
        mode_val = df[col].mode()
        if len(mode_val) > 0:
            df[col].fillna(mode_val[0], inplace=True)
        else:
            df[col].fillna('Unknown', inplace=True)

# Verify no missing values remain
print(df.isna().sum())  # Should show 0 for all columns
```

### df.dropna()
Remove missing values.

```python
# Drop rows with any missing values
df.dropna()

# Drop rows where all values are missing
df.dropna(how='all')

# Drop rows with missing values in specific columns
df.dropna(subset=['age', 'name'])

# Drop columns with missing values
df.dropna(axis=1)

# Keep rows with at least n non-null values
df.dropna(thresh=2)

# Inplace
df.dropna(inplace=True)
```

### df.interpolate()
Fill missing values using interpolation.

```python
# Linear interpolation (default)
df.interpolate()

# Polynomial interpolation
df.interpolate(method='polynomial', order=2)

# Forward/backward direction
df.interpolate(limit_direction='forward')
df.interpolate(limit_direction='backward')
df.interpolate(limit_direction='both')

# Limit number of consecutive NaNs to fill
df.interpolate(limit=2)

# Time-based interpolation
df['date'] = pd.to_datetime(df['date'])
df.set_index('date').interpolate(method='time')
```

---

## 9. Merging and Joining

### pd.concat()
Concatenate DataFrames along axis - for stacking or side-by-side joining.

**Key Parameters:**
- `objs`: List or dict of DataFrames to concatenate
- `axis`: 0 for vertical (stack rows), 1 for horizontal (add columns)
- `join`: 'outer' (union, default) or 'inner' (intersection)
- `ignore_index`: Create new integer index instead of preserving original (bool)
- `keys`: Create hierarchical index using provided keys (list)
- `names`: Names for MultiIndex levels (list)
- `verify_integrity`: Check for duplicate indices (bool)
- `sort`: Sort non-concatenation axis (bool)
- `copy`: Copy data (bool, default True)

```python
import pandas as pd

# Vertical concatenation (stack rows, axis=0)
df_combined = pd.concat([df1, df2])  # Default: axis=0
df_combined = pd.concat([df1, df2], axis=0)

# Reset index after concatenation (continuous numbering)
df_combined = pd.concat([df1, df2], ignore_index=True)
# Without ignore_index: indices might repeat [0,1,2,0,1,2]
# With ignore_index: [0,1,2,3,4,5]

# Horizontal concatenation (add columns side-by-side, axis=1)
df_combined = pd.concat([df1, df2], axis=1)
# Aligns by index, creates NaN where indices don't match

# Inner join - only keep common indices/columns
df_combined = pd.concat([df1, df2], join='inner')
# Vertical (axis=0): keeps only common columns
# Horizontal (axis=1): keeps only common row indices

# Outer join - keep all indices/columns (default)
df_combined = pd.concat([df1, df2], join='outer')
# Creates NaN for missing values

# Add keys to identify source DataFrame
df_combined = pd.concat([df1, df2], keys=['first', 'second'])
# Creates MultiIndex: ('first', 0), ('first', 1), ('second', 0)...

df_combined = pd.concat([df1, df2, df3], keys=['jan', 'feb', 'mar'])
# Useful for time series or categorical grouping

# Name the hierarchical indices
df_combined = pd.concat([df1, df2], keys=['Q1', 'Q2'], names=['Quarter', 'ID'])

# Concatenate dictionary of DataFrames
dfs = {'North': df1, 'South': df2, 'East': df3}
df_combined = pd.concat(dfs)
# Automatically uses dict keys as the MultiIndex

# Concatenate multiple DataFrames from a list
all_dfs = [df1, df2, df3, df4, df5]
df_combined = pd.concat(all_dfs, ignore_index=True)

# Verify no duplicate indices (raises error if found)
df_combined = pd.concat([df1, df2], verify_integrity=True)

# Sort the non-concatenation axis
df_combined = pd.concat([df1, df2], sort=True)  # Sort columns alphabetically

# Specific columns only
df_combined = pd.concat([df1[['A', 'B']], df2[['A', 'B']]])

# Concatenate with different columns
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'B': [5, 6], 'C': [7, 8]})
df_combined = pd.concat([df1, df2])
# Result has columns A, B, C with NaN where data missing

df_combined = pd.concat([df1, df2], join='inner')
# Result has only column B (common column)

# Append multiple times (building a result iteratively)
result = pd.DataFrame()
for i, chunk in enumerate(data_chunks):
    result = pd.concat([result, process(chunk)], ignore_index=True)

# Better: Collect all chunks then concatenate once (more efficient)
chunks = []
for chunk in data_chunks:
    chunks.append(process(chunk))
result = pd.concat(chunks, ignore_index=True)

# Concatenate time series data
dfs = {
    '2023-01': df_jan,
    '2023-02': df_feb,
    '2023-03': df_mar
}
df_q1 = pd.concat(dfs, names=['month', 'record_id'])

# Concatenate with filtering
df_combined = pd.concat([df1[df1['status'] == 'active'], 
                        df2[df2['status'] == 'active']])

# Horizontal concat with suffixes (better to use merge)
df_combined = pd.concat([df1, df2], axis=1)
# If columns overlap, they both appear (might want suffixes)

# Mix DataFrames and Series
df = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
series = pd.Series([5, 6], name='C')
df_combined = pd.concat([df, series], axis=1)

# Concatenate with reindexing
df_combined = pd.concat([df1, df2]).reset_index(drop=True)

# Example: Combining yearly data
years = [2020, 2021, 2022, 2023]
dfs_by_year = [pd.read_csv(f'data_{year}.csv') for year in years]
df_all = pd.concat(dfs_by_year, keys=years, names=['year', 'row_id'])

# Example: Concatenate groups
grouped = df.groupby('category')
results = []
for name, group in grouped:
    processed = process_group(group)
    results.append(processed)
df_processed = pd.concat(results, ignore_index=True)
```

### pd.merge()
SQL-style joins - essential for combining DataFrames.

**Key Parameters:**
- `left`, `right`: DataFrames to merge
- `how`: Type of join - 'inner', 'left', 'right', 'outer', 'cross'
- `on`: Column(s) to join on (must exist in both DataFrames)
- `left_on`, `right_on`: Columns to join on if names differ
- `left_index`, `right_index`: Use index instead of columns (bool)
- `suffixes`: Tuple of suffixes for overlapping columns (default ('_x', '_y'))
- `indicator`: Add column showing merge type for each row (bool or str)
- `validate`: Check for merge type - 'one_to_one', 'one_to_many', 'many_to_one', 'many_to_many'
- `sort`: Sort result by join keys (bool)

**Join Types:**
- `inner`: Keep only matching keys from both DataFrames (intersection)
- `left`: Keep all keys from left, matching from right (left join)
- `right`: Keep all keys from right, matching from left (right join)
- `outer`: Keep all keys from both (union)
- `cross`: Cartesian product (all combinations)

```python
import pandas as pd

# Sample DataFrames
df1 = pd.DataFrame({'key': ['A', 'B', 'C'], 'value1': [1, 2, 3]})
df2 = pd.DataFrame({'key': ['A', 'B', 'D'], 'value2': [4, 5, 6]})

# Inner join (default) - only matching keys
df_merged = pd.merge(df1, df2, on='key')
# Result: key=['A','B'], value1=[1,2], value2=[4,5]

# Left join - all rows from left, matching from right
df_merged = pd.merge(df1, df2, on='key', how='left')
# Result: key=['A','B','C'], value2 has NaN for 'C'

# Right join - all rows from right, matching from left
df_merged = pd.merge(df1, df2, on='key', how='right')
# Result: key=['A','B','D'], value1 has NaN for 'D'

# Outer join - all records from both
df_merged = pd.merge(df1, df2, on='key', how='outer')
# Result: key=['A','B','C','D'] with NaNs where no match

# Cross join - Cartesian product (all combinations)
df_merged = pd.merge(df1, df2, how='cross')
# Result: Every row in df1 matched with every row in df2

# Multiple join keys
df_merged = pd.merge(df1, df2, on=['key1', 'key2'])
df_merged = pd.merge(df1, df2, on=['date', 'city', 'product'])

# Different column names in each DataFrame
df_merged = pd.merge(df1, df2, left_on='id', right_on='user_id')
df_merged = pd.merge(df1, df2, left_on='customer_id', right_on='cust_id')

# Multiple columns with different names
df_merged = pd.merge(df1, df2, 
                     left_on=['date', 'store_id'], 
                     right_on=['transaction_date', 'store'])

# Merge on index
df_merged = pd.merge(df1, df2, left_index=True, right_index=True)

# Merge column with index
df_merged = pd.merge(df1, df2, left_on='id', right_index=True)
df_merged = pd.merge(df1, df2, left_index=True, right_on='user_id')

# Custom suffixes for overlapping columns
df_merged = pd.merge(df1, df2, on='key', suffixes=('_left', '_right'))
df_merged = pd.merge(df1, df2, on='key', suffixes=('_old', '_new'))
df_merged = pd.merge(df1, df2, on='key', suffixes=('', '_from_df2'))

# Indicator column - shows source of each row
df_merged = pd.merge(df1, df2, on='key', how='outer', indicator=True)
# Creates '_merge' column: 'left_only', 'right_only', or 'both'

df_merged = pd.merge(df1, df2, on='key', how='outer', indicator='source')
# Custom indicator column name

# Filter based on merge type
only_in_left = df_merged[df_merged['_merge'] == 'left_only']
only_in_right = df_merged[df_merged['_merge'] == 'right_only']
in_both = df_merged[df_merged['_merge'] == 'both']

# Validate merge relationships
df_merged = pd.merge(df1, df2, on='key', validate='one_to_one')
df_merged = pd.merge(df1, df2, on='key', validate='one_to_many')
df_merged = pd.merge(df1, df2, on='key', validate='many_to_one')
# Raises error if validation fails - useful for data quality checks

# Sort by join keys
df_merged = pd.merge(df1, df2, on='key', sort=True)

# Complex example - multiple tables
df_final = pd.merge(df1, df2, on='id', how='left')
df_final = pd.merge(df_final, df3, on='category', how='left')
df_final = pd.merge(df_final, df4, left_on='user_id', right_on='id', how='inner')

# Merge and handle duplicate column names
df_merged = pd.merge(df1, df2, on='key', suffixes=('_sales', '_inventory'))

# Merge with filtering
# Only merge rows where condition is met
df1_filtered = df1[df1['status'] == 'active']
df_merged = pd.merge(df1_filtered, df2, on='key')

# Multiple merges with method chaining
result = (df1
    .merge(df2, on='id', how='left')
    .merge(df3, on='category', how='left')
    .merge(df4, left_on='user_id', right_on='id', how='inner')
)

# Merge and immediately filter
df_merged = pd.merge(df1, df2, on='key', how='outer', indicator=True)
df_merged = df_merged[df_merged['_merge'] == 'both'].drop('_merge', axis=1)
```

### df.join()
Join DataFrames using index.

```python
# Join on index (left join by default)
df_joined = df1.join(df2)

# Specify join type
df_joined = df1.join(df2, how='inner')
df_joined = df1.join(df2, how='outer')

# Join on column
df_joined = df1.join(df2, on='key')

# Join multiple DataFrames
df_joined = df1.join([df2, df3, df4])

# Suffixes for overlapping columns
df_joined = df1.join(df2, lsuffix='_left', rsuffix='_right')
```

---

## 10. Reshaping Data

### df.melt()
Transform from wide to long format.

```python
# Basic melt
df_long = df.melt()

# Specify id variables (columns to keep)
df_long = df.melt(id_vars=['name'])

# Specify value variables (columns to melt)
df_long = df.melt(id_vars=['name'], value_vars=['math', 'english'])

# Name the variable and value columns
df_long = df.melt(id_vars=['name'],
                  var_name='subject',
                  value_name='score')

# Example:
# Before: name | math | english
#         Alice| 90   | 85
# After:  name  | subject | score
#         Alice | math    | 90
#         Alice | english | 85
```

### df.pivot()
Transform from long to wide format.

```python
# Basic pivot
df_wide = df.pivot(index='name', columns='subject', values='score')

# Multiple value columns
df_wide = df.pivot(index='name', 
                   columns='subject',
                   values=['score', 'grade'])

# Example:
# Before: name  | subject | score
#         Alice | math    | 90
#         Alice | english | 85
# After:  subject| math | english
#         name   |      |
#         Alice  | 90   | 85
```

### df.stack() and df.unstack()
Pivot level of column/index labels.

```python
# Stack: pivot columns to rows (wide to long)
df_stacked = df.stack()

# Unstack: pivot rows to columns (long to wide)
df_unstacked = df.unstack()

# Specify level to stack/unstack
df.stack(level=0)
df.unstack(level=-1)

# Handle missing values
df.unstack(fill_value=0)
```

---

## 11. Applying Functions

### df.apply()
Apply function along axis - very versatile for custom transformations.

**Key Parameters:**
- `func`: Function to apply to each column/row
- `axis`: 0 or 'index' (apply to columns), 1 or 'columns' (apply to rows)
- `raw`: Pass ndarray instead of Series (bool, faster but less convenient)
- `result_type`: 'expand' (list-like to columns), 'reduce' (return Series), 'broadcast' (return same shape)
- `args`: Positional arguments to pass to func (tuple)
- `**kwargs`: Keyword arguments to pass to func

```python
import pandas as pd
import numpy as np

# Apply to each column (axis=0, default)
df.apply(np.sum)  # Sum of each column
df.apply(np.mean)  # Mean of each column
df.apply(lambda x: x.max() - x.min())  # Range of each column
df.apply(len)  # Length of each column (number of rows)

# Apply to each row (axis=1)
df.apply(lambda row: row['age'] * 2, axis=1)  # Returns Series
df.apply(lambda row: row.sum(), axis=1)  # Sum across columns
df.apply(lambda row: row['price'] * row['quantity'], axis=1)  # Calculate total

# Create new column from multiple columns
df['full_name'] = df.apply(lambda row: f"{row['first_name']} {row['last_name']}", axis=1)
df['bmi'] = df.apply(lambda row: row['weight'] / (row['height'] ** 2), axis=1)

# Return multiple values (creates new DataFrame)
df.apply(lambda row: pd.Series({
    'age_doubled': row['age'] * 2,
    'name_upper': row['name'].upper(),
    'age_category': 'adult' if row['age'] >= 18 else 'minor'
}), axis=1)

# Use result_type parameter
df.apply(lambda x: [x.min(), x.max()], result_type='expand')  # Creates 2 columns
df.apply(lambda x: [x.min(), x.max()], result_type='broadcast')  # Same shape as input

# Apply custom function to Series (single column)
def categorize_age(age):
    if age < 18:
        return 'minor'
    elif age < 65:
        return 'adult'
    else:
        return 'senior'

df['age_category'] = df['age'].apply(categorize_age)

# Apply with additional arguments
def add_bonus(salary, bonus, multiplier=1.1):
    return (salary + bonus) * multiplier

df['new_salary'] = df['salary'].apply(add_bonus, args=(1000,), multiplier=1.15)
df['new_salary'] = df['salary'].apply(add_bonus, bonus=1000, multiplier=1.15)

# Apply with external data (using variables)
tax_rate = 0.25
df['after_tax'] = df['salary'].apply(lambda x: x * (1 - tax_rate))

# Apply string methods
df['name_clean'] = df['name'].apply(str.strip)  # Remove whitespace
df['email_lower'] = df['email'].apply(str.lower)
df['name_title'] = df['name'].apply(lambda x: x.title() if isinstance(x, str) else x)

# Apply with conditionals
df['discount'] = df['quantity'].apply(
    lambda x: 0.2 if x > 100 else (0.1 if x > 50 else 0)
)

# Apply with type checking
def safe_divide(x):
    try:
        return 100 / x
    except (ZeroDivisionError, TypeError):
        return np.nan

df['result'] = df['value'].apply(safe_divide)

# Apply with dictionary mapping (map is more efficient for this)
mapping = {'A': 'Excellent', 'B': 'Good', 'C': 'Average'}
df['grade_name'] = df['grade'].apply(lambda x: mapping.get(x, 'Unknown'))

# Apply to datetime operations
df['date'] = pd.to_datetime(df['date'])
df['day_of_week'] = df['date'].apply(lambda x: x.strftime('%A'))
df['is_weekend'] = df['date'].apply(lambda x: x.dayofweek >= 5)

# Apply with regex
import re
df['phone_clean'] = df['phone'].apply(lambda x: re.sub(r'\D', '', str(x)))

# Apply with JSON parsing
import json
df['parsed_data'] = df['json_column'].apply(lambda x: json.loads(x) if pd.notna(x) else {})
df['nested_value'] = df['parsed_data'].apply(lambda x: x.get('key', None))

# Performance tip: raw=True for numeric operations (faster)
df.apply(lambda x: x.sum(), raw=True)  # x is ndarray, not Series

# Apply with multiple columns explicitly
def calculate_score(row):
    score = row['math'] * 0.4 + row['english'] * 0.3 + row['science'] * 0.3
    if score > 90:
        return 'A'
    elif score > 80:
        return 'B'
    else:
        return 'C'

df['final_grade'] = df.apply(calculate_score, axis=1)

# Apply with progress indication (useful for large DataFrames)
from tqdm import tqdm
tqdm.pandas()
df['result'] = df['column'].progress_apply(some_function)

# Complex example: Multiple operations
def process_row(row):
    # Perform multiple calculations
    total = row['qty'] * row['price']
    discount = total * 0.1 if row['member'] else 0
    tax = (total - discount) * 0.08
    final = total - discount + tax
    
    return pd.Series({
        'total': total,
        'discount': discount,
        'tax': tax,
        'final_amount': final
    })

df[['total', 'discount', 'tax', 'final_amount']] = df.apply(process_row, axis=1)

# Vectorized alternative (usually faster than apply)
# Instead of: df['result'] = df['col'].apply(lambda x: x * 2)
# Use: df['result'] = df['col'] * 2
# Apply is best when you NEED row-wise custom logic
```

### df.applymap()
Apply function element-wise to DataFrame.

```python
# Apply to every element
df.applymap(str.lower)
df.applymap(lambda x: x * 2)

# Format numbers
df.applymap(lambda x: f'{x:.2f}' if isinstance(x, float) else x)

# Note: In pandas 2.1+, applymap is renamed to map
# df.map(lambda x: x * 2)
```

### Series.map()
Map values using dictionary or function.

```python
# Using dictionary
mapping = {'NY': 'New York', 'LA': 'Los Angeles', 'SF': 'San Francisco'}
df['city_full'] = df['city'].map(mapping)

# Using function
df['age'].map(lambda x: x * 2)

# Map to another Series
df['age'].map(df['age'] + 10)
```

---

## 12. Working with String Data

### String Methods (str accessor)
Pandas provides vectorized string operations via the str accessor.

**Common Methods:**
- Case: `lower()`, `upper()`, `title()`, `capitalize()`, `swapcase()`
- Cleaning: `strip()`, `lstrip()`, `rstrip()`, `replace()`
- Searching: `contains()`, `startswith()`, `endswith()`, `find()`, `match()`
- Splitting: `split()`, `rsplit()`, `partition()`, `extract()`
- Joining: `join()`, `cat()`
- Formatting: `pad()`, `center()`, `ljust()`, `rjust()`, `zfill()`

```python
import pandas as pd

# Case conversion
df['name_lower'] = df['name'].str.lower()
df['name_upper'] = df['name'].str.upper()
df['name_title'] = df['name'].str.title()  # First Letter Of Each Word
df['name_cap'] = df['name'].str.capitalize()  # First letter only

# Remove whitespace
df['clean_text'] = df['text'].str.strip()  # Both ends
df['clean_text'] = df['text'].str.lstrip()  # Left side
df['clean_text'] = df['text'].str.rstrip()  # Right side

# String replacement
df['phone_clean'] = df['phone'].str.replace('-', '')  # Remove dashes
df['text_clean'] = df['text'].str.replace('old', 'new')
df['price_clean'] = df['price'].str.replace('$', '').str.replace(',', '')

# Regex replacement
df['phone'] = df['phone'].str.replace(r'\D', '', regex=True)  # Keep only digits
df['text'] = df['text'].str.replace(r'\s+', ' ', regex=True)  # Multiple spaces to one

# Check if contains substring
df[df['email'].str.contains('@gmail.com')]
df[df['name'].str.contains('John', case=False)]  # Case insensitive
df[df['text'].str.contains('error|warning', case=False)]  # Multiple patterns

# Handle NaN in contains
df[df['email'].str.contains('@', na=False)]  # Treat NaN as False

# Starts with / Ends with
df[df['name'].str.startswith('A')]
df[df['file'].str.endswith('.csv')]
df[df['code'].str.startswith(('A', 'B', 'C'))]  # Multiple prefixes

# String length
df['name_length'] = df['name'].str.len()
df[df['description'].str.len() > 100]  # Long descriptions

# Split strings
df['first_name'] = df['full_name'].str.split(' ').str[0]
df['last_name'] = df['full_name'].str.split(' ').str[-1]

# Split into multiple columns
df[['first', 'last']] = df['name'].str.split(' ', expand=True)
df[['area', 'exchange', 'number']] = df['phone'].str.split('-', expand=True)

# Split with limit
df['first_two'] = df['text'].str.split(' ', n=1)  # Split at first space only

# Extract using regex
df['digits'] = df['text'].str.extract(r'(\d+)')  # First number
df[['code', 'value']] = df['text'].str.extract(r'([A-Z]+)-(\d+)')  # Multiple groups

# Find position
df['pos'] = df['text'].str.find('keyword')  # Returns -1 if not found
df['pos'] = df['email'].str.find('@')  # Position of @

# Get substring by position
df['first_3'] = df['name'].str[:3]  # First 3 characters
df['last_3'] = df['name'].str[-3:]  # Last 3 characters
df['middle'] = df['name'].str[2:5]  # Characters 2-4

# Concatenate strings
df['full_name'] = df['first_name'].str.cat(df['last_name'], sep=' ')
df['address'] = df['street'].str.cat([df['city'], df['state'], df['zip']], sep=', ')

# Join list of strings
df['tags_str'] = df['tags_list'].str.join(', ')  # If column contains lists

# Padding
df['padded'] = df['id'].str.pad(width=8, fillchar='0')  # Pad with zeros
df['centered'] = df['text'].str.center(50)  # Center in 50 chars
df['left'] = df['text'].str.ljust(30)  # Left justify
df['right'] = df['text'].str.rjust(30)  # Right justify
df['zeros'] = df['number'].str.zfill(5)  # Pad with leading zeros

# Count occurrences
df['comma_count'] = df['text'].str.count(',')
df['word_count'] = df['text'].str.count(r'\w+')  # Count words

# Repeat string
df['repeated'] = df['char'].str.repeat(3)  # 'A' -> 'AAA'

# Boolean checks
df['is_alpha'] = df['code'].str.isalpha()  # All letters
df['is_digit'] = df['code'].str.isdigit()  # All digits
df['is_alnum'] = df['code'].str.isalnum()  # Letters and numbers
df['is_upper'] = df['code'].str.isupper()  # All uppercase
df['is_lower'] = df['code'].str.islower()  # All lowercase
df['is_space'] = df['text'].str.isspace()  # All whitespace

# Normalize whitespace
df['normalized'] = df['text'].str.split().str.join(' ')  # Single spaces

# Remove special characters
df['clean'] = df['text'].str.replace(r'[^a-zA-Z0-9\s]', '', regex=True)

# Extract email domains
df['domain'] = df['email'].str.split('@').str[1]

# Format phone numbers
df['formatted_phone'] = df['phone'].str.replace(
    r'(\d{3})(\d{3})(\d{4})', r'(\1) \2-\3', regex=True
)

# URL parsing
df['protocol'] = df['url'].str.extract(r'^(https?://)')  
df['domain'] = df['url'].str.extract(r'https?://([^/]+)')

# Complex cleaning pipeline
df['clean_text'] = (df['text']
    .str.strip()                              # Remove leading/trailing space
    .str.lower()                              # Lowercase
    .str.replace(r'\s+', ' ', regex=True)    # Normalize whitespace
    .str.replace(r'[^a-z0-9\s]', '', regex=True)  # Remove special chars
)

# Handle missing values
df['clean'] = df['text'].fillna('').str.strip()  # Fill NaN before string ops
```

---

## 13. Working with Time Data

### pd.to_datetime()
Convert to datetime objects.

```python
# Convert column to datetime
df['date'] = pd.to_datetime(df['date'])

# Specify format
df['date'] = pd.to_datetime(df['date'], format='%Y-%m-%d')

# Handle errors
df['date'] = pd.to_datetime(df['date'], errors='coerce')  # NaT for invalid
df['date'] = pd.to_datetime(df['date'], errors='ignore')  # leave as is

# From multiple columns
df['date'] = pd.to_datetime(df[['year', 'month', 'day']])

# Unix timestamp
df['date'] = pd.to_datetime(df['timestamp'], unit='s')
```

### dt Accessor
Access datetime properties.

```python
# Set date column as datetime
df['date'] = pd.to_datetime(df['date'])

# Extract components
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['hour'] = df['date'].dt.hour
df['minute'] = df['date'].dt.minute
df['second'] = df['date'].dt.second

# Day of week
df['day_of_week'] = df['date'].dt.dayofweek  # 0=Monday, 6=Sunday
df['day_name'] = df['date'].dt.day_name()

# Other properties
df['quarter'] = df['date'].dt.quarter
df['is_month_start'] = df['date'].dt.is_month_start
df['is_month_end'] = df['date'].dt.is_month_end
df['is_quarter_start'] = df['date'].dt.is_quarter_start
df['days_in_month'] = df['date'].dt.days_in_month

# Format as string
df['date_str'] = df['date'].dt.strftime('%Y-%m-%d')
```

### df.resample()
Resample time-series data.

```python
# Set datetime index
df.set_index('date', inplace=True)

# Resample to different frequency
df.resample('D').mean()    # Daily
df.resample('W').sum()     # Weekly
df.resample('M').mean()    # Monthly
df.resample('Q').sum()     # Quarterly
df.resample('Y').mean()    # Yearly

# Upsample (increase frequency)
df.resample('H').ffill()   # Hourly, forward fill

# Custom aggregation
df.resample('M').agg({
    'sales': 'sum',
    'price': 'mean'
})

# Multiple functions
df.resample('M')['sales'].agg(['sum', 'mean', 'count'])
```

---

## 13. Exporting Data

### df.to_csv()
Write DataFrame to CSV file - most common export format.

**Key Parameters:**
- `path_or_buf`: File path or object (str or file-like)
- `sep`: Field delimiter (default ',')
- `na_rep`: String representation of missing values (default '')
- `columns`: Columns to write (list)
- `header`: Write column names (bool or list of str)
- `index`: Write row index (bool, default True)
- `index_label`: Column label for index (str or list)
- `mode`: 'w' for write (default), 'a' for append
- `encoding`: File encoding (e.g., 'utf-8', 'latin1', 'cp1252')
- `compression`: 'infer' (default), 'gzip', 'bz2', 'zip', 'xz', None
- `quoting`: Control quoting behavior (csv.QUOTE_MINIMAL, etc.)
- `quotechar`: Character for quoting (default '"')
- `line_terminator`: String for line endings (default '\n')
- `chunksize`: Write in batches (int)
- `date_format`: Format for datetime objects (str)
- `doublequote`: Control quoting of quote char (bool)
- `escapechar`: String escape character (str)

```python
import pandas as pd
import csv

# Basic export
df.to_csv('output.csv')

# Without index (most common for clean exports)
df.to_csv('output.csv', index=False)

# Custom separator (tab-separated)
df.to_csv('output.tsv', sep='\t', index=False)

# Pipe-separated
df.to_csv('output.txt', sep='|', index=False)

# Export specific columns only
df.to_csv('output.csv', columns=['name', 'age', 'city'], index=False)
df[['col1', 'col2', 'col3']].to_csv('output.csv', index=False)

# Handle missing values
df.to_csv('output.csv', na_rep='NULL', index=False)  # NULL for NaN
df.to_csv('output.csv', na_rep='NA', index=False)  # NA for NaN
df.to_csv('output.csv', na_rep='', index=False)  # Empty string (default)
df.to_csv('output.csv', na_rep='-999', index=False)  # Numeric sentinel

# Append to existing file
df.to_csv('output.csv', mode='a', header=False, index=False)
# header=False prevents writing column names again

# Write with specific encoding
df.to_csv('output.csv', encoding='utf-8', index=False)
df.to_csv('output.csv', encoding='latin1', index=False)
df.to_csv('output.csv', encoding='utf-8-sig', index=False)  # UTF-8 with BOM (Excel compatible)

# Compression
df.to_csv('output.csv.gz', compression='gzip', index=False)
df.to_csv('output.csv.bz2', compression='bz2', index=False)
df.to_csv('output.csv.zip', compression='zip', index=False)
df.to_csv('output.csv.xz', compression='xz', index=False)

# Auto-detect compression from filename
df.to_csv('output.csv.gz', index=False)  # Automatically uses gzip

# Custom column names in output
df.to_csv('output.csv', header=['Name', 'Age', 'Location'], index=False)

# No header row
df.to_csv('output.csv', header=False, index=False)

# Include index with custom label
df.to_csv('output.csv', index_label='ID')

# Multiple index levels with labels
df.to_csv('output.csv', index_label=['Level1', 'Level2'])

# Format datetime columns
df.to_csv('output.csv', date_format='%Y-%m-%d', index=False)
df.to_csv('output.csv', date_format='%Y-%m-%d %H:%M:%S', index=False)
df.to_csv('output.csv', date_format='%m/%d/%Y', index=False)  # US format

# Quoting behavior
df.to_csv('output.csv', quoting=csv.QUOTE_ALL, index=False)  # Quote all fields
df.to_csv('output.csv', quoting=csv.QUOTE_MINIMAL, index=False)  # Quote only if needed (default)
df.to_csv('output.csv', quoting=csv.QUOTE_NONNUMERIC, index=False)  # Quote non-numeric
df.to_csv('output.csv', quoting=csv.QUOTE_NONE, index=False)  # Never quote

# Custom quote character
df.to_csv('output.csv', quotechar="'", index=False)  # Single quote instead of double

# Custom line terminator
df.to_csv('output.csv', line_terminator='\r\n', index=False)  # Windows style
df.to_csv('output.csv', line_terminator='\n', index=False)  # Unix style (default)

# Escape character for special characters
df.to_csv('output.csv', escapechar='\\', quoting=csv.QUOTE_NONE, index=False)

# Write to string instead of file
csv_string = df.to_csv(index=False)
print(csv_string)

# Write to buffer
from io import StringIO
buffer = StringIO()
df.to_csv(buffer, index=False)
csv_content = buffer.getvalue()

# Float formatting
df.to_csv('output.csv', float_format='%.2f', index=False)  # 2 decimal places
df.to_csv('output.csv', float_format='%.4f', index=False)  # 4 decimal places

# Large files - write in chunks
for chunk in pd.read_csv('large_input.csv', chunksize=10000):
    processed = process(chunk)
    processed.to_csv('output.csv', mode='a', 
                    header=not os.path.exists('output.csv'), 
                    index=False)

# Write specific rows (filtered data)
df[df['age'] > 18].to_csv('adults.csv', index=False)
df[df['status'] == 'active'].to_csv('active_users.csv', index=False)

# Multiple files based on grouping
for city, group in df.groupby('city'):
    group.to_csv(f'data_{city}.csv', index=False)

# Write with timestamp in filename
from datetime import datetime
timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
df.to_csv(f'output_{timestamp}.csv', index=False)

# Decimal separator (European format)
df.to_csv('output.csv', decimal=',', sep=';', index=False)
# Common for European Excel

# Double quote handling
df.to_csv('output.csv', doublequote=True, index=False)
# True: " becomes "" (default)
# False: uses escapechar

# Complex example: Production-ready export
output_file = 'data_export.csv'
df.to_csv(
    output_file,
    index=False,
    encoding='utf-8-sig',  # Excel compatible
    na_rep='',
    date_format='%Y-%m-%d',
    float_format='%.2f',
    quoting=csv.QUOTE_MINIMAL,
    compression='gzip'
)
print(f"Exported {len(df)} rows to {output_file}")

# Export with validation
try:
    df.to_csv('output.csv', index=False)
    print(f"Successfully exported {len(df)} rows")
except Exception as e:
    print(f"Export failed: {e}")

# Export subset of data with custom processing
df_export = df[['name', 'email', 'age']].copy()
df_export['email'] = df_export['email'].str.lower()
df_export.to_csv('clean_export.csv', index=False)
```

### df.to_excel()
Write DataFrame to Excel file.

```python
# Basic export
df.to_excel('output.xlsx')

# Without index
df.to_excel('output.xlsx', index=False)

# Specify sheet name
df.to_excel('output.xlsx', sheet_name='Data')

# Multiple sheets
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sheet1')
    df2.to_excel(writer, sheet_name='Sheet2')

# Start at specific row/column
df.to_excel('output.xlsx', startrow=2, startcol=1)
```

### df.to_sql()
Write DataFrame to SQL database.

```python
import sqlite3

# Create connection
conn = sqlite3.connect('database.db')

# Write to database
df.to_sql('table_name', conn, if_exists='replace', index=False)

# Options for if_exists:
# - 'fail': raise error if table exists (default)
# - 'replace': drop and recreate table
# - 'append': add data to existing table

# Specify data types
df.to_sql('table_name', conn, dtype={'age': 'INTEGER', 'name': 'TEXT'})

# Close connection
conn.close()
```

### df.to_dict()
Convert DataFrame to dictionary.

```python
# Default: dict of lists
df.to_dict()

# Records: list of dicts
df.to_dict('records')
# [{'name': 'Alice', 'age': 25}, {'name': 'Bob', 'age': 30}]

# Index: dict of dicts
df.to_dict('index')
# {0: {'name': 'Alice', 'age': 25}, 1: {'name': 'Bob', 'age': 30}}

# List: dict of lists (default)
df.to_dict('list')

# Series: dict of Series
df.to_dict('series')

# Split: dict with index, columns, and data
df.to_dict('split')
```

### df.to_json()
Write DataFrame to JSON.

```python
# Default orientation
df.to_json('output.json')

# Records orientation (list of dicts)
df.to_json('output.json', orient='records')

# Pretty print
df.to_json('output.json', indent=4)

# Include index
df.to_json('output.json', orient='index')

# Return JSON string
json_str = df.to_json(orient='records')
```

### df.to_html()
Render DataFrame as HTML table.

```python
# Basic HTML
html = df.to_html()

# Without index
html = df.to_html(index=False)

# Write to file
df.to_html('output.html')

# Custom classes
html = df.to_html(classes=['table', 'table-striped'])

# Format floats
html = df.to_html(float_format=lambda x: f'{x:.2f}')
```

---

## 15. Other Useful Operations

### Handling Duplicates
Find and remove duplicate rows - essential for data quality.

```python
import pandas as pd

# Check for duplicates (returns boolean Series)
df.duplicated()  # True where row is duplicate

# Count duplicates
print(f"Number of duplicates: {df.duplicated().sum()}")

# Check specific columns only
df.duplicated(subset=['email'])  # Duplicate emails
df.duplicated(subset=['name', 'age'])  # Duplicate name AND age

# Keep first occurrence (default)
df.duplicated(keep='first')  # First is False, rest are True

# Keep last occurrence
df.duplicated(keep='last')  # Last is False, rest are True

# Mark all duplicates (including first)
df.duplicated(keep=False)  # All duplicates marked True

# View duplicate rows
duplicate_rows = df[df.duplicated()]
print(f"Found {len(duplicate_rows)} duplicate rows")

# View all occurrences of duplicates (including first)
all_duplicates = df[df.duplicated(subset=['email'], keep=False)]
all_duplicates.sort_values('email')  # See them grouped

# Drop duplicates (keep first)
df_clean = df.drop_duplicates()

# Drop duplicates in specific columns
df_clean = df.drop_duplicates(subset=['email'])  # Unique emails
df_clean = df.drop_duplicates(subset=['name', 'date'])  # Unique name-date combos

# Keep last occurrence instead
df_clean = df.drop_duplicates(keep='last')

# Keep neither (remove all duplicates)
df_clean = df.drop_duplicates(keep=False)  # Only keep unique rows

# Inplace modification
df.drop_duplicates(inplace=True)
df.drop_duplicates(subset=['email'], inplace=True)

# Drop duplicates and reset index
df_clean = df.drop_duplicates().reset_index(drop=True)

# Find which values are duplicated
email_duplicates = df[df['email'].duplicated(keep=False)]
print(email_duplicates.sort_values('email'))

# Count occurrences of each duplicate
duplicate_counts = df[df.duplicated(subset=['email'], keep=False)].groupby('email').size()
print("Emails appearing multiple times:")
print(duplicate_counts[duplicate_counts > 1].sort_values(ascending=False))

# Keep row with max/min value among duplicates
# Keep most recent record for each email
df_latest = df.sort_values('date').drop_duplicates(subset=['email'], keep='last')

# Keep highest value for each duplicate
df_best = df.sort_values('score').drop_duplicates(subset=['user_id'], keep='last')

# Complex deduplication: keep based on multiple criteria
df_sorted = df.sort_values(['email', 'updated_date', 'priority'], 
                           ascending=[True, False, False])
df_clean = df_sorted.drop_duplicates(subset=['email'], keep='first')
# Keeps most recent, highest priority record for each email

# Identify duplicate groups with IDs
df['dup_group'] = df.groupby('email').ngroup()
duplicates = df[df['email'].duplicated(keep=False)].sort_values('dup_group')

# Compare first and last occurrence
first_occur = df[~df.duplicated(subset=['email'], keep='first')]
last_occur = df[~df.duplicated(subset=['email'], keep='last')]

# Save duplicates before dropping
duplicates = df[df.duplicated()].copy()
duplicates.to_csv('duplicates.csv', index=False)
df_clean = df.drop_duplicates()

# Fuzzy duplicate detection (requires fuzzywuzzy/thefuzz)
# For names that might have slight variations
import numpy as np

def find_fuzzy_duplicates(series, threshold=90):
    from thefuzz import fuzz
    duplicates = []
    for i, val1 in enumerate(series):
        for j, val2 in enumerate(series[i+1:], i+1):
            if fuzz.ratio(str(val1).lower(), str(val2).lower()) > threshold:
                duplicates.append((i, j, val1, val2))
    return duplicates

# Check for duplicates across multiple DataFrames
common = pd.merge(df1[['email']], df2[['email']], on='email', how='inner')
print(f"Found {len(common)} emails in both DataFrames")
```

### df.memory_usage()
Return memory usage of each column.

```python
# Memory usage per column
df.memory_usage()

# Include index
df.memory_usage(index=True)

# Deep introspection (actual memory)
df.memory_usage(deep=True)

# Total memory
total_memory = df.memory_usage(deep=True).sum()
print(f"Total memory: {total_memory / 1024**2:.2f} MB")
```

### df.clip()
Trim values at input thresholds.

```python
# Clip all values
df.clip(lower=0, upper=100)

# Clip specific column
df['age'].clip(lower=18, upper=65)

# Clip only lower bound
df.clip(lower=0)

# Clip only upper bound
df.clip(upper=100)

# Inplace
df.clip(lower=0, upper=100, inplace=True)
```

### Correlation and Covariance

```python
# Correlation matrix
df.corr()

# Correlation with specific column
df.corr()['age']

# Correlation between two columns
df['age'].corr(df['salary'])

# Pearson (default), Kendall, or Spearman
df.corr(method='pearson')
df.corr(method='kendall')
df.corr(method='spearman')

# Covariance matrix
df.cov()

# Covariance between two columns
df['age'].cov(df['salary'])
```

---

## Section Summary

This reference guide covers the essential Pandas DataFrame operations organized into 15 key categories:

1. **Creating DataFrames** - Multiple ways to instantiate DataFrames from various sources (read_csv, read_excel, from_dict, etc.)
2. **Viewing and Inspecting** - Methods to examine structure and content (head, tail, info, describe, sample, value_counts)
3. **Selection and Indexing** - Accessing specific data using labels and positions (loc, iloc, at, iat)
4. **Filtering and Querying** - Subsetting data based on conditions (boolean indexing, query, isin, between)
5. **Modifying Data** - Adding, removing, and transforming columns and rows (drop, rename, set_index)
6. **Aggregation and Statistics** - Summarizing and computing metrics (groupby, agg, pivot_table)
7. **Sorting** - Ordering data by values or index (sort_values, sort_index, nlargest, nsmallest)
8. **Handling Missing Data** - Detecting, filling, and removing NaN values (isna, fillna, dropna, interpolate)
9. **Merging and Joining** - Combining multiple DataFrames (concat, merge, join)
10. **Reshaping Data** - Transforming between wide and long formats (melt, pivot, stack, unstack)
11. **Applying Functions** - Custom transformations across DataFrame (apply, applymap, map)
12. **Working with String Data** - String operations and text processing (str accessor methods)
13. **Working with Time Data** - Datetime operations and resampling (to_datetime, dt accessor, resample)
14. **Exporting Data** - Writing DataFrames to various formats (to_csv, to_excel, to_sql, to_json)
15. **Other Useful Operations** - Duplicates, memory management, clipping, and correlation

---

## Key Takeaways

- **Method Chaining**: Many operations return DataFrames, allowing chains like `df.dropna().groupby('col').mean()`
- **Inplace Parameter**: Most methods return a new DataFrame by default; use `inplace=True` to modify the original
- **Axis Parameter**: `axis=0` operates on rows (down), `axis=1` operates on columns (across)
- **Copy vs View**: Some operations return views of the original data; use `.copy()` to ensure independence
- **Performance**: Use vectorized operations instead of loops; consider `apply()` for complex row-wise operations
- **Missing Data**: Always check for and handle NaN values appropriately for your use case
- **Index Matters**: Understanding and properly managing the index is crucial for many operations, especially merging and joining

---

## Practice Recommendations

### Beginner Level
1. Create DataFrames from dictionaries and CSV files
2. Practice selection with `loc`, `iloc`, and boolean indexing
3. Perform basic filtering and sorting operations
4. Handle missing values with `fillna()` and `dropna()`
5. Export DataFrames to CSV and Excel

### Intermediate Level
1. Master `groupby()` with multiple aggregation functions
2. Merge and join multiple DataFrames with different join types
3. Reshape data using `pivot()`, `melt()`, and `pivot_table()`
4. Apply custom functions with `apply()` and `map()`
5. Work with datetime data and perform resampling

### Advanced Level
1. Optimize performance with appropriate data types and memory management
2. Chain complex operations efficiently
3. Handle large datasets with chunking and compression
4. Create custom aggregation functions for `groupby()` and `agg()`
5. Build complete ETL pipelines combining multiple operations

### Practice Dataset Ideas
- **Sales Data**: Track products, dates, quantities, and revenue
- **Student Records**: Names, grades, subjects, and demographics
- **Weather Data**: Timestamps, temperatures, and conditions
- **Financial Data**: Stock prices, volumes, and indicators
- **Social Media**: Posts, timestamps, likes, and user information

### Recommended Next Steps
1. Explore the [official Pandas documentation](https://pandas.pydata.org/docs/)
2. Practice with real-world datasets from Kaggle or UCI ML Repository
3. Learn about MultiIndex for hierarchical indexing
4. Study time series analysis techniques
5. Integrate Pandas with visualization libraries (Matplotlib, Seaborn)
6. Learn performance optimization techniques (categorical types, chunking)

---

**Happy Data Wrangling with Pandas!** 🐼
