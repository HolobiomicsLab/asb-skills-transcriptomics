---
name: gene-signature-scoring
description: Use when you have a scRNA-seq count matrix plus a set of pre-defined
  gene signatures (.gmt) and want per-cell signature Z-scores and a statistic of whether
  signature variation tracks a cell similarity structure such as a phylogeny or KNN
  graph.
when_to_use_negative:
- You want data-driven heritable gene modules rather than testing pre-defined signatures
  — use PhyloVision's unguided Hotspot module identification instead
- The input lacks a cell-cell similarity structure (no phylogeny or trajectory/graph);
  Geary's C phylogenetic autocorrelation is undefined without a cell metric
- You only need phylogeny visualization with overlays — tools like iTOL or CeLaVi
  suffice and offer no joint expression/lineage analysis
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PhyloVision
  role: 'Runs the signature-centric analysis: computes VISION signature Z-scores and
    Geary''s C phylogenetic autocorrelation with num_neighbors = 30, and emits an
    interactive report'
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Provides the signature-scoring procedure for per-cell module/signature scores,
    the filterGenesFano gene filter, the front-end for the web report, and createGeneSignature
    for custom signatures
  repo: https://github.com/YosefLab/VISION
- name: R
  role: Implementation language; used for ape's collapse.singles tree preprocessing
    and hypergeometric significance tests
- name: Cassiopeia
  role: Source of the reconstructed single-cell phylogeny (Quinn et al. 2021) used
    as the latent space for the autocorrelation analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:4dcdc0b01df63e1a192bd1d36dd80ad85d5e48a19573cfabaef2f730afe5a43e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - VISION
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-signature-scoring

## Summary

Score user-defined gene signatures (e.g., MSigDB Hallmark, C2, C5/BP supplied as .gmt files) per cell and evaluate whether their expression variation agrees with a cell-cell similarity structure, such as a phylogeny, using VISION signature Z-scores and Geary's C autocorrelation. This is the signature-centric analysis that PhyloVision exposes for testing whether gene programs are heritable along a lineage tree.

## When to use

Use when you have a scRNA-seq count matrix plus a set of pre-defined gene signatures (.gmt) and want per-cell signature Z-scores and a statistic of whether signature variation tracks a cell similarity structure such as a phylogeny or KNN graph.

## When NOT to use

- You want data-driven heritable gene modules rather than testing pre-defined signatures — use PhyloVision's unguided Hotspot module identification instead
- The input lacks a cell-cell similarity structure (no phylogeny or trajectory/graph); Geary's C phylogenetic autocorrelation is undefined without a cell metric
- You only need phylogeny visualization with overlays — tools like iTOL or CeLaVi suffice and offer no joint expression/lineage analysis

## Inputs

- Raw UMI count matrix (cells x genes) for the clone (e.g., GEO GSE161363 / GSM4905334)
- Cassiopeia-reconstructed tumor phylogeny (Newick-style tree with cells as leaves) from Quinn et al. 2021
- Cell meta-data table matching the cells present in both expression matrix and lineage
- Gene signature collections in .gmt format (MSigDB Hallmark, C2, C5 BP) from https://www.gsea-msigdb.org/gsea/msigdb/

## Outputs

- Cell-level signature Z-scores for every signature and cell
- Phylogenetic autocorrelation results: C' statistic (= 1 - Geary's C, in [0,1], 1 = perfect autocorrelation) and Benjamini-Hochberg FDR per signature
- Table of C' and FDR values for all evaluated signatures (signature_autocorrelation_table.csv), with FDR < 0.05 signatures retained in the reported collapsable table
- Interactive web-based report with UMAP and tSNE30 projections of signature scores

## How to apply

Build a PhyloVision/VISION object supplying the phylogeny as the latent space, the .gmt signature collections (e.g., MSigDB Hallmark, C2, C5 BP), and cell meta-data; download signatures directly from the MSigDB database. In R, preprocess inputs first: collapse unifurcations with ape's collapse.singles, library-size-normalize raw UMI counts to the clone's median UMI number, and select informative genes with VISION's filterGenesFano (default parameters) for the projection_genes parameter. Run the signature-centric analysis with num_neighbors = 30 and projection_methods = c('UMAP', 'tSNE30'); this computes cell-level signature Z-scores using VISION's scoring procedure, then computes a phylogenetic autocorrelation statistic (Geary's C, reported as C' = 1 - C) on a KNN graph built from cophenetic distances. Assess significance with empirical p-values against randomly generated signatures, corrected with Benjamini-Hochberg, and retain signatures with FDR < 0.05. Write C' statistics and FDR values for all evaluated signatures to a table (e.g., signature_autocorrelation_table.csv).

## Related tools

- **PhyloVision** (Runs the signature-centric analysis: computes VISION signature Z-scores and Geary's C phylogenetic autocorrelation with num_neighbors = 30, and emits an interactive report) — https://github.com/YosefLab/VISION
- **VISION** (Provides the signature-scoring procedure for per-cell module/signature scores, the filterGenesFano gene filter, the front-end for the web report, and createGeneSignature for custom signatures) — https://github.com/YosefLab/VISION
- **R** (Implementation language; used for ape's collapse.singles tree preprocessing and hypergeometric significance tests)
- **Cassiopeia** (Source of the reconstructed single-cell phylogeny (Quinn et al. 2021) used as the latent space for the autocorrelation analysis)

## Examples

```
sig <- createGeneSignature(name = "Interesting Process", sigData = sigData)
```

## Evaluation signals

- Every evaluated signature in the output table has both a C' statistic constrained to [0,1] (C' = 1 - Geary's C, where 1 indicates perfect autocorrelation) and an FDR value
- Only signatures with FDR < 0.05 (Benjamini-Hochberg corrected empirical p-values against randomly generated signatures) are retained in the reported collapsable table
- Cell set in the analysis equals the intersection of cells present in both the expression matrix and the Cassiopeia lineage tree; unifurcations collapsed via collapse.singles
- Autocorrelation is computed on a KNN graph built from cophenetic distances with num_neighbors = 30, matching the published parameterization
- Signatures trace back to concrete .gmt sources (MSigDB Hallmark, C2, C5 BP) rather than ad hoc gene lists

## Limitations

- PhyloVision is implemented in R, which may create interoperability challenges for using separate Python software such as Cassiopeia
- Hypotheses generated from the metastasis dataset analysis lack experimental validation
- Informative-gene selection uses VISION's filterGenesFano with default parameters; deviating from the clone-median library-size normalization or KNN size can change signature Z-scores and Geary's C values

## Evidence

- [results] An autocorrelation statistic will be then computed by evaluating the consistency of the signature scores between nearby cells on the phylogeny, using Geary's C statistic: [evidence span withheld — anchor status: not_found]
- [methods] It then assesses the agreement between the signature scores and the phylogeny with a phylogenetic autocorrelation statistic: "It then assesses the agreement between the signature scores and the phylogeny with a phylogenetic autocorrelation statistic"
- [methods] Informative genes were found using the filterGenesFano function in VISION with default parameters: "Informative genes were found using the filterGenesFano function in VISION with default parameters"
- [readme] VISION aids in the interpretation of single-cell RNA-seq (scRNA-seq) data by selecting for gene signatures which describe coordinated variation between cells: "VISION aids in the interpretation of single-cell RNA-seq (scRNA-seq) data by selecting for gene signatures which describe coordinated variation between cells."
- [methods] Then, we assess significance using a hypergeometric test with R: "Then, we assess significance using a hypergeometric test with R"
