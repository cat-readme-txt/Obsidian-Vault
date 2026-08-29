## Smart Pointers

Smart pointers use **RAII** to manage dynamically allocated resources: the resource is automatically released when its owner is destroyed.

```cpp
#include <memory>
```

### Types

| Type | Ownership | Copyable | Main use |
|---|---|---|---|
| `std::unique_ptr<T>` | One exclusive owner | No | Default choice for dynamic ownership |
| `std::shared_ptr<T>` | Multiple shared owners | Yes | Objects requiring shared ownership |
| `std::weak_ptr<T>` | Non-owning observer | Yes | Prevent cycles; safely observe shared objects |

### `std::unique_ptr`

```cpp
auto ptr = std::make_unique<int>(42); // returns std::unique_ptr<int>

std::cout << *ptr;       // 42
std::cout << ptr.get();  // Raw pointer; ptr still owns the object.
```

Only one owner is allowed:

```cpp
auto a = std::make_unique<int>(42);

auto b = a;             // Error: copy constructor is deleted.
auto b = std::move(a);  // Valid: transfers ownership; a becomes nullptr.
```

Common operations:

```cpp
ptr.get();            // Return raw pointer; ownership unchanged.
ptr.release();        // Return raw pointer and relinquish ownership; does NOT delete.
ptr.reset();          // Delete managed object and become nullptr.
ptr.reset(new T{});   // Delete current object and take ownership of the new object.
ptr.swap(other);      // Exchange ownership.
*ptr;                 // Dereference managed object.
ptr->member;          // Access managed object's member.
if (ptr) {}           // Check whether ptr owns an object.
```

`release()` transfers responsibility to the caller:

```cpp
int* raw = ptr.release();

delete raw;  // Caller must eventually release the resource.
```

Arrays:

```cpp
auto arr = std::make_unique<int[]>(5);

arr[0] = 10;  // Automatically uses delete[] when destroyed.
```

Function parameters:

```cpp
void Own(std::unique_ptr<T> ptr);   // Takes ownership.
void Modify(std::unique_ptr<T>& ptr); // Can replace or reset pointer.
void Observe(const T& object);     // Observes without taking ownership.

Own(std::move(ptr));
Observe(*ptr);
```

> After `Own(std::move(ptr))`, `ptr` is empty; do not dereference it.

### `std::shared_ptr`

```cpp
auto a = std::make_shared<int>(42);
auto b = a;  // Both own the same object.

std::cout << a.use_count();  // 2
```

The managed object is destroyed when its last shared owner is destroyed or reset:

```cpp
b.reset();  // use_count becomes 1.
a.reset();  // use_count becomes 0; managed object is destroyed.
```

Common operations:

```cpp
ptr.get();          // Return raw pointer; ownership unchanged.
ptr.reset();        // Release this owner's share.
ptr.use_count();    // Number of shared owners.
ptr.swap(other);    // Exchange ownership.
*ptr;               // Dereference.
ptr->member;        // Access a member.
if (ptr) {}         // Check whether non-null.
```

A dynamically allocated **control block** stores ownership information, including shared and weak reference counts.

```cpp
void Foo(std::shared_ptr<int> ptr) {
    std::cout << ptr.use_count();  // 2
}

auto owner = std::make_shared<int>(7);  // Count: 1.

Foo(owner);  // Passing by value temporarily creates another owner.
```

Never construct separate `shared_ptr` owners from the same raw pointer:

```cpp
T* raw = new T{};

std::shared_ptr<T> a(raw);
std::shared_ptr<T> b(raw);  // Wrong: separate control blocks; double deletion.
```

Correct:

```cpp
auto a = std::make_shared<T>();
auto b = a;  // Same control block.
```

Convert exclusive ownership into shared ownership:

```cpp
auto unique = std::make_unique<T>();

std::shared_ptr<T> shared = std::move(unique);
```

### `std::weak_ptr`

A `weak_ptr` observes an object owned by `shared_ptr` without increasing its owning reference count.

```cpp
auto owner = std::make_shared<int>(42);

std::weak_ptr<int> observer = owner;

std::cout << owner.use_count();  // 1
```

Use `lock()` to safely access the object:

```cpp
if (auto temporary_owner = observer.lock()) {
    std::cout << *temporary_owner;
}
```

After the final owner disappears:

```cpp
owner.reset();

observer.expired();  // true
observer.lock();     // Empty shared_ptr.
```

Common operations:

```cpp
observer.lock();       // Return shared_ptr if object still exists.
observer.expired();    // Whether all shared owners are gone.
observer.use_count();  // Number of shared owners.
observer.reset();      // Stop observing.
```

`weak_ptr` works with `shared_ptr`, not `unique_ptr`:

```cpp
std::weak_ptr<T> observer = unique_ptr_object;  // Error.
```

### Ownership Cycles

Two objects holding `shared_ptr` references to each other can prevent both from being destroyed:

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::shared_ptr<Node> prev;  // Potential ownership cycle.
};
```

Break the cycle using a non-owning `weak_ptr`:

```cpp
struct Node {
    std::shared_ptr<Node> next;  // Owning.
    std::weak_ptr<Node> prev;   // Non-owning.
};
```

### Doubly Linked List with `unique_ptr`

```cpp
template <typename T>
struct Node {
    T data;

    std::unique_ptr<Node<T>> next;  // Owns next node.
    Node<T>* prev = nullptr;        // Observes previous node.
};

template <typename T>
class DoublyLinkedList {
private:
    std::unique_ptr<Node<T>> head_;  // Owns first node.
    Node<T>* tail_ = nullptr;        // Observes last node.
};
```

- `head_` owns the first node.
- Each `next` owns the following node.
- `prev` and `tail_` are non-owning raw pointers.
- Every node has exactly one owner.

### Common Mistakes

```cpp
delete ptr.get();              // Wrong: smart pointer will delete it again.
auto copy = unique_ptr_object; // Wrong: unique_ptr cannot be copied.
*empty_pointer;                // Wrong: dereferencing nullptr.
*weak_pointer;                 // Wrong: weak_ptr must first be locked.
```

### Selection Rule

```cpp
T object;            // Prefer when dynamic allocation is unnecessary.
std::unique_ptr<T>   // Use when exactly one owner exists.
std::shared_ptr<T>   // Use when multiple owners are necessary.
std::weak_ptr<T>     // Use for non-owning access to a shared object.
T*                   // Use for non-owning observation when lifetime is guaranteed.
```