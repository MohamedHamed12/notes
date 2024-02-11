sources
1. [video](https://www.youtube.com/watch?v=Osa9wSMVgS0&list=PLpbZuj8hP-I6F-Zj1Ay8nQ1rMnmFnlK2f&index=10)
2. [notes](https://github.com/hemanth/functional-programming-jargon?tab=readme-ov-file)

### 1. Pure Functions

- same output for the same input 
- Easier to understand, test
### 2. Immutability

- meaning they cannot be modified after creation.
- create new data structures with the desired changes.
- Facilitates concurrency and parallelism by avoiding shared mutable state.

### 3. First-Class and Higher-Order Functions

- meaning they can be passed as arguments to other functions
- Higher-order functions take other functions as arguments
```python
# Higher-order function example in Python
def operate(operation, a, b):
    return operation(a, b)

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

result1 = operate(add, 5, 3)  # Output: 8
result2 = operate(subtract, 5, 3)  # Output: 2

```
### 4. Recursion

- Iteration is achieved through recursion rather than mutable loops.

### 5. Function Composition

- Combining multiple functions to create new functions.
- Allows for building complex functionality from simple
```python 
# Function composition example in Python
def add_two(x):
    return x + 2

def square(x):
    return x * x

add_two_then_square = lambda x: square(add_two(x))

result = add_two_then_square(3)  # Output: 25

```

### closures

allows functions to retain access to variables from their enclosing scope even after the enclosing scope has finished executing.
```python
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

counter = make_counter()
print(counter())  # Output: 1
print(counter())  # Output: 2

```

### Partial
```js
function add(a, b) {
    return a + b;
}
const add5 = add.bind(null, 5); // Partial application
console.log(add5(3)); // Output: 8

```

## Benefits

- **Readability**: easier to understand and maintain.
- **Concurrency**:  easier to reason about concurrent programs, 
- **Testability**:  isolated, easier to test in isolation.
- **Scalability**: suited for distributed and parallel computing, enabling scalability across multiple cores or machines.



## Popular Functional Programming Languages

- **Haskell**: A purely functional programming language with strong static typing and lazy evaluation.
- **Scala**: A multi-paradigm language that combines object-oriented and functional programming features, running on the JVM

## examples
```python 
def add(x, y): return x + y
def mul(x): return x * 2
def square(x): return x * x

res = lambda x, y: square(lambda x, y: mul(add(x, y)))

# built-in functions
def capitalize(s): return s.capitalize()
def split(s): return s.split()
def join(words): return ' '.join(words)

title_case = lambda s: join(map(capitalize, split(s)))

```


## python libs

1. [**toolz** ](https://github.com/pytoolz/toolz)
2. [funcy](https://github.com/Suor/funcy)
