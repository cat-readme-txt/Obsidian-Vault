## Navigator

- [[#ASan (AddressSanitizer)]]
- [[#UBSan (UndefinedBehaviorSanitizer)]]
- [[#ASan + UBSan]]
- [[#Valgrind]]
- [[#TSan (ThreadSanitizer)]]
- [[#LSan (LeakSanitizer)]]
- [[#DFSan (DataFlowSanitizer)]]

---

## Tool Comparison

| Tool | Primary Target | Example |
|---|---|---|
| **ASan** | Invalid memory accesses and lifetime errors | Buffer overflow, use-after-free |
| **UBSan** | Selected undefined behavior | Signed overflow, null dereference |
| **Valgrind Memcheck** | Memory errors, uninitialized values, and leaks | Invalid read, uninitialized-value use |
| **TSan** | Data races | Unsynchronized concurrent write |
| **DFSan** | Data-flow / taint tracking | Determine whether output depends on labeled input |
| **LSan** | Memory leaks | Unreachable dynamically allocated object |

## Tool Selection

```text
Out-of-bounds / use-after-free / invalid memory access
→ ASan

Signed overflow / null dereference / other selected undefined behavior
→ UBSan

Memory errors + detailed uninitialized-value analysis without sanitizer compilation
→ Valgrind Memcheck

Concurrent unsynchronized memory access
→ TSan

Memory leaks only
→ LSan

Taint / information-flow tracking
→ DFSan

General C++ runtime checking during development
→ ASan + UBSan
```

## Diagnostic Terminology

Use precise terminology when describing these tools:

```text
ASan
→ memory-error detector

UBSan
→ undefined-behavior detector

Valgrind
→ dynamic program-analysis framework

Memcheck
→ Valgrind memory-error detector

TSan
→ data-race detector

LSan
→ memory-leak detector

DFSan
→ data-flow tracking framework
```

Avoid describing all of them simply as **debuggers**. A debugger such as GDB or LLDB primarily allows inspection and control of program execution, whereas sanitizers instrument execution to detect particular classes of erroneous operations.

---

## ASan (AddressSanitizer)

- Runtime memory-error detector based on compiler instrumentation
- Detects:
  - Out-of-bounds memory accesses:
    - Heap buffer overflow
    - Stack buffer overflow
    - Global buffer overflow
  - Use-after-free
  - Stack use-after-return
  - Stack use-after-scope
  - Double-free
  - Invalid `free` / `delete`
  - Memory leaks through LeakSanitizer (LSan) on supported platforms
- Typical runtime overhead: approximately **2×**
- Usually terminates the program after the first detected error

### Common Compile Command

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=address \
  -fstandalone-debug \
  -fno-omit-frame-pointer \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

### Compile Flags

- `-std=c++20`
  - Compile according to the C++20 standard

- `-g`
  - Emit debugging information for source file names, line numbers, variables, and stack traces

- `-O1`
  - Enable level-1 optimization
  - Provides moderate optimization while generally preserving useful debugging information

- `-fsanitize=address`
  - Enable AddressSanitizer instrumentation and link the ASan runtime

- `-fstandalone-debug`
  - Emit more complete debugging type information without relying as heavily on information from other compilation units

- `-fno-omit-frame-pointer`
  - Preserve frame pointers to improve stack-trace quality

- `-Wall`
  - Enable a broad set of common compiler warnings

- `-Wextra`
  - Enable additional warnings not included in `-Wall`

- `-Werror`
  - Promote enabled warnings to compilation errors

- `-pedantic`
  - Diagnose use of compiler extensions that are not part of the selected C++ standard

### Example: Heap Buffer Overflow

```cpp
int* arr = new int[3];
arr[5] = 10;
```

Possible ASan report:

```text
ERROR: AddressSanitizer: heap-buffer-overflow
WRITE of size 4
```

### Example: Heap Use-After-Free

```cpp
int* ptr = new int{5};
delete ptr;

std::cout << *ptr;
```

Possible ASan report:

```text
ERROR: AddressSanitizer: heap-use-after-free
READ of size 4
```

### Out-of-Bounds Detection

ASan detects whether an **actual memory access** occurs outside a valid instrumented memory region.

```cpp
int* arr = new int[5];
arr[7] = 4;
```

The allocation contains five `int` objects, but the program attempts to access memory beyond that allocation.

Possible report:

```text
heap-buffer-overflow
```

For a local array:

```cpp
int arr[5]{};
arr[7] = 4;
```

ASan may instead report:

```text
stack-buffer-overflow
```

---

## UBSan (UndefinedBehaviorSanitizer)

- Runtime detector for selected forms of **undefined behavior**
- Uses compiler instrumentation to insert checks around operations that can violate C/C++ language rules
- `-fsanitize=undefined` enables a group of UBSan checks; it does **not** detect every possible form of undefined behavior

### Detects

- Null-pointer dereference
- Misaligned pointer/reference access
- Signed integer overflow
- Integer division by zero
- Invalid bit shifts
- Certain invalid type conversions
- Array subscripts outside statically known bounds
- Certain operations involving invalid object types or values

### Common Compile Command

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=undefined \
  -fstandalone-debug \
  -fno-omit-frame-pointer \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

### Compile Flags

- `-std=c++20`
  - Compile according to the C++20 standard

- `-g`
  - Emit debugging information for source locations and stack traces

- `-O1`
  - Enable level-1 optimization while generally preserving useful debugging information

- `-fsanitize=undefined`
  - Enable the standard group of UndefinedBehaviorSanitizer checks

- `-fstandalone-debug`
  - Emit more complete standalone debugging type information

- `-fno-omit-frame-pointer`
  - Preserve frame pointers to improve stack-trace quality

- `-Wall`
  - Enable a broad set of common compiler warnings

- `-Wextra`
  - Enable additional warnings not included in `-Wall`

- `-Werror`
  - Promote enabled warnings to compilation errors

- `-pedantic`
  - Diagnose use of non-standard compiler extensions

### Example: Null-Pointer Dereference

```cpp
int* ptr = nullptr;
std::cout << *ptr;
```

Possible UBSan report:

```text
runtime error: load of null pointer of type 'int'
```

### Example: Signed Integer Overflow

```cpp
int x = INT_MAX;
x += 1;
```

Possible UBSan report:

```text
runtime error: signed integer overflow
```

### Array-Bounds Detection

For:

```cpp
int arr[5]{};
arr[7] = 10;
```

the compiler knows that `arr` has type:

```cpp
int[5]
```

and therefore knows that the only valid indices are `0` through `4`.

UBSan can report:

```text
runtime error: index 7 out of bounds for type 'int[5]'
```

### UBSan Bounds Check vs. ASan Bounds Check

| UBSan | ASan |
|---|---|
| Detects violation of language/type-level bounds | Detects an invalid memory access |
| Can use statically known array bounds | Tracks validity of instrumented memory regions |
| Reports the invalid index/type | Reports the invalid memory access |
| Example: `index 7 out of bounds for type 'int[5]'` | Example: `stack-buffer-overflow` |
| Also checks many non-memory forms of undefined behavior | Primarily targets memory-safety violations |

Conceptually:

```text
UBSan:
"Is this operation valid according to the C++ language/type rules?"

ASan:
"Is this memory location valid to access right now?"
```

#### Example

```cpp
int arr[5]{};
arr[7] = 10;
```

UBSan can reason:

```text
arr has type int[5]
index 7 is invalid
```

ASan can reason:

```text
the write reached memory outside arr's valid storage region
```

The same bug can therefore be detected by both sanitizers for different reasons.

---

## ASan + UBSan

ASan and UBSan can usually be enabled together:

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=address,undefined \
  -fstandalone-debug \
  -fno-omit-frame-pointer \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

Use:

```text
ASan
→ memory-access and lifetime errors

UBSan
→ selected language-level undefined behavior

ASan + UBSan
→ both categories
```

---

## Valgrind

- Dynamic program-analysis framework containing multiple analysis tools
- Operates on the compiled executable rather than requiring compiler sanitizer instrumentation
- **Memcheck** is Valgrind's default and most commonly used memory-analysis tool

### Memcheck Detects

- Invalid memory reads
- Invalid memory writes
- Use of certain uninitialized values
- Invalid memory deallocation
- Double-free
- Memory leaks
- Mismatched allocation/deallocation operations

### Performance

- Significantly slower than native execution
- Memcheck commonly introduces substantially more runtime overhead than ASan
- Does not require recompilation with sanitizer instrumentation, although compiling with debugging information produces substantially better diagnostics

### Recommended Compile Command

```bash
clang++ -std=c++20 -g -O0 \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

### Compile Flags

- `-std=c++20`
  - Compile according to the C++20 standard

- `-g`
  - Emit debugging information so Valgrind can report source file names and line numbers

- `-O0`
  - Disable optimization
  - Keeps generated machine instructions closely associated with the original source code, improving diagnostic clarity

- `-Wall`
  - Enable a broad set of common compiler warnings

- `-Wextra`
  - Enable additional warnings not included in `-Wall`

- `-Werror`
  - Promote enabled warnings to compilation errors

- `-pedantic`
  - Diagnose use of non-standard compiler extensions

### Running Memcheck

Memcheck is Valgrind's default tool:

```bash
valgrind ./<program_name>
```

Explicitly specifying Memcheck:

```bash
valgrind --tool=memcheck ./<program_name>
```

Enable detailed leak checking:

```bash
valgrind --leak-check=full ./<program_name>
```

A commonly useful form is:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./<program_name>
```

### Relevant Runtime Options

- `--tool=memcheck`
  - Select Memcheck
  - Optional because Memcheck is the default Valgrind tool

- `--leak-check=full`
  - Perform detailed leak analysis and show where leaked allocations originated

- `--show-leak-kinds=all`
  - Display all categories of detected memory leaks

### ASan vs. Valgrind Memcheck

| ASan | Valgrind Memcheck |
|---|---|
| Requires compile-time instrumentation | Analyzes an already compiled executable |
| Lower runtime overhead | Higher runtime overhead |
| Detects buffer overflows and lifetime errors | Detects invalid accesses, leaks, and uninitialized-value usage |
| Requires compilation with `-fsanitize=address` | Does not require sanitizer-specific compilation |
| Integrated with compiler/runtime | External dynamic-analysis framework |

---

## TSan (ThreadSanitizer)

- Runtime detector for **data races**
- Instruments memory accesses and synchronization operations in multithreaded programs
- Detects conflicting accesses to shared memory that are not properly synchronized

### Data Race

A data race occurs when:

1. Two or more threads access the same memory location concurrently
2. At least one access is a write
3. The accesses are not properly synchronized

Example:

```cpp
int counter = 0;

void Increment() {
    ++counter;
}
```

If multiple threads execute `Increment()` concurrently without synchronization, accesses to `counter` can form a data race.

### Common Compile Command

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=thread \
  -fstandalone-debug \
  -fno-omit-frame-pointer \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

### Relevant Sanitizer Flag

- `-fsanitize=thread`
  - Enable ThreadSanitizer instrumentation and link the TSan runtime

### Important Limitation

TSan detects **data races**, not every possible multithreading error.

For example, a deadlock can occur without necessarily producing a data race.

---

## DFSan (DataFlowSanitizer)

- Runtime **data-flow tracking** framework
- Tracks how labeled data propagates through program operations
- Unlike ASan, UBSan, and TSan, DFSan does not primarily detect a predefined category of programming error
- Used to implement application-specific forms of **taint tracking**

### Data-Flow Model

A value can be assigned a label:

```text
input
  ↓
label A
  ↓
computation
  ↓
derived values retain information about label A
```

The program can later determine whether a result was influenced by particular labeled input.

### Applications

- Taint tracking
- Tracking propagation of untrusted input
- Information-flow analysis
- Determining which inputs influence a computed result

### Common Compile Command

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=dataflow \
  -fno-omit-frame-pointer \
  <source_file> -o <program_name>
```

### Relevant Sanitizer Flag

- `-fsanitize=dataflow`
  - Enable DataFlowSanitizer instrumentation

### DFSan Interface

Programs using DFSan commonly interact with:

```cpp
#include <sanitizer/dfsan_interface.h>
```

The program explicitly assigns and queries data-flow labels.

Compiling with:

```bash
-fsanitize=dataflow
```

instruments propagation, but the programmer must define what data is significant through the DFSan interface.

---

## LSan (LeakSanitizer)

- Runtime detector specifically for **memory leaks**
- Identifies dynamically allocated memory that remains allocated but is no longer reachable when leak checking occurs
- Can operate:
  - As part of ASan on supported platforms
  - As a standalone sanitizer
- Most leak-analysis work occurs near program termination rather than on every memory access

### Example

```cpp
int* ptr = new int[100];
ptr = nullptr;
```

After:

```cpp
ptr = nullptr;
```

the allocation still exists:

```text
free-store allocation
        ↑
no remaining pointer reaches it
```

The allocation is therefore leaked.

### Standalone Compile Command

```bash
clang++ -std=c++20 -g -O1 \
  -fsanitize=leak \
  -fno-omit-frame-pointer \
  -Wall -Wextra -Werror -pedantic \
  <source_file> -o <program_name>
```

### Relevant Sanitizer Flag

- `-fsanitize=leak`
  - Enable standalone LeakSanitizer and link the LSan runtime

### LSan with ASan

On platforms where ASan leak detection is supported:

```bash
-fsanitize=address
```

can provide:

```text
AddressSanitizer
├── invalid memory-access detection
└── LeakSanitizer-based leak detection
```

Use standalone:

```bash
-fsanitize=leak
```

when leak detection is desired without full AddressSanitizer instrumentation.