```cpp
class A {
	int a, b{0}; // default member initialization
public:
	A(): a{0} {
		// Constructor body
	}
	// b is initialized by b{b}
	// the default member initializer b{0} is ignored
	A(int a, int b): a{a}, b{b} {
		// Constructor body
	}
};
```

The syntax after `:` is the syntax for the member list initialization.

A **constructor member initializer** overrides that member's **default member initializer**. If the constructor's member initializer list explicitly initializes a data member, the default member initializer is not used. Otherwise, the default member initializer is used.