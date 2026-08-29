```mermaid
graph TD

A["Node A<br/>value = 10<br/>parent = nullptr<br/>left = B<br/>right = C"]

B["Node B<br/>value = 5<br/>parent = A<br/>left = D<br/>right = E"]

C["Node C<br/>value = 15<br/>parent = A<br/>left = F<br/>right = G"]

D["Node D<br/>value = 2<br/>parent = B<br/>left = nullptr<br/>right = nullptr"]

E["Node E<br/>value = 7<br/>parent = B<br/>left = nullptr<br/>right = nullptr"]

F["Node F<br/>value = 12<br/>parent = C<br/>left = nullptr<br/>right = nullptr"]

G["Node G<br/>value = 20<br/>parent = C<br/>left = nullptr<br/>right = nullptr"]

A -->|left| B
B -->|parent| A

A -->|right| C
C -->|parent| A

B -->|left| D
D -->|parent| B

B -->|right| E
E -->|parent| B

C -->|left| F
F -->|parent| C

C -->|right| G
G -->|parent| C
```

## Binary Search Tree Rules

Let `x` be a node in the BST:
- if `y` is a node in the left subtree of `x`, then `y.key` $<$ `x.key`.
- if `y`. is a node in the right subtree of `x`, then `y.key` $\ge$ `x.key`.

## Successor

The `successor` of a node is the node that will come right after the specified node in an [[Tree Traversal Orders|Inorder Traversal]].

#### How to find?
```mermaid
graph LR
A{{"right child node exists?"}} -->|true| B["take a right step then as far left as you can"]
A -->|false| C{{"is the current node left or right child of its parent?"}}
C -->|left| D["its parent is the successor"]
C -->|right| E["travel up until you find an ancestor that is a left child of its own parent"]
```

Example:
```mermaid
graph TD
    N20["20"] --> N10["10"]
    N20 --> N30["30"]

    N10 --> N5["5"]
    N10 --> N15["15"]

    N30 --> N25["25"]
    N30 --> N35["35"]

    N15 --> N12["12"]
```

Sample node: `10`
Its successor: `12`