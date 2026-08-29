- **Initialization** gives an object its initial value when the object is created.
- **Assignment** gives a new value to an object that already exists.

```cpp
int a;         // default-initialization (no initializer)

// Traditional initialization forms:
int b = 5;     // copy-initialization (initial value after equals sign)
int c ( 6 );   // direct-initialization (initial value in parenthesis)

// Modern initialization forms (preferred):
int d { 7 };   // direct-list-initialization (initial value in braces)
int e {};      // value-initialization (empty braces)
```

---

# Overview

|Type|Syntax|Main purpose|Notes|
|---|---|---|---|
|**Default initialization**|`T x;`|Create without explicitly providing an initial value. Doesn't have an initializer|Variable is left with an indeterminate value.|
|**Value initialization**|`T x{};`|Create with a default/zero value|==Preferred.== Implicitly initializes the variable to zero (or whatever value is closest to zero for a given type). In cases where zeroing occurs, this is called **zero-initialization**. For class types, value-initialization (and default-initialization) may instead initialize the object to predefined default values, which may be non-zero.|
|**Copy initialization**|`T x = value;`|Initialize from another value|Inherited from C. Copies the value on the right-hand side of the assignment operator (=) into the variable being created on the left-hand side. Less efficient than other forms of initialization for some complex types. However, C++17 remedied the bulk of these issues.|
|**Direct initialization**|`T x(value);`|Directly invoke a matching constructor|Initially introduced to allow more efficient complex object initialization, superseded by Direct-list initialization.|
|**Direct-list initialization**|`T x{value};`|Initialize using braces|==Preferred.== Most efficient and satisfies complex object use cases. Disallows [[Narrowing-Conversions]]. Provides a way to initialize objects with a list of values rather than a single value.|
|**Copy-list initialization**|`T x = {value};`|List initialization using `=`|Less commonly needed|
|**Copy initialization from object**|`T b = a;`|Construct `b` from `a`|Generally invokes a **copy construction** for class types: `T(const T&);`|
|**Move initialization**|`T b = std::move(a);`|Transfer resources from `a` into new `b`|Generally invokes a **move constructor**: `T(T&&);`. Useful for movable resource-owning objects. `std::move` does not itself physically transfer resources. Instead, it essentially changes how the expression is treated by the compiler so that a move constructor or move-assignment operator can be selected to perform the actual resource transfer. `a` will be in a valid but **unspecified state** without holding the original value.|
|**Copy assignment**|`b = a;`|Replace `b`'s value with `a`'s value|Generally invokes a **copy-assignment operator** for class types: T& operator=(const T&);|
|**Move assignment**|`b = std::move(a);`|Transfer resources into existing `b`|Generally invokes a **move-assignment operator**: `T& operator=(T&&);`. Useful for movable objects|
|**Compound assignment**|`x += y;`, `x *= y;`, etc.|Modify existing value using an operation|Preferred over repeated expressions|

^72ae40

> [!question]- When should I initialize with { 0 } vs {}?
> Use direct-list-initialization when you’re actually using the initial value:
>
> ```cpp
> int x { 0 };    // direct-list-initialization with initial value 0
> std::cout << x; // we're using that 0 value here
> ```
>
> Use value-initialization when the object’s value is temporary and will be replaced:
>
> ```cpp
> int x {};      // value initialization
> std::cin >> x; // we're immediately replacing that value so an explicit 0 would be meaningless
> ```

> [!question]- Why move instead of copy? (Move Initialization + Assignment)
> Suppose an object owns a large dynamically allocated memory buffer.
> 
> A conceptual copy might require:
> 
> ```text
> a
> │
> ├── allocate new memory for b
> │
> ├── copy thousands/millions of elements
> │
> └── produce b
> ```
> 
> This can be expensive.
> 
> A move can instead transfer ownership:
> 
> ```text
> Before:
> 
> a ─────────→ resource
> b does not exist
> 
> 
> Move:
> 
> a ─────────→ resource
>       transfer
>           ↓
> b ─────────→ resource
> 
> 
> After:
> 
> a → valid but unspecified state
> b → owns transferred resource
> ```
> 
> No large element-by-element copy may be required.

---

# Important Exception: `{}` Can Change Constructor Selection

Consider:

```cpp
std::vector<int> a(5);
```

versus:

```cpp
std::vector<int> b{5};
```

### Parentheses ()

```cpp
std::vector<int> a(5);
```

means:

> Construct a vector containing 5 default-initialized integers.

Result:

```text
a = {0, 0, 0, 0, 0}
```

because the constructor approximately corresponds to:

```cpp
vector(size_type count);
```

---

### Braces {}

```cpp
std::vector<int> b{5};
```

means:

> Construct a vector from the initializer list `{5}`.

Result:

```text
b = {5}
```

---

# Compound Assignment Operators

|Operator|Rough meaning|
|---|---|
|`x += y`|`x` becomes `x + y`|
|`x -= y`|`x` becomes `x - y`|
|`x *= y`|`x` becomes `x * y`|
|`x /= y`|`x` becomes `x / y`|
|`x %= y`|`x` becomes `x % y`|
|`x &= y`|Bitwise AND then assign|
|`x \|= y`|Bitwise OR then assign|
|`x ^= y`|Bitwise XOR then assign|
|`x <<= y`|Left shift then assign|
|`x >>= y`|Right shift then assign|

---

# References Must Be Initialized

A reference must normally be bound when it is created.

Correct:

```cpp
int x{5};
int& ref{x};
```

Incorrect:

```cpp
int& ref;
```

A reference cannot normally exist without referring to an object.

The reference is another name for `x`.

Therefore:

```cpp
ref = 10;
```

does **not** cause `ref` to begin referring to `10`.

Instead:

```text
ref refers to x
       ↓
ref = 10
       ↓
x = 10
```

References cannot normally be reseated after initialization.

---

# Pointer Initialization and Assignment

Pointers follow ordinary initialization and assignment rules.

Example:

```cpp
int x{5};

int* ptr{&x};
```

This initializes `ptr` with the address of `x`.

Conceptually:

```text
x
┌─────┐
│  5  │
└─────┘
  ↑
  │ address stored in
  │
ptr
```

Later:

```cpp
int y{10};

ptr = &y;
```

is pointer **assignment**.

Now the pointer stores the address of `y`.

```text
x = 5

y = 10
↑
│
ptr
```

Unlike references, pointers can normally be reassigned to point somewhere else.

---

# `const` Objects Must Be Initialized

Consider:

```cpp
const int x{5};
```

Because `x` cannot later be assigned another value, its value must normally be established when it is created.

This is invalid:

```cpp
const int x;
x = 5;
```

The object's value cannot be supplied afterward through ordinary assignment.

Therefore:

```text
const object
      ↓
must receive its appropriate value during initialization
      ↓
ordinary assignment afterward is prohibited
```

---

# Constructor and Assignment Functions

For a class `T`, the important special member functions conceptually include:

```cpp
T();                 // default constructor

T(const T& other);   // copy constructor

T(T&& other);        // move constructor

T& operator=(const T& other); // copy assignment

T& operator=(T&& other);      // move assignment

~T();                // destructor
```

Their roles are:

|Function|Purpose|
|---|---|
|`T()`|Construct an object with a default state|
|`T(const T&)`|Construct a new object by copying another|
|`T(T&&)`|Construct a new object by moving from another|
|`operator=(const T&)`|Copy into an already-existing object|
|`operator=(T&&)`|Move into an already-existing object|
|`~T()`|Destroy an object and clean up its resources|

Conceptually:

```text
              Object does not exist
                      │
                      ↓
                CONSTRUCTION
                 /         \
              copy         move
               │             │
               ↓             ↓
         Copy constructor  Move constructor
                      │
                      ↓
                  Object exists
                      │
            ┌─────────┴─────────┐
            ↓                   ↓
      Copy assignment      Move assignment
            │                   │
            └─────────┬─────────┘
                      ↓
                  Object exists
                      │
                      ↓
                  Destructor
                      │
                      ↓
              Object lifetime ends
```