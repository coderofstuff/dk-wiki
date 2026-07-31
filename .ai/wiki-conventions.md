# DAGKnight Wiki Conventions and Corrections

## Terminology

### Non-Agreeing vs Attacker

- **Use "non-agreeing blocks/side"** when describing k-colouring, ranking, or UMC voting mechanics. DAGKnight has no concept of "good" or "bad" — it only determines whether blocks are part of a given coloring for a rank.
- **Use "attacker"** only when describing a specific attack scenario (e.g., the freeloading attack in `02-k-clusters.md`).
- Files using "non-agreeing": `06-rank.md`, `03-conflict-zones.md`, `04-hierarchical-conflict-resolution.md`.
- Files keeping "attacker": `02-k-clusters.md` (specific freeloading attack example).

### Non-Blue vs Red (Before Introducing Gray)

- When describing blocks that are excluded from the cluster, prefer **"non-blue"** over "red" until gray blocks are introduced.
- Saying "red blocks that agree are gray" is confusing. Instead: "non-blue blocks that agree are classified as gray".
- Red = non-blue AND disagrees. Gray = non-blue AND agrees.

## Diagram Conventions

### Base DAG Structure (Chapters 3+)

All DAG diagrams share the same base graph:
- `rankdir=TB` (Virtual at top, CG at bottom)
- `nodesep=0.8`, `ranksep=0.1`
- Left side cluster: B → D → F → T1
- Right side cluster: C → E → G → T2, H → T3
- Cross edges (gray dashed): F→C, T2→H, E→B
- X1 parallel to CG (B→X1 gray dashed), X2 parallel to CG (C→X2 gray dashed)
- Virtual block: dotted circle, points to T1, T2, T3 with gray dashed edges

### AGENTS.md Rank Direction

- Use `rankdir=TB` for all DAG diagrams.
- Edges point backward in time (newer → older), so TB puts future at top, past at bottom.
- `labelloc="t"` with `rankdir=TB` renders at the visual top.

### Color Conventions

- Blue (`lightskyblue`): Part of k-cluster
- Red/Salmon: Excluded from cluster AND disagrees with subgroup
- Gray (`lightgray`): Excluded from cluster BUT agrees with subgroup
- Orange doublecircle: Conflict genesis
- Virtual: dotted circle, white (or colored green for diamond diagram)
- Blue arrows (`penwidth=2.0`): Selected parent / chain-parent relationships
- Gray dashed arrows: Non-selected parent relationships

## Conflict Zone Formula

```
conflict_zone = Future(CG) ∩ { Past(T1) ∪ Past(T2) ∪ ... ∪ Past(TN) }
```

Not `future(CG) ∩ past(tips)` — must use union of pasts of individual tips.

## Chapter 3 Structure

1. The Chain-Parent
2. Conflict Zone
   - Conflict Genesis
   - The Diamond Shape
3. Agreement
4. Gray Blocks
5. The Virtual Block and Virtual Selected Parent
6. Summary

## Graphviz Practical Rules

1. **Root graph `labelloc` overrides cluster settings** — remove root `labelloc` to allow cluster-level control.
2. **Never guess Graphviz behavior** — verify with SVG coordinates before claiming layout is correct.
3. **Use `xlabel` for external labels** — more reliable than invisible-edge label nodes.
4. **Use `weight=10` on layout-critical edges** to maintain identical layouts across diagram variations.
5. **Clusters conflict with explicit rank constraints** — avoid combining both.
6. **Use `constraint=false` on invisible edges** to avoid affecting layout.

## Standard Styling Templates

### Node
```dot
node [shape=circle, style=filled, fontsize=10, width=0.6, height=0.6];
```

### Edge
```dot
edge [arrowsize=0.6, color=gray60];
```

### Cluster
```dot
subgraph cluster_name {
    label="Description";
    labelloc=t;
    style="dashed";
    color=blue;
}
```

### External Label
```dot
NODE [label="X", fillcolor=color, xlabel="description"];
```

### Invisible Layout Edge
```dot
A -> B [style=invis, constraint=false];
```

## Rendering Commands

```bash
# Render single diagram
dot -Tpng dot/<file>.dot -o png/<file>.png

# Verify layout via SVG coordinates
dot -Tsvg dot/<file>.dot | grep '<text' | grep 'font-size="10.00"'

# Render all diagrams
for f in dot/*.dot; do
  base=$(basename "$f" .dot)
  dot -Tpng "$f" -o "png/${base}.png"
done
```

## DAGKnight Protocol Corrections

1. **DAGKnight does NOT introduce chain-parent** — it uses the same concept as GhostDAG (also known as "selected parent").
2. **Anticone definition**: Blocks concurrent with B (not in past, future, or B itself) — NOT about shared past/future.
3. **DAG labels**: H1/H2/H3... for honest, A1/A2/A3... for attacker (in attack scenario diagrams).

## Image Replacements

- `06-gray-classification.png` replaced by `03-gray-blocks.png` in `06-rank.md`

## Files Using Base DAG Structure

- `03-conflict-zone.dot`
- `03-conflict-zone-diamond.dot`
- `03-agreement.dot`
- `03-agreement-next-iteration.dot`
- `03-gray-blocks.dot`
- `03-gray-larger.dot`
- `05-free-vs-committed.dot`
- `06-representative-problem.dot`
- `07-tie-breaking-example.dot`
- `08-umc-cascade.dot`
- `08-umc-minority.dot`
