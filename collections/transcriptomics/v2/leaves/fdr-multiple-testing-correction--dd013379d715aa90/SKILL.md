---
name: fdr-multiple-testing-correction
description: Use when a DE analysis (limma on RNA-seq counts, e.g., ExpressAnalystR
  PerformDEAnal output) yields thousands of raw p-values and DEGs must be called at
  a defensible multiple-testing-adjusted threshold matching a reference publication.
when_to_use_negative:
- Input p-values are already FDR-adjusted — re-applying the adjustment would double-correct
  and over-conservatively shrink the DEG set
- Only a small pre-selected set of genes is being tested rather than a well-defined
  genome-wide family of simultaneous tests, where BH FDR is underpowered
- Reproducing a meta-analysis instead of a single-dataset DE analysis — the meta path
  uses PerformDEAnalMeta with its own thresholds (0.05, 0.0), not the single-data
  FDR workflow
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: ExpressAnalystR
  role: Runs the limma-based differential expression analysis (PerformDEAnal) and
    applies the FDR adjustment to p-values before DEG calling
  repo: github.com/xia-lab/ExpressAnalystR
- name: limma
  role: voom normalization (log2 CPM) and moderated statistics whose raw p-values
    are subsequently FDR-adjusted for DEG calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:a39c7b473040b5ab760fd0e2950bb6b58c970e4355a3836b0c9fccabc39fd5a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - limma
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fdr-multiple-testing-correction

## Summary

Apply False Discovery Rate (FDR) adjustment to raw p-values from differential expression analysis and call DEGs using the study-specific FDR threshold — e.g., FDR < 0.1 (no log2FC cut-off) for the salamander case, or the default FDR < 0.05 with |log2FC| > 1.5 — using ExpressAnalystR's limma-based PerformDEAnal.

## When to use

Use when a DE analysis (limma on RNA-seq counts, e.g., ExpressAnalystR PerformDEAnal output) yields thousands of raw p-values and DEGs must be called at a defensible multiple-testing-adjusted threshold matching a reference publication.

## When NOT to use

- Input p-values are already FDR-adjusted — re-applying the adjustment would double-correct and over-conservatively shrink the DEG set
- Only a small pre-selected set of genes is being tested rather than a well-defined genome-wide family of simultaneous tests, where BH FDR is underpowered
- Reproducing a meta-analysis instead of a single-dataset DE analysis — the meta path uses PerformDEAnalMeta with its own thresholds (0.05, 0.0), not the single-data FDR workflow

## Inputs

- Normalized count matrix (limma voom log2 CPM) from the salamander count table (samples SRR7499348–SRR7499365, excluding outlier SRR7499350)
- limma design matrix with time as primary factor and species as blocking factor
- DE results table from PerformDEAnal containing logFC, raw p-value, and contrast direction (e.g., time0 vs. time24)

## Outputs

- DE results table with per-gene log2FC, raw p-value, and FDR-adjusted p-value (written to salamander_de_results.csv)
- DEG count at the chosen FDR threshold (expected 2780 for the salamander case at FDR < 0.1)

## How to apply

After building the limma design matrix (SetupDesignMatrix("limma") with the metadata of interest and any blocking factor such as species in the salamander case), run the contrast via PerformDEAnal (e.g., "custom", "time0 vs. time24"), which produces per-gene log2 fold changes and raw p-values. ExpressAnalystR then adjusts p-values using the False Discovery Rate method, and DEGs are those whose FDR-adjusted p-values fall below the chosen cut-off. The threshold must match the reference study: the salamander publication defines DEGs only by FDR < 0.1 with no log2FC cut-off, while the article's default practice uses FDR < 0.05 plus absolute log fold change > 1.5, and pathway enrichment uses FDR < 0.05 with at least five DEGs per gene set. Write the complete DE table (logFC, p-value, FDR-adjusted p-value per gene) to CSV and record the DEG count — for the 17-sample salamander analysis (SRR7499348–SRR7499365 minus outlier SRR7499350), the correct count is 2780 DEGs.

## Related tools

- **ExpressAnalystR** (Runs the limma-based differential expression analysis (PerformDEAnal) and applies the FDR adjustment to p-values before DEG calling) — github.com/xia-lab/ExpressAnalystR
- **limma** (voom normalization (log2 CPM) and moderated statistics whose raw p-values are subsequently FDR-adjusted for DEG calling)

## Examples

```
dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly");
```

## Evaluation signals

- The DEG count at FDR-adjusted p < 0.1 (no log2FC cut-off) for the salamander analysis reproduces the paper's 2780 DEGs
- Output table salamander_de_results.csv contains the three required columns: logFC, p-value, FDR-adjusted p-value for every tested gene
- FDR-adjusted p-values are monotonically ordered with raw p-values and bounded in [0,1]; no gene with a raw p-value lacks an adjusted value
- Applying the alternative default rule (FDR < 0.05 and |log2FC| > 1.5) should yield a strictly smaller DEG set than the salamander FDR < 0.1 no-fold-change rule, sanity-checking the threshold choice
- Contrast direction is explicitly recorded (e.g., time0 vs. time24) so the sign of logFC is interpretable

## Limitations

- Thresholds are study-specific: the salamander publication uses only FDR < 0.1 with no log2FC cut-off, while the article's other case uses FDR < 0.05 plus |log2FC| > 1.5 — mixing thresholds breaks reproducibility against a reference count
- Calling DEGs at FDR < 0.1 without a fold-change cut-off yields a large DEG set (2780) that may include small-effect genes, needing careful downstream interpretation
- ExpressAnalystR is under active development; the README warns functionality cannot be guaranteed and functions must be executed in sequence without skipping commands

## Evidence

- [discussion] Salamander DEG threshold at FDR < 0.1: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [discussion] Default DEG threshold with fold-change cutoff: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [discussion] Pathway enrichment threshold: "A pathway was defined as significantly enriched if the FDR-adjusted p-value was less than 0.05 and there were at least five DEGs in the gene set."
- [readme] FDR adjustment is part of the DE pipeline: "Adjusted P-value using False Discovery Rate (FDR) method."
- [readme] Concrete PerformDEAnal invocation: "dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly");"
