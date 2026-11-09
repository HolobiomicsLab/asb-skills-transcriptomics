---
name: active-learning-representative-selection
description: Use when bulk sequencing data cover a large cohort (e.g., the 124-sample
  COVID-19 dataset) and the budget permits single-cell sequencing of only a small
  subset, so representatives must be chosen to minimize inference difficulty Dsc.
when_to_use_negative:
- When the goal is deconvolving a single target sample from bulk data using a single-cell
  reference rather than choosing representatives across a cohort — use scSemiProfiler's
  bulk deconvolution pipeline instead
- When real single-cell data already exist for every cohort sample, so no selection
  or inference is needed
- When only cell-type-level proportions suffice — conventional deconvolution (CIBERSORTx,
  Bisque, DWLS, MuSiC, NNLS) is constrained to cell-type resolution but does not require
  representative selection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scSemiProfiler
  role: Implements the active learning module (activeselect) that ranks clusters by
    Db + λsc*Dsc + λpb*Dpb, selects splitting samples, reassigns memberships, and
    triggers scinfer in silico inference after each selection round
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Supplies the PCA dimensionality reduction (100 PCs) and the K-Means clustering
    of bulk samples on which Db distances and cluster heterogeneity ranking operate
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:3cc877e8c2ae0757e52903bcbda90b09edaaa0e99e6eae4887371d5f149487aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# active-learning-representative-selection

## Summary

An active-learning representative-selection procedure from scSemiProfiler that iteratively identifies the most heterogeneous clusters in a cohort's bulk sequencing data and selects splitting samples for real single-cell sequencing, minimizing single-cell inference difficulty (Dsc) while keeping sequencing cost at ~1/3 of full profiling.

## When to use

Use when bulk sequencing data cover a large cohort (e.g., the 124-sample COVID-19 dataset) and the budget permits single-cell sequencing of only a small subset, so representatives must be chosen to minimize inference difficulty Dsc.

## When NOT to use

- When the goal is deconvolving a single target sample from bulk data using a single-cell reference rather than choosing representatives across a cohort — use scSemiProfiler's bulk deconvolution pipeline instead
- When real single-cell data already exist for every cohort sample, so no selection or inference is needed
- When only cell-type-level proportions suffice — conventional deconvolution (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) is constrained to cell-type resolution but does not require representative selection

## Inputs

- Preprocessed bulk sequencing data for the full cohort (e.g., the 124-sample COVID-19 bulk matrix, library-size normalized and log1p transformed)
- Optional single-cell data for already-sequenced representatives, used to compute Dsc during selection
- Selection parameters: initial cluster count B (B=4 default), heterogeneity weights λsc and λpb (both 1 by default), and the target number of representatives (e.g., 28)

## Outputs

- Ordered list of selected representative samples per round with updated sample-to-representative cluster memberships
- Per-count inference difficulty Dsc values for active vs. passive selection, tabulated as selection_comparison.csv
- The selected representative set feeding downstream semi-profiling (semiprofiled_cohort.h5ad) and cost accounting via scSemiProfiler.utils.estimate_cost (cost_summary.csv)

## How to apply

Reduce the library-size-normalized, log1p-transformed bulk cohort to 100 principal components with scikit-learn PCA (use 100 PCs whenever the cohort exceeds 100 samples), then K-Means the samples into B clusters (B=4 for the 124-sample COVID-19 cohort) and designate the sample closest to each centroid as the initial representatives. At each round, rank clusters by combined heterogeneity Db + λsc*Dsc + λpb*Dpb with λsc = λpb = 1, where Db is the Euclidean bulk distance and Dsc is single-cell inference difficulty (average Euclidean distance between representative cells and their K=1 nearest neighbors in the target sample within the PCA-reduced space). In the top-ranked cluster, select the splitting sample Rnew = argmin over j of the sum of min(Db(j,i), Db(Rc,i)) to minimize total in-cluster heterogeneity, reassign every sample to its closest representative, and re-run in silico inference via scSemiProfiler's activeselect/scinfer functions. Iterate — six further rounds in batches of 4 raised the COVID-19 cohort from 4 to 28 representatives — stopping when the representative budget (README: under 1/3 of full single-cell cost) or a Dsc plateau is reached. The rationale is that passive bulk-only selection (picking the sample with the largest Db per round) leaves high-Dsc clusters unprofiled: at 28 representatives active learning achieves an inference difficulty that passive selection only matches with 4 additional representatives.

## Related tools

- **scSemiProfiler** (Implements the active learning module (activeselect) that ranks clusters by Db + λsc*Dsc + λpb*Dpb, selects splitting samples, reassigns memberships, and triggers scinfer in silico inference after each selection round) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Supplies the PCA dimensionality reduction (100 PCs) and the K-Means clustering of bulk samples on which Db distances and cluster heterogeneity ranking operate)

## Examples

```
import scSemiProfiler; print(scSemiProfiler.utils.activeselect.__doc__)
```

## Evaluation signals

- At 28 representatives on the COVID-19 cohort, the active selector's average Dsc is lower than the passive selector's at the same representative count
- Passive selection needs roughly 4 additional representatives to reach the inference difficulty achieved by active learning at 28 representatives
- Initial representatives equal the samples closest to the K-Means centroids of the B=4 clusters in the 100-PC bulk space
- selection_comparison.csv exists and Dsc decreases monotonically as representatives are added under active learning
- Downstream cost check: 28 representatives cost about $62,640, a 0.30–0.37 ratio to the $186,000 full single-cell cost (within $60,000–$66,000)

## Limitations

- Computing Dsc requires single-cell data from already-sequenced representatives; before any sequencing, selection must fall back to bulk-only Db ranking or the global mode that assesses heterogeneity with Silhouette scores
- The resulting semi-profiled data are computationally inferred, not real measurements, and must be validated against ground-truth single-cell data (e.g., UMAP visualization and cell type proportions) before biological conclusions
- Extension to modalities beyond scRNA-seq (e.g., proteomics) is future work and would require algorithm adjustments and new benchmarking

## Evidence

- [methods] Active learning module ranks clusters and picks splitting samples: "Incrementally Select Representatives Using Active Learning To Improve Single-Cell Inference (Active Learning)"
- [methods] K-Means clustering of dimensionality-reduced bulk data defines the B clusters over which heterogeneity is ranked: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87"
- [methods] PCA implemented in scikit-learn provides the reduced space for Db and Dsc distances: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [methods] Global mode is the bulk-only alternative when no representative single-cell data exist yet: [evidence span withheld — anchor status: not_found]
- [discussion] Pipeline-level role of the active learning module: "The pipeline ingests bulk data, leverages an active learning module to judiciously select representatives for authentic single-cell sequencing"
- [readme] README framing of cohort representative selection and expected cost savings: [evidence span withheld — anchor status: external_doc]
