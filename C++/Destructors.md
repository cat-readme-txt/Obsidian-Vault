```cpp
class ResourceManager {
private:
    int* data;

public:
    // Constructor allocates memory
    ResourceManager() { 
        data = new int[100]; 
    }

    // Destructor frees memory automatically
    ~ResourceManager() { 
        delete[] data; 
    }
};
```

The `~ResourceManager()` is the destructor, it can be manually called like any other functions.

## When Do Destructors Run?

C++ manages destructor invocations implicitly based on how an object is allocated:

- **Stack Objects**: Invoked automatically when the object goes out of scope (e.g., reaching a closing brace `}`).
- **Heap Objects**: Invoked only when you explicitly call `delete` on a pointer pointing to the object.
- **Static / Global Objects**: Invoked automatically when the program finishes execution.

## Destruction Order

When multiple sub-components are destroyed, C++ tears down objects in the **exact reverse order of their construction**:

- The body of the class destructor executes.
- Member variables are destroyed in reverse declaration order.
- Base class destructors execute.

## Virtual Destructors (Crucial for Polymorphism)

If a class is designed to be inherited from (a base class), you **must** declare its destructor as `virtual`.

Without a virtual destructor, deleting a derived class instance through a base-class pointer triggers **undefined behavior**, causing the derived class cleanup code to skip entirely and leak memory.

```cpp
class Base {
public:
    virtual ~Base() {} // Always make base class destructors virtual
};

class Derived : public Base {
    int* heavyBuffer = new int[5000];
public:
    ~Derived() override { delete[] heavyBuffer; }
};

```