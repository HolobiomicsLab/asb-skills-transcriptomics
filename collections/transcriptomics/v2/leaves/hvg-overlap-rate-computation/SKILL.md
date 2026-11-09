---
name: hvg-overlap-rate-computation
description: Use when comparing two or more HVG-selection methods for cell-cell communication
  network construction and you need a single stability score per method per dataset
  from overlapping gene sets across cell type clusters.
when_to_use_negative:
- You only have one cell type (no inter-cell-type overlap rate can be computed)
- Genes lack GeneCards/GO communication-related labels, so Y vs N classification of
  HVGs is impossible
- You need to compare non-HVG gene sets (e.g. differential expression or pathway gene
  lists) rather than communication-related HVGs
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CPPLS-MLP
  role: Produces the Y HVG sets (communication-related HVGs) per cell type whose inter-cell-type
    overlap rate is computed as the stability score
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: CCPLS
  role: Baseline communication-network method whose Y HVG sets are compared against
    CPPLS-MLP's for stability
- name: Seurat
  role: Source of the FindVariableFeatures() function used to extract one of the four
    HVG lists feeding the overlap computation
- name: R
  role: Environment for running CPPLS-MLP/CCPLS (R version <= 4.2.3) and computing
    overlap statistics
- name: Python
  role: Environment (>= 3.7) for the scannpy-based HVG extraction and notebook-based
    gene classification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:376bdcccb6c9d17e34331a2be19ff691459fe6ac558c013d70d92d3573a13298
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hvg-overlap-rate-computation

## Summary

Quantifies the stability of highly variable gene (HVG) selection across cell types by computing per-cell-type percentages of communication-related ('Y') HVGs and their inter-cell-type overlap rates, benchmarking CPPLS-MLP against CCPLS on seqFISH+ and Seq-Scope data.

## When to use

Use when comparing two or more HVG-selection methods for cell-cell communication network construction and you need a single stability score per method per dataset from overlapping gene sets across cell type clusters.

## When NOT to use

- You only have one cell type (no inter-cell-type overlap rate can be computed)
- Genes lack GeneCards/GO communication-related labels, so Y vs N classification of HVGs is impossible
- You need to compare non-HVG gene sets (e.g. differential expression or pathway gene lists) rather than communication-related HVGs

## Inputs

- Precomputed HVG lists from seqFISH+ (2023_01_seqFISH+.zip) and Seq-Scope (2023_01_Seq-Scope.zip) datasets, extracted via FindVariableFeatures(), variance, scannpy and M3drop (compare_hvg.zip)
- Gene label annotations from GeneCards/GO distinguishing strong-correlation labels ('receptor', 'ligand', 'cell–cell adhesion', 'intercellular interaction') from weak-correlation labels
- Per-cell-type Y HVG (communication-related) and N HVG (non-related) gene sets from CPPLS-MLP and CCPLS runs

## Outputs

- ccpls_mlp_hvg_stability_comparison.csv with method and stability-score columns, 2 method rows per dataset
- Per-cell-type percentages of Y HVGs and per-cell-type overlap rates, separately for strong and weak correlation labels

## How to apply

Load the HVG lists extracted by four methods (FindVariableFeatures(), variance, scannpy, M3drop; 16,000 HVGs total) from the seqFISH+ and Seq-Scope datasets in compare_hvg.zip. Annotate each gene with value 1 if it carries strong-correlation labels ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction') from GeneCards/GO, and 0 otherwise; mark weak-correlation labels separately. Run CPPLS-MLP and the CCPLS baseline per cell type cluster to obtain Y HVGs (communication-related) and N HVGs (non-related). For each cell type, compute the percentage of Y HVGs over total genes, then compute the overlap rate of Y HVGs between cell types under strong and weak labels; expected average overlap is ~0.25 (seqFISH+) and ~0.31 (Seq-Scope). Aggregate per-cell-type overlap rates into one stability score per method per dataset and write to ccpls_mlp_hvg_stability_comparison.csv with 2 method rows per dataset.

## Related tools

- **CPPLS-MLP** (Produces the Y HVG sets (communication-related HVGs) per cell type whose inter-cell-type overlap rate is computed as the stability score) — https://github.com/wuzhenao/CPPLS-MLP
- **CCPLS** (Baseline communication-network method whose Y HVG sets are compared against CPPLS-MLP's for stability)
- **Seurat** (Source of the FindVariableFeatures() function used to extract one of the four HVG lists feeding the overlap computation)
- **R** (Environment for running CPPLS-MLP/CCPLS (R version <= 4.2.3) and computing overlap statistics)
- **Python** (Environment (>= 3.7) for the scannpy-based HVG extraction and notebook-based gene classification)

## Evaluation signals

- ccpls_mlp_hvg_stability_comparison.csv contains both the method and stability-score fields
- All stability scores lie within [0,1], consistent with reported average overlap rates of 0.31 for Seq-Scope and 0.25 for seqFISH+
- Each dataset (seqFISH+, Seq-Scope) contributes exactly 2 method rows (one for CPPLS-MLP, one for CCPLS), i.e. 4 rows total
- CPPLS-MLP stability score exceeds the CCPLS baseline score on both datasets, matching the reported finding of greater stability in HVG selection
- Overlap rates are computed separately under strong and weak correlation labels and the strong-label rates match the reported magnitude (~0.25–0.31)

## Limitations

- Only two benchmark datasets (seqFISH+ mouse cortex and Seq-Scope mouse colon) are available, limiting generalizability
- There is no standardized classification criterion to determine whether a gene is associated with cell communication; the Y/N labeling relies on manually queried GeneCards/GO labels with no scripted or automated procedure
- HVG extraction itself depends on the chosen method (FindVariableFeatures(), variance, scannpy, M3drop), and tool/dataset version numbers and dataset accession IDs are not provided in the article

## Evidence

- [methods] Average HVG overlap ratio was 0.31 in the Seq-Scope dataset and 0.25 in the seqFISH+ dataset: "the average overlap ratio of HVGs between these cell types was 0.31"
- [abstract] CPPLS-MLP validated superior performance compared to CCPLS in identifying cell-type-specific HVGs: "we validated the superior performance of our method in identifying cell-type-specific HVGs"
- [readme] README: HVG lists extracted with four methods feed the comparison: "we extracted the HVGs on the dataset based on four methods: FindVariableFeatures(), variance, scannpy, and M3drop"
- [discussion] Database labels used to distinguish communication-related HVGs: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [methods] Per-cell-type counts of communication-related genes ground the overlap computation: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
