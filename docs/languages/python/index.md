# __:material-language-python: Python__

Python is completely object oriented, and not "statically typed". You do not need to declare variables before using them, or declare their type. Every variable in Python is an object.

## Variables and Types

??? note "Numbers"

    Python supports two types of numbers - integers(whole numbers) and floating point numbers(decimals).

    ```python
    myint = 7
    print(myint)            # 7

    myfloat = 7.0
    print(myfloat)          # 7.0
    myfloat = float(7)
    print(myfloat)          # 7.0
    ```

??? note "Strings"

    Python supports two types of numbers - integers(whole numbers) and floating point numbers(decimals).

    ```python
    hello = "hello"
    world = "world"
    helloworld = hello + " " + world
    print(helloworld)                   # Output: hello world

    one = 1
    two = 2
    hello = "hello"
    print(one + two + hello)            # This will not work!

    astring = "Hello world!"
    print(astring.index("o"))                   # Output: 4, because the location of the first occurrence of "o" is 4 characters away
    print(astring.count("l"))                   # Output: 3
    print(astring[3:7:2])                       # Output: l
    print(astring[::-1])                        # Output: !dlrow olleH
    print(astring.upper())                      # Output: HELLO WORLD!
    print(astring.lower())                      # Output: hello world!
    print(astring.startswith("Hello"))          # Output: True      
    print(astring.endswith("asdfasdfasdf"))     # Output: False
    print(astring.split(" "))                   # Output: ['Hello', 'world!']      
    ```

    __String Formatting__

    ??? note "`%` Python 1.0"

        - `%s`: String (or any object with a string representation, like numbers)
        - `%d`: Integers
        - `%f`: Floating point numbers
        - `%.<number of digits>f`: Floating point numbers with a fixed amount of digits to the right of the dot.
        - `%x/%X`: Integers in hex representation (lowercase/uppercase)

        ```python
        # This prints out "John is 23 years old."
        name = "John"
        age = 23
        print("%s is %d years old." % (name, age))

        # This prints out: A list: [1, 2, 3]
        mylist = [1,2,3]
        print("A list: %s" % mylist)
        ```

    ??? note "`format()` Python 2.6"

        ```python
        "{0} is {1}".format(name, age)
        ```

    ??? note "`f-strings` Python 3.6+"

        ==Faster==

        ```python
        f"{name} is {age}"
        ```

??? note "List"

    - A collection that is __ordered__, and __mutable__.
    - __Heterogeneous__: A single list can store different data types, such as integers, strings, and even other lists.

    | Method | Action |
    |---|---|
    | append(item) | Adds item to the end of the list |
    | extend(iterable) | Appends all elements from another list or iterable |
    | insert(i, item) | Inserts item at a specific index i |
    | remove(item) | Removes the first occurrence of item |
    | pop([i]) | Removes and returns the item at index i (defaults to last) |
    | sort() | Sorts the list in-place (ascending by default) |
    | reverse() | Reverses the order of the list in-place |
    | index(item) | Returns the index of the first occurrence of item |

    ```python
    mylist = [1,2,3,4,5,6]
    len(mylist)                 # 6
    print(mylist[10])           # Throws error list index out of range

    # Slicing: Extract a portion using list[start:stop:step].
    print(mylist[1:4:2])        # [2, 4]

    mylist = [x**2 for x in range(5)]
    print(mylist)               # [0, 1, 4, 9, 16]
    ```

??? note "Tuple"

    A collection that is __ordered__ and __immutable__ (unchangeable)

    ```python
    # Creation
    fruits = ("apple", "banana", "cherry")
    print(fruits[0])   # Output: apple
    print(fruits[-1])  # Output: cherry (last item)

    fruits.append("mango")      # Fails

    # Single element tuple (requires a trailing comma)
    single_item = ("apple",) 

    coordinates = (4, 5)
    x, y = coordinates
    # x is 4, y is 5
    ```

??? note "Set"

    __Unordered__ collection of __unique__ elements.

    ```python    
    empty_set = set()               # Do not use {}, as it creates empty dictionary

    birds = {"Eagle", "Hawk"}

    birds.add("Owl")                    # Adds one
    birds.update(["Crow", "Sparrow"])   # Adds many
    birds.remove("Owl")                 # Works fine
    # birds.remove("Owl")               # ! Raises KeyError

    birds.discard("Owl")                # No error, even though "Owl" isn't there

    removed_item = birds.pop()          # Removes a random item (sets are unordered)
    birds.clear()                       # birds is now set()

    frontend = {"HTML", "CSS", "JS", "React"}
    backend = {"Python", "JS", "Node", "SQL"}

    # 1. Union (|) - Everyone invited
    all_skills = frontend | backend
    # Output: {'HTML', 'CSS', 'JS', 'React', 'Python', 'Node', 'SQL'}

    # 2. Intersection (&) - The common ground
    fullstack_bridge = frontend & backend
    # Output: {'JS'}

    # 3. Difference (-) - The "Only in A"
    pure_frontend = frontend - backend
    # Output: {'HTML', 'CSS', 'React'}

    # 4. Symmetric Difference (^) - The "Or but not both"
    unique_skills = frontend ^ backend
    # Output: {'HTML', 'CSS', 'React', 'Python', 'Node', 'SQL'}

    small = {1, 2}
    big = {1, 2, 3, 4}

    print(small.issubset(big))   # Output: True
    print(big.issuperset(small)) # Output: True
    print(1 in small)            # Output: True (O(1) speed!)
    ```

??? note "Dict"

    A collection of key-value pairs with unique keys

    ```python
    user = {"name": "Dev", "role": "Admin"}

    # Add a new key-value pair
    user["email"] = "dev@example.com"

    # Update an existing value
    user["role"] = "Superuser"

    # Update multiple items at once using .update()
    user.update({"status": "active", "level": 5})
    # Output: {'name': 'Dev', 'role': 'Superuser', 'email': 'dev@example.com', 'status': 'active', 'level': 5}

    print(user.keys())
    # Output: ['name', 'role', 'email', 'status', 'level']
    print(user.values())
    # Output: ['Dev', 'Superuser', 'dev@example.com', 'active', 5]

    # Returns None instead of crashing
    user_id = user.get("id") 

    # Returns a custom default value
    user_id = user.get("id", "Unknown ID") 

    stats = {"hp": 100, "mp": 50, "xp": 10}

    # Remove and store the value
    removed_xp = stats.pop("xp") 
    # Output: removed_xp is 10, stats is {"hp": 100, "mp": 50}

    # Safe removal (returns None if key is missing instead of crashing)
    gold = stats.pop("gold", 0) 
    # Output: gold is 0, stats remains unchanged

    del stats["xp"]
    # ! CRASHES with KeyError because "xp" isn't there

    stats.clear()
    # Output: {}

    for key, value in user.items():
        print(f"The {key} is {value}")
    ```

## Operators

```python
number = 1 + 2 * 3 / 4.0
remainder = 11 % 3          # 2
squared = 7 ** 2
cubed = 2 ** 3

a = = "hello" * 2
print(a)                    # hellohello

even_numbers = [2,4,6,8]
odd_numbers = [1,3,5,7]
all_numbers = odd_numbers + even_numbers
print(all_numbers)          # [1, 3, 5, 7, 2, 4, 6, 8]

print([1,2,3] * 3)          # [1, 2, 3, 1, 2, 3, 1, 2, 3]
```

## Conditions

```python
if statement is True:
    # do something
    pass
elif another_statement is True: # else if
    # do something else
    pass
else:
    # do another thing
    pass

x = 2
print(x == 2)   # True
print(x == 3)   # False
print(x < 3)    # True

name = "John"
age = 24
print(name == "John" and age == 23)     # False
print(name == "John" or age == 23)      # True

statement = False
print(statement is True)    # False

x = [1,2,3]
y = [1,2,3]
print(x == y) # True
print(x is y) # False

print(not False) 
```

## Loops

??? note "For"

    ```python
    primes = [2, 3, 5, 7]
    for prime in primes:
        print(prime)

    for x in range(3, 8, 2):
        print(x)
    # Output: 3,5,7
    ```

??? note "While"

    ```python
    count = 0
    while count < 5:
        print(count)
        count += 1
    # Output: 0,1,2,3,4
    ```

??? note "break and continue"

    ```python 
    count = 0
    while True:
        print(count)
        count += 1
        if count >= 5:
            break

    # Prints out only odd numbers - 1,3,5,7,9
    for x in range(10):
        if x % 2 == 0:
            continue
        print(x)
    ```

## Functions

```python
def sum_two_numbers(a, b):
    return a + b

# With type hinting
def add(a: int, b: int) -> int:
    return a + b

x = sum_two_numbers(1,2)
```

### Types

```python
age: int = 25
name: str = "Test"
price: float = 10.5

from typing import List, Dict, Tuple, Set

numbers: list[int] = [1, 2, 3]
scores: dict[str, int] = {"math": 90}
point: tuple[int, int] = (10, 20)
unique_ids: set[int] = {1, 2, 3}

# Optional Type
from typing import Optional

def get_user(user_id: int) -> Optional[str]:
    pass

# Union Type
from typing import Union

def parse(value: int | str):
    pass

# Type Hints for Functions
from typing import Callable

def process(func: Callable[[int, int], int]) -> int:
    return func(2, 3)
```

??? note "Lambda"

    ```python
    lambda x: x * 2
    ```


??? note "`*args`"

    variable positional arguments

    ```python
    def my_sum(*args):
        """Calculates the sum of any number of arguments."""
        total = 0
        for num in args:
            total += num
        return total

    print(my_sum(1, 2, 3))
    # Output: 6
    ```

??? note "`**kwargs`"

    variable keyword arguments

    ```python
    def print_details(**kwargs):
        """Prints key-value pairs of any number of keyword arguments."""
        for key, value in kwargs.items():
            print(f"{key}: {value}")

    print_details(name="Alice", age=30, city="New York")
    # Output:
    # name: Alice
    # age: 30
    # city: New York
    ```


## Classes and Objects

Key idea: Use `_protected` or `__private` variables to control access.

```python
class Demo:
    # Class attribute (shared by all instances)
    class_variable = 'class'

    def __init__(self, instance_variable_1, instance_variable_2):
        # Instance attributes (unique to each object)
        self.instance_variable_1 = instance_variable_1
        self.instance_variable_2 = instance_variable_2
        self._a = 10            # protected
        self.__b = 20           # private
        # Demo.class_variable += 1

    def greet(self):
        return f"Hello, I am {self.name} (v{self.version})"

# Intantiating class objects
a_obj = Demo("R2-D2", 1.0)
b_obj = Demo("C#-PO", 2.0)

# Modifying class variables

## Via Instance (Not recommended)
a_obj.class_variable = "a"
# Changing class variable using object ensures 
# a_obj gets a copy of class_variable
# b_obj still points to shared class_variable
print(a_obj.class_variable)         # Output: a
print(b_obj.class_variable)         # Output: class
```

```python title="Dunder Methods"
class ShoppingCart:
    def __init__(self, owner: str):
        """1. CONSTRUCTOR: Initializes the object state."""
        self.owner = owner
        self.items = []

    # STRING: What 'print(obj)' shows to users.
    def __str__(self) -> str:
        return f"Cart belonging to {self.owner} with {len(self.items)} items."

    # REPRESENTATION: What developers see in the console/logs.
    def __repr__(self) -> str:
        return f"ShoppingCart(owner='{self.owner}', items={self.items})"

    # LENGTH: Allows 'len(obj)' to work.
    def __len__(self) -> int:
        return len(self.items)

    # INDEXING: Allows 'obj[0]' access like a list.
    def __getitem__(self, index: int):
        return self.items[index]

    # ADDITION: Allows 'obj + "item"' syntax.
    def __add__(self, other_item: str):
        self.items.append(other_item)
        return self  # Return self to allow chaining

    # MEMBERSHIP: Allows 'if "Apple" in obj' syntax.
    def __contains__(self, item: str) -> bool:
        return item in self.items

    # CALLABLE: Allows 'obj()' to run like a function.
    def __call__(self):
        print(f"Processing checkout for {self.owner}...")

    # EQUAL: Checks if two instances are the same.
    def __eq__(self, other):
        return (self.owner === other.owner) and (self.items === other.items)

    # LESS THAN: Checks if current instances less than other.
    def __lt__(self, other):
        return (self.owner < other.owner)

# --- DEMONSTRATION ---
my_cart = ShoppingCart("Alice")

# __add__
my_cart + "Laptop" + "Mouse" 

# __len__
print(len(my_cart))       # Output: 2

# __str__
print(my_cart)            # Output: Cart belonging to Alice with 2 items.

# __getitem__
print(my_cart[0])         # Output: Laptop

# __contains__
print("Mouse" in my_cart) # Output: True

# __call__
my_cart()                 # Output: Processing checkout for Alice...
```

### self vs cls vs static

==In short: use cls when the function needs to know about the Class itself, and use static when the function is just a helper that doesn't need to touch any data.==

```python hl_lines="27-30"
class Employee:
    company = "TechCorp"  # Class Attribute (Shared)

    def __init__(self, name, salary):
        self.name = name   # Instance Attribute (Unique)
        self.salary = salary

    # 1. Instance Method (Uses self)
    def get_details(self):
        return f"{self.name} works at {self.company}"

    # 2. Class Method (Uses cls)
    @classmethod
    def change_company(cls, new_name):
        cls.company = new_name  # Changes it for EVERYONE

    # 3. Static Method (Uses nothing)
    @staticmethod
    def is_work_day(day):
        return day not in ["Saturday", "Sunday"]


# --- How they behave ---
emp1 = Employee("Alice", 90000)
emp2 = Employee("Bob", 80000)

# Changing via 'cls' affects all objects
Employee.change_company("InnovateX") 
print(emp1.company) # Output: InnovateX
print(emp2.company) # Output: InnovateX

# Static methods are just helpers
print(Employee.is_work_day("Monday")) # Output: True
```


### Inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print("Some sound")

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed
    
    # Method overriding
    def speak(self):
        print("Bark")
```

__Diamond Problem__

```python
class A:
    def show(self):
        print("A")

class B(A):
    def show(self):
        print("B")

class C(A):
    def show(self):
        print("C")

class D(B, C):
    pass

d = D()
d.show()            # Output: B
# Since B comes before C

# Because Python follows a Method Resolution Order (MRO).
print(D.mro())      # [D, B, C, A, object]
```

??? warning "Problem"

    The real issue appears when constructors or methods call parent classes.

    ```python
    class A:
        def __init__(self):
            print("A init")

    class B(A):
        def __init__(self):
            A.__init__(self)
            print("B init")

    class C(A):
        def __init__(self):
            A.__init__(self)
            print("C init")

    class D(B, C):
        def __init__(self):
            B.__init__(self)
            C.__init__(self)
            print("D init")

    D()
    ```

    ```text title="Output"
    A init
    B init
    A init
    C init
    D init
    ```

    ==A gets called twice, which can break logic (duplicate initialization, repeated DB connections, etc.).==

??? success "Solution"

    ```python
    class A:
        def __init__(self):
            print("A init")

    class B(A):
        def __init__(self):
            super().__init__()
            print("B init")

    class C(A):
        def __init__(self):
            super().__init__()
            print("C init")

    class D(B, C):
        def __init__(self):
            super().__init__()
            print("D init")

    D()
    ```

    ```text title="Output"
    A init
    C init
    B init
    D init
    ```


## Modules and Packages

Here `game.py` implements the game. `draw` module that implements the logic for drawing the game on the screen.

```python title="mygame/draw.py"
def draw_game():
    ...

def clear_screen(screen):
    ...
```

```python title="mygame/game.py"
import draw

def play_game():
    ...

def main():
    result = play_game()
    draw.draw_game(result)

# this means that if this script is executed, then 
# main() will be executed
if __name__ == '__main__':
    main()
```

When the `import draw` directive runs, the Python interpreter looks for a file in the directory in which the script was executed with the module name and a `.py` suffix. In this case it will look for `draw.py`. If it is found, it will be imported. If it's not found, it will continue looking for built-in modules.

You may have noticed that when importing a module, a `.pyc` file is created. This is a compiled Python file. Python compiles files into Python bytecode so that it won't have to parse the files each time modules are loaded.

We can import the function `draw_game` into the main script's namespace by using the `from` command.

```python
from draw import draw_game
# import the draw module
# from draw import *

def main():
    result = play_game()
    draw_game(result)
```

```python title="Custom import name"
import draw_textual as draw
```

Each module is loaded only once, so local variables inside the module act as a singleton

### Extending module load path

Python interpreter can be configured to lookf for modules other than default local directory and built-in modules.

- Use environment variable `PYTHONPATH` to specify additional directories to look for modules
- `sys.path.append`

```bash
PYTHONPATH=/foo python game.py
# Executes game.py, and enables the script to load modules from the foo directory, as well as the local directory.
```

```python
# Execute it before running the import command
sys.path.append("/foo")
import ...
```

### Exploring built-in modules

```python
# Use dir to know which functions are implemented in each module
import urllib
dir(urllib)

# Use help to learn more about specific function
help(urllib.urlopen)
```

### Packages

Packages are namespaces containing multiple packages and modules. They're just directories, and MUST contain a special file called `__init__.py`.

If we create a directory called `foo`, which marks the package name, we can then create a module inside that package called `bar`. Then we add the `__init__.py` file inside the foo directory.

```python
import foo.bar
# or
from foo import bar
```

The `__init__.py` file can also decide which modules the package exports as the API, while keeping other modules internal, by overriding the `__all__` variable.

```python
__all__ = ["bar"]
```

## Generators

Generators in Python are a way to create iterators that produce values one at a time instead of storing the whole sequence in memory. They are useful for large datasets, streams, or pipelines.

A function becomes a generator when it uses `yield`. `yield` pauses the function and resumes from the same state next time.

```python
def fib():
    a, b = 1, 1
    while 1:
        yield a
        a, b = b, a + b

counter = 0
for n in fib():
    print(n)
    if counter == 5:
        break
    counter += 1
# Output: 1, 1, 2, 3, 5
```

## Event Loop vs multi-threading

The event loop uses a single thread with a non-blocking, asynchronous approach, ideal for ==I/O-bound tasks==, while multi-threading creates multiple threads to run tasks in parallel, making it suitable for ==CPU-bound tasks.==

### Event Loop Model

The event loop operates on a single thread and uses a queue to manage incoming events and callbacks

When an I/O operation is initiated, it's delegated to the OS, and the main thread continues processing other tasks. Once the I/O operation is complete, a callback is placed in a queue, and the event loop picks it up when the main thread is free.

??? success "Strengths"

    - __Scalability for I/O-bound tasks__: Handles a high number of concurrent connections efficiently with low resource overhead (fewer threads to manage).
    - __Simpler programming model__: Avoids the complexities of managing shared memory and concurrency issues like race conditions and deadlocks, as there's only one thread of execution for the main logic.

??? danger "Weaknesses"

    - __Poor for CPU-bound tasks__: A single long-running, CPU-intensive task will block the entire event loop, making the application unresponsive.
    - __No true parallelism__: Cannot utilize multiple CPU cores by default within a single process. 

#### Coroutines

Specialized functions that can pause their execution using the await keyword and yield control back to the event loop

The key principle is that coroutines voluntarily `yield` control to the event loop.

```python
import asyncio
import time

async def my_task(seconds):
    print(f"start doing task for {seconds} s")
    await asyncio.sleep(seconds) # Yields control
    print(f"task for {seconds} s finished")
    return seconds

async def main():
    start_time = time.time()
    # Create tasks to run concurrently
    task_3s = asyncio.create_task(my_task(3))
    task_5s = asyncio.create_task(my_task(5))

    # Wait for tasks to complete
    result_3s = await task_3s
    result_5s = await task_5s

    print(f"Task 3s result: {result_3s}, Task 5s result: {result_5s}")
    print(f"total execution time {time.time() - start_time:.2f} s")

# The entry point to run the event loop
asyncio.run(main())
```

### Multi-Threading Model

In a multi-threading model, multiple threads of execution are created, each capable of running in parallel on different CPU cores.

a new thread is created for each request or task. The operating system manages the scheduling and context switching between these threads. Threads may block while waiting for I/O operations, but the OS can switch to another ready thread, ensuring the CPU stays busy.

__Strengths:__

- __True parallelism__: Excels at CPU-bound tasks by distributing the workload across multiple cores, leading to faster execution.
- __Responsiveness__: A blocking operation in one thread does not significantly impact the progress of other threads.

__Weaknesses:__

- __Higher overhead__: Creating and managing many threads consumes more system resources (memory and CPU time for context switching).
- __Complexity__: Requires careful handling of shared state, leading to potential bugs like race conditions and deadlocks, which are difficult to debug.