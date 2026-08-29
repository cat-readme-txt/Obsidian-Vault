## Micro Patterns

Micro patterns describe **how a variable's value changes over time**, not its data type.

- **Most-wanted holder:** Stores the best value encountered so far, such as the maximum, minimum, or closest match.
- **Follower:** Stores the previous value of another variable.
- **One-way flag:** Starts in one state and, once changed, never changes back.

### Example

```cpp
std::vector<int> values{8, 12, 10, 15};

int current = values[0];
int best = current;             // Most-wanted holder
int previous = current;         // Follower
bool has_decreased = false;     // One-way flag

for (size_t i = 1; i < values.size(); ++i) {
    current = values[i];

    if (current > best)
        best = current;

    if (current < previous)
        has_decreased = true;

    previous = current;
}
```

After the loop:

- `best == 15`
- `previous == 15`, the final value processed
- `has_decreased == true` because `12 → 10`


Definitions follow the standard [roles-of-variables framework](https://saja.kapsi.fi/var_roles/role_intro.html).