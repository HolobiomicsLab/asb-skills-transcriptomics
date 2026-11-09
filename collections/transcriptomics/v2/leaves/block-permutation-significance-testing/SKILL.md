---
name: block-permutation-significance-testing
description: Use when a TE-based (or other feature-based) cell clustering must be
  compared to a reference clustering (e.g., gene-expression-based Leiden/Louvain labels)
  and a raw ARI alone is insufficient — an empirical significance p-value is needed.
when_to_use_negative:
- No reference clustering exists to compare against (ARI requires two labelings of
  the same cells).
- Cells carry no meaningful local neighborhood structure, so block-wise label permutation
  adds no valid constraint over simple iid permutation.
- The task is TE quantification itself (read allocation to loci) rather than downstream
  significance assessment of clustering agreement — use MATES quantification modules
  instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MATES
  role: Provides locus-specific cell-by-TE count matrices (via the TE_quantifier module)
    whose clusterings are scored by ARI and tested with block permutation
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method whose clusterings are compared
    to MATES via ARI with block-permutation significance
- name: SoloTE
  role: Baseline single-cell TE quantification method whose clusterings are compared
    to MATES via ARI with block-permutation significance
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:ff8f9428599fc1422a8826940ef9952a58b86dbd444b03a44edcd77a9216b7a9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# block-permutation-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical validation procedure that assigns empirical p-values to clustering-agreement metrics (e.g., Adjusted Rand Index) by permuting cluster labels in blocks, preserving local neighborhood structure, as used to benchmark MATES versus scTE and SoloTE on Arabidopsis and Drosophila scRNA-seq TE clustering.

## When to use

Use when a TE-based (or other feature-based) cell clustering must be compared to a reference clustering (e.g., gene-expression-based Leiden/Louvain labels) and a raw ARI alone is insufficient — an empirical significance p-value is needed.

## When NOT to use

- No reference clustering exists to compare against (ARI requires two labelings of the same cells).
- Cells carry no meaningful local neighborhood structure, so block-wise label permutation adds no valid constraint over simple iid permutation.
- The task is TE quantification itself (read allocation to loci) rather than downstream significance assessment of clustering agreement — use MATES quantification modules instead.

## Inputs

- Gene-expression-based reference cluster labels per cell (e.g., Leiden/Louvain communities from the gene kNN graph)
- TE-based cluster labels per cell for each method and feature setting (MATES, scTE, SoloTE; TE-only and Gene+TE)
- Cell-by-TE count matrices from locus-specific TE quantification (MATES TE_quantifier module) for building kNN graphs

## Outputs

- ARI value per method/species/feature-setting comparison (e.g., MATES Arabidopsis TE-only 0.3514, Gene+TE 0.6668; SoloTE 0.3200/0.5916; scTE 0.3315/0.5775; Drosophila MATES 0.3188, scTE 0.3078)
- Empirical p-values from block-permutation tests for each ARI
- Consolidated results table (ari_comparison_results.csv) of ARI values and permutation p-values

## How to apply

For each method (MATES, scTE, SoloTE) and each feature setting (TE-only counts and combined gene-expression + TE counts), first cluster cells via Leiden/Louvain community detection on the kNN graph built from the cell-by-TE count matrices. Compute the Adjusted Rand Index (ARI) of the TE-based clustering against the gene-expression-based reference clustering (e.g., MATES Arabidopsis TE-only ARI 0.3514, Gene+TE ARI 0.6668; Drosophila TE-only ARI 0.3188). Rather than accepting the ARI at face value, assess its significance with a block-permutation test: permute cluster labels in blocks rather than iid across cells, so the local structure captured by the kNN graph is preserved, and compute an empirical p-value from the permutation null distribution. Finally, assemble all ARI values and permutation p-values into a results table (e.g., ari_comparison_results.csv) and check each reproduced ARI matches the reported figure within a small numerical tolerance.

## Related tools

- **MATES** (Provides locus-specific cell-by-TE count matrices (via the TE_quantifier module) whose clusterings are scored by ARI and tested with block permutation) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method whose clusterings are compared to MATES via ARI with block-permutation significance)
- **SoloTE** (Baseline single-cell TE quantification method whose clusterings are compared to MATES via ARI with block-permutation significance)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(gene_leiden_labels, mates_te_leiden_labels)  # then permute labels in blocks for empirical p-value
```

## Evaluation signals

- Reproduced ARI values for all three methods on both species agree with reported figures (MATES 0.3514/0.6668, SoloTE 0.3200/0.5916, scTE 0.3315/0.5775 in Arabidopsis; MATES 0.3188, scTE 0.3078 in Drosophila) within a small numerical tolerance.
- Empirical p-values are computed from block-permuted label distributions that preserve local structure, not from iid shuffling of all labels.
- ARI ordering is consistent with the reported benchmark: MATES Arabidopsis Gene+TE ARI 0.6668 > SoloTE 0.5916 > scTE 0.5775.
- Marker TEs identified for recovered populations include ATCopia66LTR and ATCopia41LTR in Arabidopsis and GYPSY12_LTR in Drosophila, consistent with gene-defined cell populations.
- All ARI values and permutation p-values are assembled into ari_comparison_results.csv with one row per method/species/feature-setting.

## Limitations

- MATES (and thus the downstream clustering comparison) typically requires more computational resources such as GPUs than scTE and SoloTE for quantifying TEs.
- Significance testing depends on the choice of block size/structure; incorrectly specified blocks can bias empirical p-values.
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage, which can affect the TE count matrices feeding the ARI comparisons.

## Evidence

- [other] Block-permutation significance for ARI cluster agreement: [evidence span withheld — anchor status: not_found]
- [other] ARI benchmark values against gene-expression reference clustering: [evidence span withheld — anchor status: not_found]
- [other] kNN graph clustering workflow feeding ARI computation: [evidence span withheld — anchor status: not_found]
