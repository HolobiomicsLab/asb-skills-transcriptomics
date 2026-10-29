---
name: mitochondrial-read-exclusion
description: Use when quantifying per-droplet contamination features (e.g., percent
  spliced reads, debris scores) in droplet snRNA-seq, where mitochondrial reads must
  be excluded so the metric reflects splicing status rather than extranuclear debris
  content.
when_to_use_negative:
- When the goal is to measure mitochondrial contamination itself (e.g., plotting pct.mt
  vs. debris score) — mito reads must be counted there, not excluded
- When input is already a filtered feature/barcode matrix without gene-level mapping,
  since mito genes can no longer be identified and excluded by pattern
- For scRNA-seq filtering with EmptyDrops-style assumptions, where mito-based debris
  scoring behaves differently than in snRNA-seq
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: diem
  role: Provides read_10x, get_gene_pct, and create_SCE used to load 10X raw counts,
    flag mitochondrial genes, and hold per-droplet metrics before debris scoring
  repo: https://github.com/marcalva/diem
- name: R
  role: Implementation language for the sparse-matrix (Matrix package) counts and
    the percent-spliced computation
- name: Matrix
  role: Stores the raw 10X counts as a column-wise sparse matrix for efficient per-droplet
    UMI tallying
- name: CellRanger
  role: Upstream aligner producing the raw_feature_bc_matrix droplet counts consumed
    by this step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:2dc291f22f2cb7608c8bcfceef75b3800f6d7d67edc180380d0eb368dbfdb169
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mitochondrial-read-exclusion

## Summary

Excludes reads mapping to mitochondrial genes when computing per-droplet metrics such as percent-spliced-reads for snRNA-seq contamination quantification, because mito-encoded genes are the canonical debris-upregulated markers and their inclusion conflates the debris signal with the nuclear/background separation metric.

## When to use

Use when quantifying per-droplet contamination features (e.g., percent spliced reads, debris scores) in droplet snRNA-seq, where mitochondrial reads must be excluded so the metric reflects splicing status rather than extranuclear debris content.

## When NOT to use

- When the goal is to measure mitochondrial contamination itself (e.g., plotting pct.mt vs. debris score) — mito reads must be counted there, not excluded
- When input is already a filtered feature/barcode matrix without gene-level mapping, since mito genes can no longer be identified and excluded by pattern
- For scRNA-seq filtering with EmptyDrops-style assumptions, where mito-based debris scoring behaves differently than in snRNA-seq

## Inputs

- Raw droplet count matrix from 10X CellRanger output (outs/raw_feature_bc_matrix), stored as a column-wise sparse Matrix (R Matrix package)
- Gene metadata with gene names/symbols to pattern-match mitochondrial genes (e.g., mt- prefixed)

## Outputs

- Per-droplet percent-spliced-reads table (e.g., percent_spliced.csv) with mitochondrial reads excluded
- Optional per-droplet mito percent column (pct.mt) retained as a separate debris indicator

## How to apply

From the CellRanger output (e.g., outs/raw_feature_bc_matrix of the 10X nuclei_2k fresh mouse brain dataset), load the raw droplet counts into a column-wise sparse matrix with the diem read_10x helper. Identify mitochondrial genes by pattern (e.g., grep '^mt-' on gene names, case-insensitive as in the diem README) and either exclude mito reads when tallying spliced vs. unspliced UMIs per droplet or compute the mito percent separately with get_gene_pct (name='pct.mt') as a parallel debris indicator. The rationale, per the paper, is that percent of reads originating from mitochondria is itself 'a measure of extranuclear contamination', so leaving mito genes inside the percent-spliced numerator/denominator distorts the bimodal nuclear-vs-background distribution used for midpoint estimation. The clean percent-spliced per-droplet table is then written out (e.g., percent_spliced.csv) and fed to the two-component Gaussian-mixture step to derive a sample-specific midpoint (~50% separation) for classifying nuclear vs. background droplets.

## Related tools

- **diem** (Provides read_10x, get_gene_pct, and create_SCE used to load 10X raw counts, flag mitochondrial genes, and hold per-droplet metrics before debris scoring) — https://github.com/marcalva/diem
- **R** (Implementation language for the sparse-matrix (Matrix package) counts and the percent-spliced computation)
- **Matrix** (Stores the raw 10X counts as a column-wise sparse matrix for efficient per-droplet UMI tallying)
- **CellRanger** (Upstream aligner producing the raw_feature_bc_matrix droplet counts consumed by this step)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); mt_genes <- grep(pattern="^mt-", x=rownames(sce@gene_data), ignore.case=TRUE, value=TRUE); sce <- get_gene_pct(x=sce, genes=mt_genes, name="pct.mt")
```

## Evaluation signals

- percent_spliced.csv contains exactly one row per quantified droplet from the raw CellRanger matrix
- The sample-specific midpoint computed downstream parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation
- Mito gene identifiers (e.g., mt- prefixed) are confirmed absent from the spliced/unspliced UMI tallies used for percent-spliced
- Per-droplet mito percent (pct.mt), when computed in parallel, shows the expected high values in debris/background droplets relative to nuclear droplets

## Limitations

- Mito percent alone is an ad hoc filtering aid; the paper notes the percent of mitochondrial reads 'can help to filter droplets' but DIEM instead uses expression-profile scoring
- Debris scoring relies on accurate estimation of debris-enriched genes; if few or no genes (including mito markers) are increased in the background distribution, resulting scores will likely be inaccurate
- Percent-spliced contamination correlation was less successful in PBMC single-cell RNA-seq data due to smaller differences between background and cell type profiles

## Evidence

- [intro] ad hoc mitochondrial cutoff as debris measure: "Additionally, an ad hoc cutoff of the percent of reads originating from the mitochondria (a measure of extranuclear contamination) can help to filter droplets"
- [readme] mito gene identification in diem README: [evidence span withheld — anchor status: external_doc]
- [readme] get_gene_pct mito percent computation: [evidence span withheld — anchor status: external_doc]
- [methods] sparse matrix storage of raw counts: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
