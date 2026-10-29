---
name: weighted-knn-imputation-of-as-probability-matrices
description: Use when a single-cell scRNA-seq-derived AS probability matrix (per-site
  PSI-like values from Leafcutter junction reads) contains NA dropouts that must be
  filled before unsupervised cell clustering.
when_to_use_negative:
- The matrix is already complete (no NA dropouts) — imputation would be unnecessary
  and may add noise
- The input is a raw gene expression count matrix rather than an AS probability matrix
  from junction reads; SCASL's imputation is designed for splicing landscapes
- The analysis goal is differential splicing testing with known cell labels (BRIE18,
  Expedition19, rMATS territory) rather than unsupervised splicing-based clustering
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Implements the iterative weighted KNN imputation of the AS probability matrix
    within its main pipeline (main.py with configs/srr.yaml)
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: 'Upstream: identifies junction reads and defines AS events from which the
    AS probability matrix to be imputed is computed'
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:5b70b503f729d5ad8f37e6413a0dc20317b928b3fcf5ee7f46153035ef1985fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# weighted-knn-imputation-of-as-probability-matrices

## Summary

An iterative, weighted KNN imputation step within the SCASL workflow that fills missing values (dropouts) in the single-cell alternative splicing (AS) probability matrix so that downstream spectral clustering can operate on a complete splicing landscape.

## When to use

Use when a single-cell scRNA-seq-derived AS probability matrix (per-site PSI-like values from Leafcutter junction reads) contains NA dropouts that must be filled before unsupervised cell clustering.

## When NOT to use

- The matrix is already complete (no NA dropouts) — imputation would be unnecessary and may add noise
- The input is a raw gene expression count matrix rather than an AS probability matrix from junction reads; SCASL's imputation is designed for splicing landscapes
- The analysis goal is differential splicing testing with known cell labels (BRIE18, Expedition19, rMATS territory) rather than unsupervised splicing-based clustering

## Inputs

- Normalized AS probability matrix (site rows x cell columns) with NA entries, produced after Leafcutter junction grouping and quality-control filtering (filtered_matrix_start/end, normalized_matrix)
- SCASL YAML configuration (configs/srr.yaml) with num_iteration and knn parameters

## Outputs

- Imputed full AS probability matrix (missing values filled), plus NA position information retained for downstream concatenation
- Input to spectral clustering yielding cell AS cluster labels (preds)

## How to apply

After SCASL builds the junction reads matrix with Leafcutter, applies the four QC filters (sites_initial, runs_initial, sites_quality, runs_quality), and normalizes junction read proportions into an AS probability matrix per site, run the imputation: an iterative, weighted KNN process controlled by the num_iteration (number of imputation iterations) and knn (number of adjacent cells) parameters in configs/srr.yaml. The rationale is that single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification, and coverage of AS events is unbalanced across cells. Concatenate the imputed matrix with the NA indicator matrix before spectral clustering so the pattern of originally missing entries still informs cluster assignment. Because imputation is stochastic, re-running with identical parameters may give minor run-to-run differences while remaining generally consistent.

## Related tools

- **SCASL** (Implements the iterative weighted KNN imputation of the AS probability matrix within its main pipeline (main.py with configs/srr.yaml)) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Upstream: identifies junction reads and defines AS events from which the AS probability matrix to be imputed is computed) — https://github.com/davidaknowles/leafcutter

## Examples

```
source activate scasl && python main.py -y configs/srr.yaml  # with num_iteration and knn set in configs/srr.yaml
```

## Evaluation signals

- The normalized_matrix intermediate file contains no remaining NA entries after imputation, while NA position information is preserved for the clustering concatenation
- SCASL clusters derived from the imputed matrix reproduce known lineage orderings, e.g. hepatocyte sequence C2-C10-C3-C1-C8-C5 and cholangiocyte sequence C0-C9-C6 consistent with embryonic day progression
- Re-running with identical num_iteration and knn parameters yields generally consistent results, allowing only minor run-to-run differences from imputation randomness
- The AS clusters recover cell identities that are not readily available via conventional clustering based on gene expression data (e.g., precancerous/early-tumor TNBC clusters), as reported in the paper

## Limitations

- Imputation, dimensionality reduction, and unsupervised clustering are stochastic, so running results may differ slightly between runs even with identical parameters
- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification; coverage of genes and AS events can be very unbalanced across single cells, straining imputation
- Many canonical clustering methods are unsuitable for processing the largely polarized AS profiles even after imputation, which is why SCASL uses spectral clustering with an NA indicator matrix
- RNA splicing profiling (SMART-seq style inputs) is much more costly than gene expression quantification, limiting dataset scale

## Evidence

- [discussion] Iterative weighted KNN imputation of missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] AS profiles dropout rationale: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis"
- [discussion] Clustering on imputed matrix plus NA indicator: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] Imputation parameters in config: "**num_iteration**: the number of iterations during weighted KNN imputation. - **knn**: the number of adjacent cells used in weighted KNN imputation."
- [intro] AS probability matrix construction preceding imputation: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
