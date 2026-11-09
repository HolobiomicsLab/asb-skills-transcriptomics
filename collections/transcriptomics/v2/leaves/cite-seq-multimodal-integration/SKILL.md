---
name: cite-seq-multimodal-integration
description: Use when evaluating or reproducing multimodal CITE-seq integration, specifically
  testing whether ALRA imputation of RNA counts improves agreement between RNA-space
  and ADT protein-space pairwise cell distances.
when_to_use_negative:
- Your matrix already has imputed values or few zeros — ALRA assumes a sparse scRNA-seq
  matrix dominated by zeros (>90% zeros in droplet-based data)
- You need methods that treat all zeros as missing data (e.g., MAGIC/DCA-style dense
  output) rather than zero-preserving imputation
- Data is not paired RNA/ADT single-cell data and there is no protein-space ground
  truth to compare distances against
tools:
- name: ALRA
  role: Performs the adaptively-thresholded low-rank approximation imputation (normalize_data,
    choose_k, alra) on the CBMC RNA matrix
  repo: github.com/KlugerLab/ALRA
- name: rsvd
  role: Provides the randomized SVD used by alra() for the rank-k approximation (prerequisite
    package)
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend for ALRA via the use.mkl flag
  repo: github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Analysis toolkit in which ALRA is integrated as RunALRA(); useful for CITE-seq
    object handling
  repo: satijalab/seurat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:8a44f369dd208bcdd1334f35f969f3627d1857c1f889004744cfbb4ed98bf381
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - Seurat
  - rpca-mkl
  - rsvd
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cite-seq-multimodal-integration

## Summary

Reproduces the ALRA-based CITE-seq benchmark in which zero-preserving low-rank imputation of scRNA-seq data makes gene-expression cell distances more consistent with ADT (protein) cell distances, improving Spearman correlation from ~0.36 to ~0.64 on the CBMC dataset.

## When to use

Use when evaluating or reproducing multimodal CITE-seq integration, specifically testing whether ALRA imputation of RNA counts improves agreement between RNA-space and ADT protein-space pairwise cell distances.

## When NOT to use

- Your matrix already has imputed values or few zeros — ALRA assumes a sparse scRNA-seq matrix dominated by zeros (>90% zeros in droplet-based data)
- You need methods that treat all zeros as missing data (e.g., MAGIC/DCA-style dense output) rather than zero-preserving imputation
- Data is not paired RNA/ADT single-cell data and there is no protein-space ground truth to compare distances against

## Inputs

- CBMC CITE-seq RNA count matrix (GEO GSE100866), 8005 cells x 20,400 genes after filtering
- CBMC ADT (protein) count matrix (GEO GSE100866)
- Normalized expression matrix (cells as rows, genes as columns) passed to alra()

## Outputs

- ALRA-imputed normalized expression matrix (result.completed[[3]])
- cbmc_distance_correlations.csv containing before/after Spearman correlations (~0.36 raw vs ~0.64 imputed)
- Pairwise Euclidean cell distance matrices in ADT space and in gene-expression space (raw and imputed)

## How to apply

Download the CBMC CITE-seq dataset (RNA counts and ADT counts) from GEO accession GSE100866 and filter out cells for which more than 90% of UMIs come from mouse genes, yielding an RNA count matrix of 8005 cells x 20,400 genes plus the ADT count matrix. Library- and log-normalize the RNA matrix with normalize_data(), set the rank to k=25 (the value reported for CBMC; alternatively use choose_k() based on singular value spacings), and run ALRA via alra() using randomized SVD from the rsvd package, passing a matrix with cells as rows and genes as columns. Compute Euclidean pairwise distances between all cell pairs in ADT (protein) space and in gene-expression space both before and after imputation, then compute the Spearman correlation of gene-space distances versus ADT-space distances for the raw and imputed matrices. Write the before/after correlations to cbmc_distance_correlations.csv; validation succeeds when the imputed Spearman correlation (~0.64) exceeds the raw one (~0.36), reflecting ALRA's ability to complete technical zeros while preserving true biological zeros.

## Related tools

- **ALRA** (Performs the adaptively-thresholded low-rank approximation imputation (normalize_data, choose_k, alra) on the CBMC RNA matrix) — github.com/KlugerLab/ALRA
- **rsvd** (Provides the randomized SVD used by alra() for the rank-k approximation (prerequisite package))
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend for ALRA via the use.mkl flag) — github.com/KlugerLab/rpca-mkl
- **Seurat** (Analysis toolkit in which ALRA is integrated as RunALRA(); useful for CITE-seq object handling) — satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); result.completed <- alra(A_norm, k=25); A_norm_completed <- result.completed[[3]]
```

## Evaluation signals

- Imputed Spearman correlation between gene-expression distances and ADT distances exceeds the raw correlation, matching the reported 0.36 -> 0.64 improvement
- ALRA preserves biological zeros: expressed-marker recovery like NCAM1 in NK cells (4% detected raw) and CR2 in B cells (1% detected raw) improves after completion
- Rank parameter matches the published value (k=25 for CBMC) or the automatically chosen k from singular value spacings
- Filtering applied correctly: cells with >90% UMIs from mouse genes removed, retaining 8005 cells x 20,400 genes
- Output file cbmc_distance_correlations.csv exists and contains both before and after Spearman values

## Limitations

- ALRA's automatically chosen k may need manual inspection; k is chosen based on spacings between singular values, which 'can be quite hard to identify the beginning of noise from just looking at the spectrum itself'
- rpca-mkl speedup is optional and only available on Linux/OS X; pre-compiled MKL libraries are not included for Windows
- Zero count fraction in droplet-based scRNA-seq can exceed 90%, so imputation quality depends heavily on the true vs technical zero structure of the dataset

## Evidence

- [abstract] Low-rank imputation preserves biological zeros while completing technical zeros: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [intro] ALRA thresholds entries using symmetry of biological zeros around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [readme] ALRA requires rsvd for randomized SVD: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [readme] ALRA input orientation: cells as rows, genes as columns: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] ALRA pipeline: rank-k approximation then per-gene thresholding and rescaling: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene. Finally, the matrix is rescaled."
