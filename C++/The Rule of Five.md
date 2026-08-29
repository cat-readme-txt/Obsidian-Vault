> If your class needs to manually define **any one** of these five special member functions, it probably needs to define **all five**:

```cpp
~ClassName();                              				// destructor
ClassName(const ClassName<T>& other);         			// copy constructor
ClassName<T>& operator=(const ClassName<T>& other); 	// copy-assignment operator
ClassName(ClassName<T>&& rhs);							// move constructor
ClassName<T>& operator=(ClassName<T>&& rhs);			// move assignment operator
```

Defining either a destructor, copy constructor, or a copy-assignment operator, the compiler will automatically generate a default for the other two, which will likely not have the intended behavior.

Defining either the move constructor or the move assignment operator will not have the compiler generate the other, they are not independent.

Move operators are automatically generated for your class, only if:

- No copy operations are declared
- No move operations are declared
- No destructor is declared in that class