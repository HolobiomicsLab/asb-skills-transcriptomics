---
name: circular-pseudotime-gene-ranking
description: Use when scRNA-seq cells plausibly follow a periodic process (e.g., cell
  cycle) and you need to identify the associated genes ab initio from expression kinetics
  rather than marker genes, then benchmark the ranking against PCA-based alternatives.
when_to_use_negative:
- The process of interest is non-periodic/linear (e.g., developmental trajectory)
  — linear pseudo-time methods like Monocle, not a circular manifold, are appropriate.
- You already trust marker-gene or label-based cell-cycle scoring (e.g., scanpy.tl.score_genes_cell_cycle,
  Seurat::CellCycleScoring) and only need to refine labels along pseudo-time via cyclum.postproc.refine_labels
  rather than re-rank genes.
- Input data cannot support a circular manifold assumption (e.g., asynchronous non-cycling
  cells), where ab-initio periodicity inference is meaningless.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects gene expression onto a circular manifold,
    infers circular pseudo-time, and provides gene rankings by match to the inferred
    periodicity; its R utilities (mat2txt, vec2cls, mat2cls) bridge the rankings into
    GSEA.
  repo: https://github.com/KChen-lab/Cyclum
- name: GSEA
  role: Computes normalized enrichment scores of the Cyclum- and PCA-ranked gene lists
    against the GO:0007049 cell-cycle gene set for benchmarking.
  repo: http://software.broadinstitute.org/gsea/index.jsp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:24e4aac1190a1f4e5b16ed87111bdd4328773665403dca153aa83a8e8fce7b21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - GSEA
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# circular-pseudotime-gene-ranking

## Summary

Rank genes ab initio by the degree of match between each gene's expression kinetics and a Cyclum-inferred circular pseudo-time, and validate that ranking against PC1/PC2-based rankings using GSEA normalized enrichment scores (NES) against the GO:0007049 cell-cycle gene set.

## When to use

Use when scRNA-seq cells plausibly follow a periodic process (e.g., cell cycle) and you need to identify the associated genes ab initio from expression kinetics rather than marker genes, then benchmark the ranking against PCA-based alternatives.

## When NOT to use

- The process of interest is non-periodic/linear (e.g., developmental trajectory) — linear pseudo-time methods like Monocle, not a circular manifold, are appropriate.
- You already trust marker-gene or label-based cell-cycle scoring (e.g., scanpy.tl.score_genes_cell_cycle, Seurat::CellCycleScoring) and only need to refine labels along pseudo-time via cyclum.postproc.refine_labels rather than re-rank genes.
- Input data cannot support a circular manifold assumption (e.g., asynchronous non-cycling cells), where ab-initio periodicity inference is meaningless.

## Inputs

- mESC scRNA-seq gene-by-cell expression matrix (raw counts or log-ready intensities)
- Cyclum-inferred circular pseudo-time per cell (continuous phenotype)
- PCA components PC1 and PC2 per cell (continuous phenotypes)
- GO:0007049 cell-cycle biological-process gene set (GMT)

## Outputs

- Three gene-ranked lists (Cyclum-periodicity match, PCA PC1, PCA PC1-PC2)
- GSEA-compatible expression matrix (.txt) and phenotype (.cls) files
- gsea_nes_results.csv with NES for cell-cycle enrichment per ranking (expected: Cyclum NES = 1.57; PC1 NES = 1.07; PC2 NES = 1.06)

## How to apply

Preprocess the mESC scRNA-seq expression matrix with log2(x+1) transformation, then center (mean = 1) and scale (variance = 1) so the circular pattern is preserved. Fit Cyclum, a sinusoidal autoencoder that projects high-dimensional gene expression onto a circular manifold, to infer each cell's circular pseudo-time. Rank genes by the degree of match between each gene's expression kinetics and the inferred periodicity; in parallel, run PCA on the same data and rank genes by PC1 and by PC1-PC2 as comparators. Export each ranked list and phenotype to GSEA format using the Cyclum R utilities mat2txt (expression matrix to GSEA-compatible .txt), vec2cls/mat2cls (continuous phenotypes such as pseudo-time or PCs to .cls). Run GSEA on all three rankings against the GO:0007049 cell-cycle biological-process gene set and collect the normalized enrichment scores into gsea_nes_results.csv.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects gene expression onto a circular manifold, infers circular pseudo-time, and provides gene rankings by match to the inferred periodicity; its R utilities (mat2txt, vec2cls, mat2cls) bridge the rankings into GSEA.) — https://github.com/KChen-lab/Cyclum
- **GSEA** (Computes normalized enrichment scores of the Cyclum- and PCA-ranked gene lists against the GO:0007049 cell-cycle gene set for benchmarking.) — http://software.broadinstitute.org/gsea/index.jsp

## Examples

```
Rscript -e 'mat2txt(expr, "expr.txt"); mat2cls(list(Cyclum=pseudotime, PC1=pc1, PC2=pc2), "pheno.cls")'
```

## Evaluation signals

- The NES of the Cyclum-based ranking is the highest of the three rankings and reproduces the reported value of 1.57.
- The PCA-based NES values reproduce the reported 1.07 (PC1) and 1.06 (PC2).
- After preprocessing, the transformed expression matrix has variance = 1 and mean = 1 (scaled, centered log2(x+1) data).
- gsea_nes_results.csv contains NES entries for exactly the three ranked lists (Cyclum, PC1, PC1-PC2).
- CyclumAutoTune selects a sensible number of linear components so the inferred manifold is 'most circular' before ranking.

## Limitations

- Cyclum is a model-based approach that fits data to predefined circular manifolds — it targets periodic processes only; for general cell-cycle characterization it outperformed model-free reCAT, but non-periodic biology is out of scope.
- The pipeline is Python/Keras/TensorFlow-based and requires TensorFlow 1.x (tested on TF 1.14.0, latest tested 1.15.2); pip will pull latest dependencies that differ from tested versions.
- The reported preprocessing (centered mean = 1, scaled variance = 1, log2(x+1)) is stated to 'work well' for preserving the circular pattern; other transforms may not.

## Evidence

- [intro] Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity.: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [methods] scaled, centered, log2(x+1) preprocessing: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] GSEA export utilities: [evidence span withheld — anchor status: external_doc]
- [readme] continuous phenotypes for GSEA: [evidence span withheld — anchor status: not_found]
- [discussion] model-based circular manifold rationale: "Cyclum is a model-based approach that fits the data to predefined circular manifolds."
