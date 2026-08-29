A function call usually gets a stack frame containing some combination of:

- local variables
- parameters
- saved registers
- return address
- temporary values
- bookkeeping information

Conceptually:

```
higher addresses
┌─────────────────────┐
│ main() frame        │
├─────────────────────┤
│ foo() frame         │
├─────────────────────┤
│ bar() frame         │
└─────────────────────┘
lower addresses
```

If:

```
main()
    → foo()
        → bar()
```

then `bar()`'s frame is typically on top.

When `bar()` returns, its frame is removed first.

That's why it is called a **stack**:

```
LIFO
Last In, First Out
```