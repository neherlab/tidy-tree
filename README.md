# Tidy Tree

Build phylogenetic trees guided by hierarchical lineage structures.

## Overview

Tidy Tree constructs phylogenetic trees by building subtrees for individual lineages and stitching them together according to a lineage guide tree. This approach is useful when you have:

- A hierarchical classification of sequences into lineages
- Representative founder sequences for each lineage
- A known or hypothesized relationship between lineages

## Algorithm

The tool works in three main steps:

1. **Assign sequences to lineages**: Each input sequence is assigned to its lineage based on the provided mapping table

2. **Build subtrees**: For each lineage, a subtree is constructed containing:
   - All sequences belonging to that lineage
   - The founder sequence of that lineage
   - The founder sequences of its child lineages (connection points)

3. **Stitch subtrees**: Subtrees are combined following the lineage guide tree structure. Child lineage subtrees are grafted onto parent subtrees at the positions of the child founder sequences.

The subtrees are built using [IQ-TREE](http://www.iqtree.org/), a fast and accurate phylogenetic inference tool.

## Requirements

- Python 3.7+
- BioPython
- IQ-TREE (must be installed separately)

## Installation

1. Clone or download this repository

2. Install Python dependencies:
```bash
pip install -r requirements.txt
```

3. Install IQ-TREE:
   - Download from http://www.iqtree.org/
   - Or install via conda: `conda install -c bioconda iqtree`
   - Or install via apt: `sudo apt install iqtree`

## Usage

```bash
python tidy_tree.py \
  --alignment aligned_sequences.fasta \
  --founder-sequences aligned_founders.fasta \
  --guide-tree lineage_tree.newick \
  --lineage-assignments assignments.tsv \
  --output-tree output_tree.newick
```

### Required Arguments

- `--alignment`, `-s`: Aligned input sequences in FASTA format
- `--founder-sequences`, `-f`: Aligned lineage founder sequences in FASTA format
- `--guide-tree`, `-g`: Lineage guide tree in Newick format
- `--lineage-assignments`, `-a`: Tab-separated file mapping sequences to lineages
- `--output-tree`, `-o`: Output file for the final tree (Newick format)

### Optional Arguments

- `--seq-id-column`: Column name for sequence IDs in assignments file (default: `seq_id`)
- `--lineage-column`: Column name for lineage in assignments file (default: `lineage`)
- `--iqtree-path`: Path to IQ-TREE executable (default: `iqtree`)
- `--model`: Substitution model for IQ-TREE (default: `GTR+G`)
- `--threads`: Number of CPU threads for IQ-TREE (default: 1)
- `--iqtree-args`: Additional arguments to pass to IQ-TREE
- `--ignore-missing-founders`: Ignore lineages in guide tree that lack founder sequences (by default, the script will error if founders are missing)
- `--keep-founders`: Keep founder sequences as leaves in the final tree
- `--root-lineage`: Root lineage for the tree

## Input File Formats

### 1. Sequences (`--sequences`)

Standard FASTA format with aligned sequences:

```
>seq001
ATGCGATCGATCG---ATCG
>seq002
ATGCGATCGATCGATGATCG
>seq003
ATGCGAT---TCGATGATCG
```

**Important**: All sequences must be pre-aligned and have the same length.

### 2. Founder Sequences (`--founder-sequences`)

FASTA format with one founder sequence per lineage:

```
>LineageA
ATGCGATCGATCGATGATCG
>LineageB
ATGCGAT---TCGATGATCG
>LineageC
ATGCGATCG---GATGATCG
```

**Important**:
- Founder sequence IDs must match the lineage names in the guide tree
- Founders must be aligned with the same alignment as the input sequences

### 3. Guide Tree (`--guide-tree`)

Newick format describing the hierarchical relationship between lineages:

```
(LineageA,(LineageB,LineageC));
```

Or with branch lengths:

```
(LineageA:0.1,(LineageB:0.05,LineageC:0.05):0.05);
```

### 4. Assignments (`--lineage-assignments`)

Tab-separated file mapping sequence IDs to lineage names. Must include a header row with column names:

```
seq_id	lineage
seq001	LineageA
seq002	LineageA
seq003	LineageB
seq004	LineageB
seq005	LineageC
```

**Important**:
- The file must have a header row with column names
- Default column names are `seq_id` and `lineage`, but you can specify different names using `--seq-id-column` and `--lineage-column`
- Lines starting with `#` are treated as comments and ignored

Example with custom column names:
```
sequence_name	clade
seq001	LineageA
seq002	LineageA
```

Then use: `--seq-id-column sequence_name --lineage-column clade`

## Example

See the `examples/` directory for a complete example with sample data:

```bash
python tidy_tree.py \
  --alignment examples/sequences.fasta \
  --founder-sequences examples/founders.fasta \
  --guide-tree examples/guide_tree.nwk \
  --lineage-assignments examples/assignments.tsv \
  --output-tree examples/output_tree.nwk \
  --model GTR+F+I+G4 \
  --threads 4
```

With additional context:
```bash
python tidy_tree.py \
  --alignment examples/sequences.fasta \
  --founder-sequences examples/founders.fasta \
  --guide-tree examples/guide_tree.nwk \
  --lineage-assignments examples/assignments.tsv \
  --output-tree examples/output_tree.nwk \
  --contextual-sequences examples/context.fasta \
  --root-context c1 \
  --threads 4
```



## Advanced Options

### Substitution Models

Common models for DNA sequences:
- `GTR+G`: General time reversible with gamma rate heterogeneity (default)
- `GTR+F+I+G4`: GTR with empirical base frequencies, invariant sites, and 4 rate categories
- `HKY+G`: Hasegawa-Kishino-Yano with gamma rates
- `JC`: Jukes-Cantor (simplest model)

For model selection, you can use `TEST` or `MFP`:
```bash
--model TEST
```

### Additional IQ-TREE Arguments

Pass extra arguments to IQ-TREE using `--iqtree-args`:

```bash
--iqtree-args "-bb 1000 -alrt 1000"
```

This would add 1000 ultrafast bootstrap replicates and SH-aLRT branch support.

## Output

The program outputs a single Newick tree file containing:
- All input sequences
- All lineage founder sequences
- Tree topology reflecting both within-lineage relationships and between-lineage relationships

## Troubleshooting

**IQ-TREE not found**:
```
Use --iqtree-path to specify the full path to the IQ-TREE executable
```

**Sequence length mismatch**:
```
All sequences must be pre-aligned with the same length
```

**Lineage not found**:
```
Check that lineage names match exactly between the guide tree, founders, and assignments files
```

## License

MIT License - see LICENSE file for details
