## Overview

|Level|What must your tests cover?|How to identify it|
|---|---|---|
|**S0**|Every method/function is called at least once|“Did every function run?”|
|**S1**|Every method/function is called from every place that calls it|“Did every call site execute?”|
|**C0**|Every executable statement runs at least once|“Did every line/statement execute?”|
|**C1**|Every branch outcome occurs|“Did every `if` go both true and false?”|
|**C2**|Every possible execution path occurs|“Did we test every possible route through the control-flow graph?”|

## S0

```cpp
int Add(int a, int b) {
    return a + b;
}

int Multiply(int a, int b) {
    return a * b;
}
```

S0 Coverage Test:
```cpp
Add(2, 3);
Multiply(2, 3);
```

## S1
```cpp
void Foo() {
    Helper();   // call site 1
}

void Bar() {
    Helper();   // call site 2
}
```

S1 Coverage Test:
```cpp
Foo();
Bar();
```

S0 would only require `Helper()` be called somewhere, instead of requiring both call sites to be called.

S0 Coverage Test:
```cpp
Foo();
```

## C0
```cpp
int Abs(int x) {
    if (x < 0) {
        return -x;
    }

    return x;
}
```

C0 Coverage Test:
```cpp
Abs(5);   // return x runs
Abs(-5);  // if statement & return -x runs
```
requires all statements be executed.

## C1
```cpp
int Pos(int x) {
    if (x > 0) {
        std::cout << "x is positive"
    }

    std::cout << "done";
}
```
the `if` statement must be evaluated as `true` and `false` for all branches to be tested.

C1 Coverage Test:
```cpp
Pos(5);   // executes true if and 2 print statements
Pos(-5);  // executes false if and 1 print statement
```

C0 Coverage Test:
```cpp
Pos(5);   // executes if and 2 print statements
```

## C2
```cpp
if (a) {
    // A
}

if (b) {
    // B
}
```
All branches need to be executed for C2 coverage.

C2 Coverage Test:
```cpp
a=true,  b=true
a=true,  b=false
a=false, b=true
a=false, b=false
```

C1 Coverage Test:
```cpp
a=true, b=true
a=false, b=false
```