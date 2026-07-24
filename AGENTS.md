# Wiki Editing Guidelines

This directory contains the DAGKnight wiki — a beginner-friendly, self-contained guide to the protocol.

## Rules

### Content

1. **No repository source code**. Use pseudocode or simplified Python implementations instead. The wiki must be understandable without access to the codebase.
2. **All algorithms explained independently**. Each algorithm stands on its own — a reader should be able to understand it without reading the paper.
3. **Explain the "why"**. Every algorithm section must include motivation (why it exists, what problem it solves) and design rationale (why it's designed this way).
4. **Progressive disclosure**. Chapters build on previous concepts. Follow the chapter order (01 → 08).

### Graphics

5. **Use DOT, never ASCII art**. All diagrams must be rendered from DOT files.
6. **Source files in `dot/`**. Save all `.dot` files in `wiki/dot/` with descriptive names prefixed by chapter number (e.g., `06-gray-classification.dot`).
7. **Rendered files in `png/`**. Rendered PNGs go in `wiki/png/` with matching names.
8. **Reference PNGs in markdown**. Use `![alt text](png/filename.png)` — never embed DOT source in markdown.

### Rendering

9. **Regenerate PNGs on DOT changes**. When editing a `.dot` file, regenerate the PNG:
   ```bash
   dot -Tpng wiki/dot/XX-filename.dot -o wiki/png/XX-filename.png
   ```
10. **Regenerate all PNGs in batch**:
    ```bash
    cd wiki/dot && for f in *.dot; do dot -Tpng "$f" -o "../png/${f%.dot}.png"; done
    ```
11. **DOT files must be valid**. Test each `.dot` file before committing:
    ```bash
    dot -Tpng wiki/dot/XX-filename.dot -o /dev/null
    ```
12. **Always update the corresponding wiki text**. When a diagram is changed (layout, labels, added/removed elements, etc.), the markdown chapter that references it MUST be updated to match. The prose describing the diagram should accurately reflect what the reader sees. If a label changes in the DOT, the markdown bullet points referencing that label must change too. Never leave a diagram out of sync with its surrounding text.
13. **Visually verify every rendered image**. After rendering a PNG, open it and confirm it matches the text describing it. Check layout direction (past should be at the bottom, future at the top), label placement, and that the prose accurately describes what the reader sees. Use `rankdir=TB` for all DAG diagrams (edges point backwards in time from newer to older, so TB puts newer/future at top and older/past at bottom). Note: `labelloc` is relative to the rank direction — `labelloc="t"` means "top of the rank direction", so with `rankdir=TB`, `t` renders at the visual top.

### Knowledge

14. **Never guess**. If you are unsure about how a tool, attribute, or behavior works, read the official documentation to verify before answering or making changes. Do not rely on memory or intuition for technical details. Incorrect guesses waste the user's time and create back-and-forth.

### Structure

15. **No inline DOT code blocks**. DOT source belongs only in `dot/`. Markdown chapters reference `png/` files.
16. **Keep DOT files minimal**. Use compact node labels, use `rankdir=TB` for DAG diagrams, and include legends when colors carry meaning.

### DOT Style

17. **Node shapes**: circles for blocks, doublecircles for special blocks (genesis, conflict genesis), boxes for virtual blocks
18. **Color convention**: blue (`lightskyblue`) for cluster members, red/salmon for excluded blocks, gray (`lightgray`) for gray blocks, orange for conflict genesis
19. **Edge direction**: arrows point backwards in time (from newer to older), consistent with DAG convention
