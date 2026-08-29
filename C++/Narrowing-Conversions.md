> [!note] Definition
> A narrowing conversion changes a value to a data type that might not have enough space or precision to hold all values of the original type. This can cause data loss, such as cutting off decimal numbers or changing a large integer into a smaller one.

Disallowed by [[Initialization and Assignment Types#^72ae40|Direct-List Initialization]] and other list-typed initializations.

```cpp
int main()
{
    // An integer can only hold non-fractional values.
    // Initializing an int with fractional value 4.5 requires the compiler to convert 4.5 to a value an int can hold.
    // Such a conversion is a narrowing conversion, since the fractional part of the value will be lost.

    int w1 { 4.5 }; // compile error: list-init does not allow narrowing conversion

    int w2 = 4.5;   // compiles: w2 copy-initialized to value 4
    int w3 (4.5);   // compiles: w3 direct-initialized to value 4

    return 0;
}
```

Note that this restriction on narrowing conversions only applies to the list-initialization, not to any subsequent assignments to the variable:

```cpp
int main()
{
    int w1 { 4.5 }; // compile error: list-init does not allow narrowing conversion of 4.5 to 4

    w1 = 4.5;       // okay: copy-assignment allows narrowing conversion of 4.5 to 4

    return 0;
}
```