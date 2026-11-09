---
name: cross-method-stability-scoring
description: Use when two HVG-selection or gene-classification methods (e.g. CPPLS-MLP
  and CCPLS) must be compared for selection stability on spatial/single-cell datasets
  with cell-type clusters, and per-cell-type Y HVG lists are available.
when_to_use_negative:
- When the goal is to classify whether individual genes participate in cell communication
  (the MLP classification step) rather than compare method stability — use the search_gene
  criterion instead.
- When HVG lists per cell type are not yet available for each method; the skill requires
  precomputed Y/N HVG assignments, not raw count matrices alone.
- When assessing network-construction accuracy of communication graphs (e.g. GPX1
  spatial expression validation) rather than HVG-selection stability.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CPPLS-MLP
  role: 'Method under test: constructs MIMO cell–cell communication networks and identifies
    communication-related Y HVGs whose inter-cell-type overlap is scored'
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: CCPLS
  role: Baseline communication-network/HVG-selection method against which CPPLS-MLP
    stability scores are compared
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: Supplies FindVariableFeatures() used to extract one of the four precomputed
    HVG lists feeding the stability comparison
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: pls
  role: R package providing the Constrained Partial Least Squares (CPPLS) regression
    backbone of the CPPLS-MLP pipeline being benchmarked
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:405969acedca40a22a201dde67018e64fb1dc723ab332c994ccfcd605f53515a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CCPLS
  - CPPLS-MLP
  - Python
  - R
  - Seurat
  - pls
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-method-stability-scoring

## Summary

A benchmarking procedure that quantifies the stability of communication-related HVG selection (e.g. CPPLS-MLP vs the CCPLS baseline) by computing the overlap rate of Y HVGs across cell types on seqFISH+ and Seq-Scope spatial datasets, producing a per-method stability score within [0,1].

## When to use

Use when two HVG-selection or gene-classification methods (e.g. CPPLS-MLP and CCPLS) must be compared for selection stability on spatial/single-cell datasets with cell-type clusters, and per-cell-type Y HVG lists are available.

## When NOT to use

- When the goal is to classify whether individual genes participate in cell communication (the MLP classification step) rather than compare method stability — use the search_gene criterion instead.
- When HVG lists per cell type are not yet available for each method; the skill requires precomputed Y/N HVG assignments, not raw count matrices alone.
- When assessing network-construction accuracy of communication graphs (e.g. GPX1 spatial expression validation) rather than HVG-selection stability.

## Inputs

- seqFISH+ mouse cortex spatial dataset (2023_01_seqFISH+.zip)
- Seq-Scope mouse colon spatial dataset (2023_01_Seq-Scope.zip)
- Precomputed HVG lists from compare_hvg.zip, extracted via FindVariableFeatures(), variance, scanpy and M3drop (16,000 HVGs total)
- GeneCards/GO communication labels per gene (strong- and weak-correlation label classes)
- Cell type cluster assignments per dataset

## Outputs

- Per-cell-type Y HVG percentages and inter-cell-type overlap rates under strong and weak correlation labels
- Aggregated stability score per method (CPPLS-MLP, CCPLS) per dataset
- ccpls_mlp_hvg_stability_comparison.csv with columns for method and stability score (2 method rows per dataset)

## How to apply

Load the seqFISH+ (2023_01_seqFISH+.zip) and Seq-Scope (2023_01_Seq-Scope.zip) datasets together with the precomputed HVG lists extracted by four methods (FindVariableFeatures(), variance, scanpy, M3drop; 16,000 HVGs total in compare_hvg.zip). Annotate genes with strong-correlation communication labels ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction') from GeneCards/GO, assigning 1 to labeled (strong_com) genes and 0 to unlabeled ones, and mark weak-correlation labels separately. Run CPPLS-MLP and CCPLS on both datasets to classify Y HVGs (communication-related) and N HVGs per cell type cluster; the article reports 354 and 345 communication-related genes on seqFISH+ and Seq-Scope, respectively. For each cell type, compute the percentage of Y HVGs among total obtained genes, then the overlap rate of Y HVGs between cell types as the stability/uniqueness score, separately under strong and weak correlation labels. Aggregate the per-cell-type overlap rates into one stability score per method (CPPLS-MLP, CCPLS) per dataset and write the benchmark table, expecting average overlap ratios around 0.31 (Seq-Scope) and 0.25 (seqFISH+).

## Related tools

- **CPPLS-MLP** (Method under test: constructs MIMO cell–cell communication networks and identifies communication-related Y HVGs whose inter-cell-type overlap is scored) — https://github.com/wuzhenao/CPPLS-MLP
- **CCPLS** (Baseline communication-network/HVG-selection method against which CPPLS-MLP stability scores are compared) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (Supplies FindVariableFeatures() used to extract one of the four precomputed HVG lists feeding the stability comparison) — https://github.com/wuzhenao/CPPLS-MLP
- **pls** (R package providing the Constrained Partial Least Squares (CPPLS) regression backbone of the CPPLS-MLP pipeline being benchmarked) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
run 2023_01_Seq-Scope.RMD
```

## Evaluation signals

- Schema check: ccpls_mlp_hvg_stability_comparison.csv contains method and stability-score fields.
- Range check: every stability score lies within [0,1], consistent with reported overlap rates (0.31 for Seq-Scope, 0.25 for seqFISH+).
- Row-count check: each dataset (seqFISH+, Seq-Scope) contributes exactly 2 method rows (CPPLS-MLP and CCPLS).
- Count sanity: CPPLS-MLP should yield roughly 354 communication-related genes on seqFISH+ and 345 on Seq-Scope before scoring.
- Label-partition check: Y/N HVG splits are computed separately under strong-correlation (label=1) and weak-correlation label classes.

## Limitations

- Only two benchmark datasets (seqFISH+ and Seq-Scope) are used for performance assessment, limiting generalizability of stability scores.
- Gene labeling relies on manually querying GeneCards/GO with no scripted or automated procedure, so label sets may not be reproducible bit-for-bit.
- No version numbers or accession IDs are stated for tools or datasets, and MLP implementation details (architecture, hyperparameters) are unspecified in the article.

## Evidence

- [methods] Average HVG overlap ratio was 0.31 in the Seq-Scope dataset and 0.25 in the seqFISH+ dataset: "the average overlap ratio of HVGs between these cell types was 0.31"
- [readme] HVG lists were extracted by four methods feeding the comparison: "we extracted the HVGs on the dataset based on four methods: FindVariableFeatures(), variance, scannpy, and M3drop"
- [abstract] Superiority of CPPLS-MLP over CCPLS validated: "we validated the superior performance of our method in identifying cell-type-specific HVGs"
- [methods] Communication-related gene counts per dataset: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
- [discussion] Benchmark limited to two datasets: "Despite the limited number of datasets, we chose these due to their representativeness"
