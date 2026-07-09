# DAGKnight Wiki

A complete beginner-friendly guide to the DAGKnight consensus protocol. This wiki explains every algorithm independently, using pseudocode and simple Python implementations instead of production code.

## What You'll Learn

DAGKnight is a **parameterless** DAG-based consensus protocol that:
- Requires **no hardcoded latency parameter** — it adapts to real-time network conditions
- Tolerates up to **50% adversarial hashrate**
- Is **self-stabilizing** — recovers from past attacks
- Is **adaptive** — confirmation times depend on recent, not historical, latency

## Table of Contents

| Chapter | Topic | Prerequisites |
|---------|-------|---------------|
| [01 — DAG Basics](01-dag-basics.md) | Past, future, anticone, tips, virtual block, genesis | None |
| [02 — k-Clusters](02-k-clusters.md) | k-cluster definition, PHANTOM, why parameterization is limiting | Chapter 01 |
| [03 — Conflict Zones](03-conflict-zones.md) | Chain-parent, agreement, conflict genesis, gray blocks | Chapter 01 |
| [04 — Hierarchical Conflict Resolution](04-hierarchical-conflict-resolution.md) | The main ordering algorithm, iterative conflict resolution | Chapters 03, 06, 07 |
| [05 — K-Colouring](05-k-colouring.md) | The k-colouring algorithm, free search vs committed | Chapters 01, 03 |
| [06 — Rank](06-rank.md) | Rank calculation, representatives, the min-k search | Chapters 05 |
| [07 — Tie-Breaking](07-tie-breaking.md) | Algorithm 4, high-rank witnesses, recovery from rank inflation | Chapters 05, 06 |
| [08 — UMC Voting](08-umc-voting.md) | Uniform Majority Coverage, cascade voting, incremental approach | Chapters 05, 06 |

## How to Read This Wiki

1. **Follow the chapter order** — each chapter builds on previous concepts
2. **Study the pseudocode** — every algorithm is presented in clear pseudocode
3. **Run the Python examples** — each algorithm includes a simplified Python implementation you can execute
4. **Understand the "why"** — each section explains not just what the algorithm does, but why it's designed that way

## Quick Glossary

| Term | Definition |
|------|------------|
| **DAG** | Directed Acyclic Graph — a network of blocks where edges point backwards in time |
| **Past(B)** | All blocks provably created before B |
| **Future(B)** | All blocks provably created after B |
| **Anticone(B)** | All blocks concurrent with B (neither in past(B), future(B), nor B itself) |
| **Tips** | Blocks not referenced by any other block (the "frontier" of the DAG) |
| **Virtual Block** | A hypothetical block pointing to all tips |
| **k-cluster** | A subset where every block has at most k concurrent blocks within the subset |
| **Rank** | The minimal k for which a k-cluster uniformly covers the DAG |
| **Conflict Zone** | The region between a conflict genesis and the current tips |
| **UMC** | Uniform Majority Coverage — a cascading voting mechanism |
| **Gray Blocks** | Red blocks that agree with the winning subgroup |
