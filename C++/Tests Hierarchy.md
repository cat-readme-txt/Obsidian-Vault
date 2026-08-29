|Test type|What it tests|Scope|Typical speed|Typical example|
|---|---|---|---|---|
|**Unit test**|One small function/class/member|Very small|Fastest|Does `Square(5)` return `25`?|
|**Component test**|One larger module/component|Small-medium|Fast|Does an entire `Parser` class work correctly?|
|**Integration test**|Multiple components working together|Medium|Slower|Does parser + database layer work together?|
|**System test**|The whole application|Large|Slow|Does the entire program perform a user workflow correctly?|
|**Exploratory test**|Human investigation of behavior|Whole system|Manual|Try unusual inputs and workflows to discover bugs|
|**Manual/acceptance-type testing**|Human verifies final behavior|Whole system|Slowest|Does the finished application behave correctly from a user's perspective?|