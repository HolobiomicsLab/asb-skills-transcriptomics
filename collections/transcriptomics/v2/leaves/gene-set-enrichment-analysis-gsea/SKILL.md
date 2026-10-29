---
name: gene-set-enrichment-analysis-gsea
description: Use when genes ranked by an inferred latent process (e.g., Cyclum circular
  pseudotime match scores, or PCA components 1-2) must be validated against a curated
  gene set such as GO:0007049 to test whether the ranking captures cell-cycle biology.
when_to_use_negative:
- The goal is assigning individual cells to cell-cycle phases from marker genes (e.g.,
  Seurat::CellCycleScoring or cyclone-style prediction) rather than validating a discovered
  gene ranking.
- The gene ranking comes from a non-periodic (linear) latent variable where a circular-manifold
  model like Cyclum is not the competing method.
- No curated gene set database (e.g., GO biological process) is available for enrichment
  testing.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects gene expression onto a circular manifold
    and yields the ab-initio gene ranking (by match of expression kinetics to the
    inferred circular periodicity) used as the primary GSEA input
  repo: github.com/KChen-lab/cyclum
- name: GSEA
  role: Computes the normalized enrichment score of each ranked gene list against
    the GO:0007049 cell-cycle gene set
  repo: http://software.broadinstitute.org/gsea/index.jsp
- name: Cyclum R helpers (mat2txt, vec2cls, mat2cls)
  role: Export the expression matrix and continuous phenotypes (pseudotime or PCs)
    in GSEA-compatible .txt and .cls formats
  repo: github.com/KChen-lab/cyclum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:51d8704d77e846fbd039550573e50f7bf951c0a0b906d58982a287db1f70d809
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - Cyclum R helpers (mat2txt, vec2cls, mat2cls)
  - GSEA
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gene-set-enrichment-analysis-gsea

## Summary

Benchmark ab-initio gene rankings from a circular-manifold model (Cyclum) against PCA-based rankings by running GSEA against the GO:0007049 cell-cycle gene set and comparing normalized enrichment scores (NES). This validates that discovered genes capture periodic cell-cycle biology rather than generic linear variation.

## When to use

Use when genes ranked by an inferred latent process (e.g., Cyclum circular pseudotime match scores, or PCA components 1-2) must be validated against a curated gene set such as GO:0007049 to test whether the ranking captures cell-cycle biology.

## When NOT to use

- The goal is assigning individual cells to cell-cycle phases from marker genes (e.g., Seurat::CellCycleScoring or cyclone-style prediction) rather than validating a discovered gene ranking.
- The gene ranking comes from a non-periodic (linear) latent variable where a circular-manifold model like Cyclum is not the competing method.
- No curated gene set database (e.g., GO biological process) is available for enrichment testing.

## Inputs

- mESC scRNA-seq expression matrix preprocessed with log2(x+1), centered (mean = 1) and scaled (variance = 1)
- Cyclum-inferred circular pseudo-time for cells and ab-initio gene ranking by match to the inferred circular periodicity
- Gene rankings by PCA component 1 (PC1) and components 1-2 (PC1-2) computed on the same matrix
- GO:0007049 cell-cycle biological-process gene set

## Outputs

- gsea_nes_results.csv containing normalized enrichment scores (NES) for the Cyclum, PC1, and PC1-2 rankings against the cell-cycle gene set
- GSEA-compatible exported files: expression matrix (.txt) and continuous phenotype files (.cls)

## How to apply

Preprocess the mESC scRNA-seq matrix with log2(x+1) transformation, then center (mean = 1) and scale (variance = 1) so the circular pattern is preserved. Fit Cyclum's sinusoidal autoencoder to project expression onto the circular manifold, infer circular pseudo-time, and rank genes ab initio by the degree of match between each gene's expression kinetics and the inferred periodicity; independently compute PCA and rank genes by PC1 and by PC1-2. Export each ranking as a GSEA-ready input: write the expression matrix with mat2txt to a GSEA-compatible .txt and the continuous phenotypes (pseudotime or PCs) with vec2cls/mat2cls to a GSEA-compatible .cls. Run GSEA on each of the three ranked lists (Cyclum, PC1, PC1-2) against the GO:0007049 cell-cycle biological-process gene set and collect normalized enrichment scores. Decision point: the Cyclum-based ranking is judged superior only if its NES is the highest of the three.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects gene expression onto a circular manifold and yields the ab-initio gene ranking (by match of expression kinetics to the inferred circular periodicity) used as the primary GSEA input) — github.com/KChen-lab/cyclum
- **GSEA** (Computes the normalized enrichment score of each ranked gene list against the GO:0007049 cell-cycle gene set) — http://software.broadinstitute.org/gsea/index.jsp
- **Cyclum R helpers (mat2txt, vec2cls, mat2cls)** (Export the expression matrix and continuous phenotypes (pseudotime or PCs) in GSEA-compatible .txt and .cls formats) — github.com/KChen-lab/cyclum

## Examples

```
mat2txt(expr, "expr.txt"); vec2cls(pseudotime, "pseudotime.cls")  # then run GSEA against the GO:0007049 gene set for Cyclum, PC1, and PC1-2 rankings
```

## Evaluation signals

- Cyclum-based ranking yields NES = 1.57 for cell-cycle genes, the highest of the three rankings; PC1 gives NES = 1.07 and PC1-2 gives NES = 1.06 on mESC
- gsea_nes_results.csv exists and contains exactly three NES values (Cyclum, PC1, PC1-2)
- Exported files are GSEA-compatible: expression matrix as .txt (via mat2txt) and phenotypes as .cls (via vec2cls/mat2cls)
- Preprocessing invariant holds: data are log2(x+1) transformed, scaled to variance = 1 and centered to mean = 1, preserving the circular pattern
- Ordinal check: NES(Cyclum) > NES(PC1) and NES(Cyclum) > NES(PC1-2)

## Limitations

- NES values (1.57 vs. 1.07/1.06) are specific to the mESC dataset and this gene set; they are not transferable thresholds for other datasets
- PCA-based rankings capture only linear variation and may under-represent circular periodicity, which is the basis of the comparison rather than a flaw in GSEA itself
- Cyclum's TensorFlow/Keras dependency is version-sensitive (tested with TensorFlow 1.14.0, latest tested 1.15.2; Python 3.7), affecting reproducibility of the ranking that feeds GSEA

## Evidence

- [methods] scaled (variance = 1), centered (mean = 1) log-transformed data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed ("
- [intro] Cyclum identifies genes by match of kinetics to inferred periodicity: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [readme] README describes GSEA export helpers: [evidence span withheld — anchor status: external_doc]
- [readme] README describes continuous phenotype export for GSEA: [evidence span withheld — anchor status: external_doc]
- [abstract] Cyclum autoencoder finds circular trajectories: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
