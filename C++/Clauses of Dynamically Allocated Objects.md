An implicit contract:

| Clause | Result of Violation |
| --- | --- |
| 1. You will eventually return (`delete`) the memory that you borrow | Memory leak |
| 2. You will immediately stop using the memory that you've returned | Dangling pointer |
| 3. You will return (`delete`) memory that you did not borrow / You will not twice return (`delete`) memory that you've borrowed once | Runtime error |