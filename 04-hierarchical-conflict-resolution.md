# 04 — Hierarchical Conflict Resolution

## The Main Algorithm

DAGKnight's main ordering algorithm (Algorithm 2 in the paper) is called **Order-DAG**. It takes a DAG as input and returns:

1. The selected tip (which becomes the chain-parent)
2. A deterministic ordering of all blocks in the DAG

### The Big Picture

```
Input:  DAG G
Output: (selected_tip, ordering_of_G)

function Order-DAG(G):
    # BASE CASE: Genesis block
    if G contains only genesis:
        return (genesis, [genesis])

    # RECURSIVE STEP: Order each tip's past
    for each tip B in G:
        (chain_parent_B, ordering_B) = Order-DAG(past(B))

    # CONFLICT RESOLUTION: Select among competing tips
    candidates = all tips of G

    while |candidates| > 1:
        # Find the conflict point
        conflict_genesis = latest common chain ancestor of all candidates

        # Split into agreement subgroups
        subgroups = partition candidates by their next chain ancestor above conflict_genesis

        # Evaluate each subgroup's rank
        for each subgroup:
            subgroup.rank = Calculate-Rank(subgroup, future(conflict_genesis))

        # Find the minimum rank
        best_rank = min(subgroup.rank for all subgroups)

        # Eliminate subgroups with higher rank
        tied = subgroups where subgroup.rank == best_rank

        # Break ties if needed
        if |tied| > 1:
            winners = Tie-Breaking(future(conflict_genesis), tied)
        else:
            winners = tied[0]

        # Eliminate losers from candidates
        for each subgroup not in winners:
            candidates -= subgroup.blocks

        candidates = blocks from winning subgroup

    # SELECTED TIP: The sole survivor
    selected_tip = the single remaining candidate

    # BUILD ORDERING: Chain order + anticone blocks
    return (selected_tip, ordering_of_selected_tip + [selected_tip] + anticone_of_selected_tip)
```

## Step by Step Breakdown

### Step 1: Recursive Ordering

For each tip B, recursively call Order-DAG on B's past. This establishes each tip's:
- Chain-parent (which parent it selected from its past)
- Ordering (deterministic block order within its past)

This recursion bottoms out at genesis, which has no parents and returns immediately.

### Step 2: Find the Conflict

The **latest common chain ancestor (LCCA)** of all candidates is the conflict genesis. This is the most recent block where all candidates still "agreed."

The first iteration of hierarchical conflict resolution looks exactly like the agreement diagram from Chapter 03:

![Iteration 1: Three tips, two subgroups](png/03-agreement.png)

Here we have 3 tips: **T1** (left side) and **T2, T3** (right side). The conflict genesis is **CG**. The two subgroups are:
- **Left subgroup** (blue): {T1, F, D, B} — chain path T1 → F → D → B → CG
- **Right subgroup** (yellow/orange): {T2, T3, G, H, E, C} — chain paths T2 → G → E → C → CG and T3 → H → E → C → CG

DAGKnight calculates the rank of each subgroup. Suppose the left subgroup has rank 2 and the right subgroup has rank 5. The left subgroup wins (lower rank), and the right subgroup is eliminated.

### How Subgroups Form

Subgroups are formed by walking each tip's chain-parent path backward toward the conflict genesis, and grouping tips that share the same block just above the conflict genesis:

```python
def next_chain_ancestor(block, conflict_genesis):
    """Walk the chain from block toward conflict_genesis, return the block just above CG."""
    current = block
    while current != conflict_genesis:
        parent = chain_parent(current)
        if chain_parent(parent) == conflict_genesis:
            return parent
        current = parent
```

Tips with the same next chain ancestor form a subgroup. In the diagram above:
- T1's next chain ancestor above CG is B → **Left subgroup**
- T2's and T3's next chain ancestor above CG is C → **Right subgroup**

Not every chain extension above CG forms a subgroup. Only a chain extension that is **reachable from at least one tip** via chain ancestry counts. If CG has a child X that no tip chains through, X does not form a group — the perception (agreement) must propagate from the tips downward. The number of groups is bounded by the number of CG's children, but could be fewer if some branches are not reachable from any tip.

### Step 3: Rank Each Subgroup

For each subgroup, calculate its rank using the algorithm from Chapter 06:

```
rank_i = Calculate-Rank(subgroup_i, future(conflict_genesis))
```

### Step 4: Eliminate Losers

Subgroups with higher rank are eliminated. Only the subgroup(s) with the minimum rank survive.

### Step 5: Tie-Breaking (if needed)

If multiple subgroups share the minimum rank, the tie-breaking algorithm (Chapter 07) selects the winner.

### Step 6: Iterate — Narrow to the Winning Subgroup

If the winning subgroup still has multiple tips, go back to Step 2 with the winning subgroup's blocks as the new candidate set. This is the **hierarchical** aspect — conflicts are resolved from oldest to newest.

In our example, the left subgroup has only one tip (T1), so the algorithm terminates. But imagine the **right subgroup** had won instead. With two tips (T2 and T3), the algorithm continues:

![Iteration 2: Narrowed conflict zone](png/03-agreement-next-iteration.png)

Notice what changed:
- The old conflict zone (CG → tips) is **grayed out** — already resolved
- A **new conflict genesis** emerges: **E** (the latest common chain ancestor of T2 and T3)
- The conflict zone has narrowed to the region between E and {T2, T3}
- Two new subgroups form within the old right side:
  - **Left side** (blue): T2 → G → E
  - **Right side** (yellow/orange): T3 → H → E

This narrowing process repeats until a single tip remains. That tip becomes the **selected tip**.

## Why Hierarchical?

The while loop creates a **conflict hierarchy** — conflicts are resolved from oldest to newest, with each level narrowing the candidate set.

### Why Explicit Conflict Resolution?

DAGKnight could not simply "pick the tip with the most blue work" like some protocols do. Instead, it must work with the topology it can see. Tips disagree on their view of the DAG — each tip represents a different perception of what the network looks like. To compare competing perceptions, DAGKnight must first find where they last agreed (the conflict genesis), then evaluate which side built a better-connected network above that point.

The side with the lower rank is the side that achieved better connectivity within the conflict zone. Lower rank means the network was more tightly connected during that period — which is exactly what you want to reward.

### Why Not Evaluate from Genesis?

The core of DAGKnight is being **adaptive to actual latency observed in the DAG**. By evaluating from the conflict genesis upward — rather than from genesis to tips — DAGKnight judges only the recent latency conditions that matter for the current conflict. It does not drag very old history into the decision.

### A General Structure

Hierarchical conflict resolution is a general framework, not specific to DAGKnight:
- **Nakamoto consensus** (longest chain) uses it implicitly when choosing among competing forks
- **Phantom** uses it with hash-power-based tie-breaking
- **HashGuard** uses it with hash-based tip selection
- **DAGKnight** uses it with rank-based evaluation

What makes DAGKnight unique is not the hierarchy itself, but what it uses to judge subgroups: the rank, which measures the actual connectivity observed in the DAG.

### Two Benefits of Hierarchy

1. **Early-to-recent ordering**: Oldest conflicts are resolved first, providing stability for settled parts of the DAG.

2. **Adaptiveness**: Each level of conflict can have a different rank. If the network was slow in the past (high rank) but is fast now (low rank), the hierarchy naturally adapts — because each level evaluates only the topology above its own conflict genesis, not the entire history.

## The Return Value

### Selected Tip

The final survivor of the while loop. This block becomes the **chain-parent** of the next block to be mined.

### Ordering

The ordering is built by concatenation:

```
ordering = order_of_selected_tip + [selected_tip] + anticone_blocks
```

Where:
- `order_of_selected_tip` is the recursive ordering from the selected tip's past
- `[selected_tip]` is the selected tip itself
- `anticone_blocks` are blocks in the selected tip's anticone, ordered by hash-based topological sort

This ensures that:
1. Blocks in the selected tip's past come first
2. The selected tip comes next
3. Anticone blocks come last, in a deterministic order

## Decoupling Ordering from Colouring

### The Problem

If we directly used the k-cluster for ordering, a sudden increase in network latency would change the k value retroactively, causing all past orderings to change. This is called **ordering instability**.

### DAGKnight's Solution

DAGKnight **decouples** cluster-selection from chain-ordering:

- **Cluster-selection** (k-colouring) determines which blocks are blue/red
- **Chain-ordering** (chain-parent selection) determines the canonical order
- A block's chain-parent is the block that **minimizes its rank**

This means:
- Different parts of the chain can have different k values
- Past parts of the chain keep their k value (stable)
- Only recent parts adapt to new conditions

### Example

```
Chain: A → B → C → D → E → F
         k=2    k=2    k=5    k=5    k=3

At C, an attack was detected (k went from 2 to 5).
At E, the attack ended (k dropped to 3).
Blocks A and B keep k=2 (stable ordering).
Blocks C and D have k=5 (attack period).
Block F has k=3 (recovery period).
```

## Gray Blocks in the Hierarchy

During the rank calculation for each subgroup, gray blocks are identified:

```
gray_block = red_block whose next_chain_ancestor matches the winning subgroup's
```

Gray blocks:
- Are counted as **neutral** in UMC voting
- Don't contribute to the winning or losing side
- Help prevent unfair disadvantage to subgroups with red blocks that happen to agree with them

## Complexity

| Aspect | Complexity |
|--------|------------|
| While loop iterations | O(|tips|) — at least one subgroup eliminated per iteration |
| Rank calculation per subgroup | O(K_search × zone_size × K) |
| Total (with rank search) | O(|tips| × log(max_k) × zone_size × K) |
| Overall recursion | Polynomial in |G| |

## Summary

| Concept | Definition |
|---------|------------|
| **Order-DAG** | Main algorithm: recursively orders DAG and resolves conflicts |
| **LCCA** | Latest common chain ancestor — the conflict point |
| **Agreement subgroups** | Tips partitioned by shared chain extension above conflict |
| **Hierarchy** | Conflicts resolved from oldest to newest |
| **Decoupling** | Cluster-selection separate from chain-ordering for stability |
| **Selected tip** | Final survivor, becomes chain-parent |

The hierarchical conflict resolution algorithm is the orchestrator of DAGKnight. It uses all the components we've studied (k-colouring, UMC voting, rank, tie-breaking) to produce a deterministic, stable, and adaptive ordering of the entire DAG.
