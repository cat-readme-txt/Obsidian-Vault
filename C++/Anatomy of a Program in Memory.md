## Overview
Consider:
```cpp
int global = 1;

void foo() {
    static int calls = 0;
    int local = 5;

    int* p = new int{10};

    delete p;
}

int main() {
    foo();
}
```
Conceptually:
```
CODE / STATIC
┌───────────────────────────────┐
│ machine code for main(), foo()│
│ global = 1                    │
│ calls = 0                     │
└───────────────────────────────┘

HEAP
┌───────────────────────────────┐
│ dynamically allocated int 10  │
└───────────────────────────────┘

STACK
┌───────────────────────────────┐
│ main() frame                  │
├───────────────────────────────┤
│ foo() frame                   │
│ local = 5                     │
│ p = address of heap object    │
└───────────────────────────────┘

```

---

## Code / Static Region

Allocation of memory for variables declared as static and global variables.

Conceptually related in lifetime, exists from the start to the end of the program.

Three parts:
- program code
- global variables
- static variables

#### Program Code
The compiled machine code also lives in this region of memory, called the executable-code region.

#### Global & Static Variables
Example:
```cpp
int global_x = 5;			// global static storage duration variable
static int global_y = 10;	// global static storage duraction variable

int main() {
    static int count = 0;	// static storage duration local variable
    int num = 0;			// automatic storage duration local variable
}
```

Here, all variables have a static storage duration except for `num`, meaning that their lifetime spans from the start to the end of execution of the program and are not recreated.

> [!question]- Why is `global_x` also a static storage duration variable?
> Variables declared in the global scope are automatically have static storage duration, the `static` keyword for `global_y` changes the variable from the automatic `external linkage` (accessible by the linker across different files through keyword `extern`) to `internal linkage` (restricted access by the linker to be only readable in that specific file where it is declared).

---

## Heap / Free Storage

Dynamic allocation of objects on the free store.

**The size of objects may be unknown at compile-time**

Can request memory for these objects at run-time.

For example:

```
int* p = new int{42};
```

Here there are actually two separate objects:

```
stack                     heap
┌─────────────┐          ┌──────────┐
│ p           │─────────→│ int = 42 │
└─────────────┘          └──────────┘
```

`p` itself is usually an automatic local variable, so the pointer may be on the stack.

But the object created by:

```
new int{42}
```

has dynamic storage duration and lives in dynamically allocated storage.

It remains alive until:

```
delete p;
```

when `foo()` returns:

```
p disappears
```

but the dynamically allocated integer does **not** automatically disappear.

That produces a memory leak if nothing else owns its address.

This is why modern C++ strongly prefers RAII types such as:

```
std::vector
std::string
std::unique_ptr
std::shared_ptr
```

---

## Stack

Allocation of memory to variables on the stack.

**The size of variables must be known at compile-time.**

The stack is commonly used for **automatic storage duration**.

```cpp
void foo() {
    int x = 10;
    double y = 3.14;
}
```

Here, `x` and `y` live in the function's [[Stack Frame]]. 

When `foo()` is called, a new stack frame is created.

When `foo()` returns, `x` and `y` **automatically deallocate and destroy**

> [!question]- Why do Hep and Stack "grow" towards each other?
> Historically, a process might look roughly like:
> 
> ```
low addresses
┌──────────────────┐
│ code             │
├──────────────────┤
│ static data      │
├──────────────────┤
│ heap             │
│ ↓ grows upward   │
│                  │
│ free space       │
│                  │
│ ↑ grows downward │
│ stack            │
└──────────────────┘
high addresses
> ```
> ##### Why arrange it this way?
> 
> Because neither the heap nor the stack has a fixed size known ahead of time.
> 
> If they start at opposite ends, each can grow into unused address space.
> 
> But this is only a **conceptual/classical model**.
> 
> Modern operating systems use:
> - virtual memory
> - address-space randomization
> - memory mappings
> - multiple thread stacks
> - shared libraries
> - allocator arenas
> - guard pages
> so the real layout is considerably more complicated.