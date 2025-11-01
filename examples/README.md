# Example Data

This directory contains a minimal example demonstrating the input format for Tidy Tree.

## Files

- `sequences.fasta`: 9 aligned DNA sequences (seq001-seq009)
- `founders.fasta`: 3 lineage founder sequences (LineageA, LineageB, LineageC)
- `guide_tree.nwk`: Guide tree showing LineageA as sister to (LineageB + LineageC)
- `assignments.tsv`: Mapping of sequences to lineages with header row (3 sequences per lineage)

## Lineage Structure

```
         ┌─ LineageA (seq001, seq002, seq003)
    ─────┤
         │     ┌─ LineageB (seq004, seq005, seq006)
         └─────┤
               └─ LineageC (seq007, seq008, seq009)
```

## Running the Example

From the project root directory:

```bash
python tidy_tree.py \
  --alignment examples/sequences.fasta \
  --founder-sequences examples/founders.fasta \
  --guide-tree examples/guide_tree.nwk \
  --lineage-assignments examples/assignments.tsv \
  --output-tree examples/output_tree.nwk \
  --threads 2
```

## Expected Output

The tool will:
1. Build a subtree for LineageB (3 sequences + LineageB founder)
2. Build a subtree for LineageC (3 sequences + LineageC founder)
3. Build a subtree for LineageA (3 sequences + LineageA founder + LineageB founder + LineageC founder)
4. Graft LineageB and LineageC subtrees onto the LineageA subtree
5. Output the final tree to `output_tree.nwk`

The final tree will contain all 9 sequences plus the 3 founder sequences (12 tips total).
