---
name: single-cell-batch-mixing-metric-implementation
description: Use when evaluating batch mixing of integrated single-cell RNA-seq datasets
  with imbalanced cell type proportions, where standard cell type-agnostic iLISI scores
  are mis and a per-cell-type normalized metric is needed.
when_to_use_negative:
- 'Cell type annotations are unavailable: ''One obvious limitation of CiLISI and similar
  metrics is that it requires cell type annotations'''
- A cell type is present in only one of the datasets being compared, since CiLISI
  cannot be calculated for such types
- Integrating technical replicates with matched cell compositions, where cell type-agnostic
  batch mixing metrics remain appropriate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scIntegrationMetrics
  role: R package providing LISI, per-cell type LISI (CiLISI), and silhouette average
    width (ASW) used as the reference implementation and comparator for the reimplemented
    metric
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: immunogenomics/LISI
  role: Underlying LISI algorithm that scIntegrationMetrics builds upon; run per cell
    type at perplexity 30 in the reimplementation
  repo: https://github.com/immunogenomics/LISI
- name: STACAS
  role: Semi-supervised integration method whose benchmark pipeline (scib-pipeline)
    uses CiLISI as its integration metric
  repo: https://github.com/carmonalab/STACAS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:bdb3a156bbe4a771386c500d89701fdaf157b41e5150821799d55455aa48c207
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - STACAS
  - immunogenomics/LISI
  - scIntegrationMetrics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-batch-mixing-metric-implementation

## Summary

Reimplements the CiLISI (cell type-aware iLISI) batch-mixing metric from the STACAS integration benchmark: a normalized iLISI computed per cell type that scores 1 when cells of the same type are well mixed across batches and 0 when batches do not mix. This avoids the inflation of standard iLISI caused by cell type imbalance across datasets.

## When to use

Use when evaluating batch mixing of integrated single-cell RNA-seq datasets with imbalanced cell type proportions, where standard cell type-agnostic iLISI scores are misleading and a per-cell-type normalized metric is needed.

## When NOT to use

- Cell type annotations are unavailable: 'One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations'
- A cell type is present in only one of the datasets being compared, since CiLISI cannot be calculated for such types
- Integrating technical replicates with matched cell compositions, where cell type-agnostic batch mixing metrics remain appropriate

## Inputs

- Simulated single-cell integration datasets with batch origin and cell type annotations (e.g., Seurat objects or annotated count matrices)
- Batch origin labels per cell
- Cell type annotation labels per cell

## Outputs

- cilisi_scores.csv containing per-cell-type CiLISI scores
- Reference batch-mixing scores computed with the scIntegrationMetrics package

## How to apply

Load the simulated single-cell integration datasets carrying batch-origin and cell type annotations produced by the upstream simulation task. For each cell type present in both datasets, run the LISI algorithm independently on the subset of cells of that type with perplexity set to 30, then min-max rescale each per-cell-type iLISI score to the [0,1] interval and drop cell types occurring in only one dataset, as required by the CiLISI definition. In parallel, compute the reference batch-mixing score with the scIntegrationMetrics package on the same inputs. Validate the reimplementation by confirming the per-cell-type CiLISI values match the scIntegrationMetrics reference output within a small numerical tolerance, then write the per-cell-type scores to cilisi_scores.csv. The rationale is that iLISI computed on all cells conflates batch mixing with cell type separation when cell type abundance differs across samples, which is a major component of biological variance and particularly challenging for integration methods.

## Related tools

- **scIntegrationMetrics** (R package providing LISI, per-cell type LISI (CiLISI), and silhouette average width (ASW) used as the reference implementation and comparator for the reimplemented metric) — https://github.com/carmonalab/scIntegrationMetrics
- **immunogenomics/LISI** (Underlying LISI algorithm that scIntegrationMetrics builds upon; run per cell type at perplexity 30 in the reimplementation) — https://github.com/immunogenomics/LISI
- **STACAS** (Semi-supervised integration method whose benchmark pipeline (scib-pipeline) uses CiLISI as its integration metric) — https://github.com/carmonalab/STACAS

## Evaluation signals

- Reimplemented CiLISI values match scIntegrationMetrics package output within a small numerical tolerance on the simulated datasets
- Per-cell-type scores approach 1 when cells of the same type are well mixed across batches and approach 0 when cells from different batches do not mix
- All scores are min-max rescaled to the [0,1] interval
- Cell types occurring in only one dataset are dropped from the output rather than scored
- Output file cilisi_scores.csv exists with one row per shared cell type

## Limitations

- CiLISI and similar metrics require cell type annotations, which may be missing or noisy
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As with LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells
- Batch mixing metrics that are more robust to lopsided cell proportions are still lacking

## Evidence

- [other] CiLISI is a normalized batch LISI computed on a per-cell type basis: it tends to 1 when cells of the same type are well mixed across batches and becomes zero: [evidence span withheld — anchor status: not_found]
- [readme] A collection of metrics to evaluate scRNA-seq data integration quality, including LISI, per-cell type LISI (CiLISI), and silhouette average width (ASW).: [evidence span withheld — anchor status: external_doc]
- [discussion] One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations.: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset.: "Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset."
- [readme] It includes our new integration metric CiLISI we computed with our [scIngrationMetrics](https://github.com/carmonalab/scIntegrationMetrics) R package.: [evidence span withheld — anchor status: external_doc]
