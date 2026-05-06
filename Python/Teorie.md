# Python — Plan Complet de Atac pentru Interviu

---

## PARTEA 1 — TEORIE: Concepte Esențiale

---

### 1. Tipuri de Date & Mutabilitate

Unul dintre cele mai frecvente subiecte de teorie la interviuri Python.

| Tip | Mutable | Ordonat | Duplicate | Sintaxă |
|---|---|---|---|---|
| `list` | ✅ | ✅ | ✅ | `[1, 2, 3]` |
| `tuple` | ❌ | ✅ | ✅ | `(1, 2, 3)` |
| `dict` | ✅ | ✅ (Python 3.7+) | ❌ chei | `{"a": 1}` |
| `set` | ✅ | ❌ | ❌ | `{1, 2, 3}` |
| `frozenset` | ❌ | ❌ | ❌ | `frozenset({1, 2})` |
| `str` | ❌ | ✅ | ✅ | `"hello"` |

```python
# Mutable vs Immutable — efectul în funcții
def modify_list(lst):
    lst.append(4)       # modifică originalul!

def modify_tuple(tpl):
    # tpl += (4,)       # creează un obiect NOU, nu modifică originalul

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)          # [1, 2, 3, 4] — modificat!

# Tuple ca cheie în dict (posibil deoarece e immutable/hashable)
coords = {(0, 0): "origin", (1, 2): "point A"}
```

> **Întrebare tipică:** *"De ce nu poți folosi o listă ca cheie într-un dicționar?"*
> **Răspuns:** Cheile unui dict trebuie să fie hashable (immutable). Lista e mutable, deci nu are hash stabil.

---

### 2. List, Dict, Set — Operații și Complexitate

```python
# --- LIST ---
lst = [3, 1, 4, 1, 5, 9, 2, 6]

lst.append(7)           # O(1) — adaugă la final
lst.insert(0, 0)        # O(n) — inserare la poziție arbitrară
lst.pop()               # O(1) — șterge ultimul element
lst.pop(0)              # O(n) — șterge primul element (costisitor!)
lst.remove(4)           # O(n) — caută și șterge prima apariție
lst.sort()              # O(n log n) — in-place
sorted(lst)             # O(n log n) — returnează listă nouă
lst.index(5)            # O(n) — caută indexul unui element
5 in lst                # O(n) — căutare liniară

# --- DICT ---
d = {"name": "Ana", "age": 25}

d["city"] = "Iași"      # O(1) — adăugare/modificare
d.get("city", "N/A")    # O(1) — get cu valoare default (nu aruncă KeyError)
d.pop("age")            # O(1) — ștergere
"name" in d             # O(1) — verificare cheie (mult mai rapid ca în list!)
d.keys()                # view al cheilor
d.values()              # view al valorilor
d.items()               # view al perechilor (k, v)
d.update({"x": 1})      # merge cu alt dict

# --- SET ---
s = {1, 2, 3, 4, 5}

s.add(6)                # O(1)
s.remove(3)             # O(1) — aruncă KeyError dacă nu există
s.discard(99)           # O(1) — nu aruncă eroare
3 in s                  # O(1) — mult mai rapid ca în list!

# Operații pe seturi
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
a & b                   # {3, 4}     — intersecție
a | b                   # {1,2,3,4,5,6} — reuniune
a - b                   # {1, 2}     — diferență
a ^ b                   # {1, 2, 5, 6} — diferență simetrică
```

---

### 3. Comprehensions

```python
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# List comprehension
patrate_pare = [x**2 for x in nums if x % 2 == 0]
# [4, 16, 36, 64, 100]

# Dict comprehension
lungimi = {word: len(word) for word in ["cat", "elephant", "hi"]}
# {"cat": 3, "elephant": 8, "hi": 2}

# Set comprehension — elimină duplicate automat
litere_unice = {char for char in "mississippi"}
# {'m', 'i', 's', 'p'}

# Nested list comprehension (flatten)
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

### 4. Lambda, Map, Filter, Zip

```python
# Lambda — funcție anonimă
square = lambda x: x**2
print(square(5))        # 25

# Map — aplică o funcție pe fiecare element
nums = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, nums))       # [2, 4, 6, 8, 10]

# Filter — filtrează elementele
positive = list(filter(lambda x: x > 0, [-1, 2, -3, 4]))  # [2, 4]

# Combinate
result = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, nums)))
# [4, 16] — pătratele numerelor pare

# Zip — combină două liste
names = ["Ana", "Ion", "Maria"]
scores = [95, 87, 92]
combined = list(zip(names, scores))
# [("Ana", 95), ("Ion", 87), ("Maria", 92)]

# Zip pentru dict din două liste
d = dict(zip(names, scores))
# {"Ana": 95, "Ion": 87, "Maria": 92}

# Sorted cu key
employees = [{"name": "Ana", "salary": 3000}, {"name": "Ion", "salary": 5000}]
sorted_emp = sorted(employees, key=lambda x: x["salary"], reverse=True)
```

---

### 5. Generators & Iterators

```python
# Generator function — folosește yield în loc de return
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

gen = fibonacci()
first_10 = [next(gen) for _ in range(10)]
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Generator expression — ca list comprehension dar lazy
# Nu alocă totul în memorie dintr-o dată
big_squares_gen = (x**2 for x in range(10**6))   # instant, 0 memorie
big_squares_lst = [x**2 for x in range(10**6)]   # alocă tot în RAM

# Când folosești generator:
# - Date prea mari să încapă în memorie
# - Procesare stream (exact ca Spark — lazy evaluation!)

# next() și StopIteration
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for num in countdown(5):
    print(num)   # 5, 4, 3, 2, 1
```

> **Conexiunea cu Spark:** Lazy evaluation din Spark e inspirată din același principiu ca generatoarele Python — nu calculezi nimic până când nu ai nevoie de rezultat.

---

### 6. Decorators

```python
import time
from functools import wraps

# Decorator de bază — timer
def timer(func):
    @wraps(func)    # păstrează __name__ și __doc__ ale funcției originale
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} a rulat în {elapsed:.4f}s")
        return result
    return wrapper

@timer
def process_data(n):
    return sum(range(n))

process_data(1000000)   # process_data a rulat în 0.0234s

# Decorator cu parametri
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
            raise Exception("All attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Network error")
    return "success"
```

> **Întrebare tipică:** *"Ce face @wraps și de ce e important?"*
> **Răspuns:** Fără @wraps, funcția decorată pierde `__name__`, `__doc__` și alte metadata — devine "wrapper" în loc de numele original. Asta îngreunează debugging și logging.

---

### 7. OOP — Clase Complete

```python
class DataPipeline:
    # Class variable — partajat între toate instanțele
    pipeline_count = 0

    def __init__(self, name: str, source: str):
        self.name = name           # public
        self.source = source       # public
        self._records = []         # protected (convenție)
        self.__id = id(self)       # private (name mangling)
        DataPipeline.pipeline_count += 1

    # Regular method
    def load(self, data: list):
        self._records = data
        return self                # method chaining

    def filter_records(self, condition):
        self._records = [r for r in self._records if condition(r)]
        return self

    # Property — getter
    @property
    def record_count(self):
        return len(self._records)

    # Setter
    @record_count.setter
    def record_count(self, value):
        raise AttributeError("Cannot set record_count directly")

    # Class method — operează pe clasă, nu pe instanță
    @classmethod
    def get_pipeline_count(cls):
        return cls.pipeline_count

    # Static method — nu are acces la instanță sau clasă
    @staticmethod
    def validate_source(source: str) -> bool:
        return source.startswith(("s3://", "hdfs://", "gs://"))

    # Dunder methods
    def __len__(self):
        return len(self._records)

    def __repr__(self):
        return f"DataPipeline(name='{self.name}', records={len(self)})"

    def __str__(self):
        return f"Pipeline '{self.name}' cu {len(self)} înregistrări"

    def __contains__(self, item):
        return item in self._records

    def __iter__(self):
        return iter(self._records)


# Moștenire
class BatchPipeline(DataPipeline):
    def __init__(self, name, source, batch_size=1000):
        super().__init__(name, source)  # apelează __init__ al părintelui
        self.batch_size = batch_size

    def process_in_batches(self):
        for i in range(0, len(self._records), self.batch_size):
            batch = self._records[i:i + self.batch_size]
            yield batch   # generator!

# Utilizare
p = DataPipeline("ETL_Job", "s3://bucket/data")
p.load([1, 2, 3, 4, 5]).filter_records(lambda x: x > 2)
print(p)            # Pipeline 'ETL_Job' cu 3 înregistrări
print(len(p))       # 3
print(repr(p))      # DataPipeline(name='ETL_Job', records=3)
```

---

### 8. Error Handling

```python
# Try / Except / Else / Finally
def safe_divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("Nu se poate împărți la zero!")
        return None
    except TypeError as e:
        print(f"Tip greșit: {e}")
        return None
    else:
        # Rulează DOAR dacă nu a apărut nicio excepție
        print(f"Rezultat: {result}")
        return result
    finally:
        # Rulează ÎNTOTDEAUNA — bun pentru cleanup (close connection, etc.)
        print("Operație finalizată.")

# Excepții custom
class DataValidationError(Exception):
    def __init__(self, field, message):
        self.field = field
        super().__init__(f"Validation error on '{field}': {message}")

def validate_age(age):
    if not isinstance(age, int):
        raise DataValidationError("age", "trebuie să fie int")
    if age < 0 or age > 150:
        raise DataValidationError("age", f"valoare invalidă: {age}")
    return True
```

---

### 9. Collections — Structuri Avansate

```python
from collections import Counter, defaultdict, deque, OrderedDict, namedtuple

# Counter — frecvența elementelor
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
count = Counter(words)
# Counter({'apple': 3, 'banana': 2, 'cherry': 1})
count.most_common(2)    # [('apple', 3), ('banana', 2)]
count['apple']          # 3
count['missing']        # 0 — nu aruncă KeyError!

# defaultdict — dict cu valoare default
groups = defaultdict(list)
for word in ["cat", "dog", "car", "dot"]:
    groups[word[0]].append(word)
# {'c': ['cat', 'car'], 'd': ['dog', 'dot']}

# deque — queue/stack eficient
dq = deque([1, 2, 3])
dq.appendleft(0)        # O(1) — spre deosebire de list.insert(0, x) care e O(n)
dq.popleft()            # O(1)
dq.append(4)            # O(1)

# namedtuple — tuple cu nume de câmpuri
Employee = namedtuple('Employee', ['name', 'salary', 'dept'])
emp = Employee("Ana", 5000, "IT")
print(emp.name)         # Ana
print(emp[0])           # Ana — indexare normală funcționează
```

---

### 10. Managementul Memoriei

```python
# Stack vs Heap
# Stack: variabile locale, apeluri de funcții — gestionat automat, rapid
# Heap: obiecte create cu new/constructori — gestionat de Garbage Collector

# Garbage Collector — reference counting
import sys
x = [1, 2, 3]
print(sys.getrefcount(x))   # numărul de referințe

# id() — adresa în memorie
a = [1, 2, 3]
b = a           # b și a POINTEAZĂ LA ACELAȘI obiect
b.append(4)
print(a)        # [1, 2, 3, 4] — a e și el modificat!

c = a.copy()    # c e o COPIE independentă (shallow copy)
c.append(5)
print(a)        # [1, 2, 3, 4] — a nemodificat

# Deep copy pentru obiecte nested
import copy
nested = [[1, 2], [3, 4]]
shallow = nested.copy()
deep = copy.deepcopy(nested)

nested[0].append(99)
print(shallow[0])   # [1, 2, 99] — shallow copy e afectat!
print(deep[0])      # [1, 2] — deep copy e independent

# Context Manager — __enter__ și __exit__
class ManagedResource:
    def __enter__(self):
        print("Deschid resursa")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Închid resursa")
        return False  # nu suprimă excepțiile

with ManagedResource() as r:
    print("Folosesc resursa")
# Deschid resursa → Folosesc resursa → Închid resursa
```

---

### 11. String Methods — Quick Reference

```python
s = "  Hello, World!  "

s.strip()               # "Hello, World!" — elimină spații
s.lower()               # "  hello, world!  "
s.upper()               # "  HELLO, WORLD!  "
s.split(", ")           # ["  Hello", "World!  "]
s.replace("World", "Python")
s.startswith("Hello")   # False (are spații la început)
s.strip().startswith("Hello")  # True

"hello".capitalize()    # "Hello"
"hello world".title()   # "Hello World"
", ".join(["a", "b", "c"])  # "a, b, c"

# f-strings (Python 3.6+)
name, salary = "Ana", 5000
print(f"{name} câștigă {salary:,} RON")  # Ana câștigă 5,000 RON
print(f"{3.14159:.2f}")                  # 3.14

# Slicing
s = "Hello World"
s[0:5]      # "Hello"
s[::-1]     # "dlroW olleH" — inversare
s[6:]       # "World"
```

---

### 12. Pandas — Workflow Complet Data Engineering

```python
import pandas as pd
import numpy as np

# Creare DataFrame
df = pd.DataFrame({
    'name':   ['Ana', 'Ion', 'Maria', 'Ana', None],
    'dept':   ['IT', 'HR', 'IT', 'IT', 'HR'],
    'salary': [5000, 3000, 7000, 5000, None],
    'age':    [28, 35, 42, 28, 31]
})

# --- INSPECTIE ---
df.shape            # (5, 4) — rânduri, coloane
df.dtypes           # tipurile coloanelor
df.info()           # overview complet
df.describe()       # statistici numerice
df.head(3)          # primele 3 rânduri
df.isnull().sum()   # numărul de valori null pe coloană

# --- CLEANING ---
df.dropna(subset=['name'])                      # șterge rânduri fără name
df['salary'].fillna(df['salary'].mean())        # impute cu media
df.drop_duplicates(subset=['name', 'salary'])   # elimină duplicate
df['salary'] = df['salary'].astype(int)         # schimbă tipul

# --- FILTRARE & SELECTIE ---
df[df['salary'] > 4000]                         # filtru simplu
df[(df['dept'] == 'IT') & (df['salary'] > 4000)]# filtru multiplu
df[['name', 'salary']]                          # selectare coloane
df.loc[0:2, 'name':'dept']                      # loc — label-based
df.iloc[0:2, 0:2]                               # iloc — index-based

# --- TRANSFORMARI ---
df['salary_level'] = df['salary'].apply(
    lambda x: 'senior' if x > 5000 else 'junior'
)

df['full_info'] = df['name'] + ' - ' + df['dept']

# --- GROUPBY & AGGREGARE ---
df.groupby('dept')['salary'].mean()             # media pe dept
df.groupby('dept').agg(
    avg_salary=('salary', 'mean'),
    max_salary=('salary', 'max'),
    count=('name', 'count')
).reset_index()

# --- MERGE (echivalent JOIN) ---
dept_df = pd.DataFrame({'dept': ['IT', 'HR'], 'budget': [100000, 50000]})
merged = df.merge(dept_df, on='dept', how='left')   # LEFT JOIN

# --- PIVOT & RESHAPE ---
pivot = df.pivot_table(values='salary', index='dept',
                       aggfunc=['mean', 'max', 'count'])

melted = pivot.melt(var_name='metric', value_name='value')

# --- SORTING ---
df.sort_values('salary', ascending=False)
df.sort_values(['dept', 'salary'], ascending=[True, False])
```

---
