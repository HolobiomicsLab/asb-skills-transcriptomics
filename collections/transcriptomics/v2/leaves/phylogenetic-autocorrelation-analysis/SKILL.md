---
name: phylogenetic-autocorrelation-analysis
description: Use when you have matched single-cell expression data and a reconstructed
  lineage tree and need to test whether user-defined gene signatures are heritable,
  i.e. associated with phylogenetic structure.
when_to_use_negative:
- You want de novo modules rather than testing user-defined signatures — use Hotspot-based
  module detection instead
- No lineage/phylogeny is available or the tree covers cells absent from the expression
  matrix — Geary's C on cophenetic distances requires matched cells on a tree
- You only need phylogeny visualization with annotation overlays (iTOL, CeLaVi) without
  joint expression-lineage statistics
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: PhyloVision
  role: Runs the signature-centric phylogenetic autocorrelation analysis (Geary's
    C on a cophenetic KNN graph) and emits the interactive report
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Front-end of the web report and source of filterGenesFano for informative-gene
    selection and signature-scoring procedures
  repo: https://github.com/YosefLab/VISION
- name: Cassiopeia
  role: Provides the reconstructed single-cell phylogenies used as the latent space
- name: R
  role: Implementation language; ape's collapse.singles is used to prune unifurcations
    before analysis
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:165cc2e8949bd9f233f7e2e2dfbb6b04bd54a0ed1b7c6a143b44d8f0d2d8151a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - VISION
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# phylogenetic-autocorrelation-analysis

## Summary

Quantifies whether gene signature scores vary consistently among nearby cells on a single-cell phylogeny by computing Geary's C phylogenetic autocorrelation statistics (reported as C' = 1 - C) on a KNN graph built from cophenetic distances, with Benjamini-Hochberg FDR against randomly generated signatures.

## When to use

Use when you have matched single-cell expression data and a reconstructed lineage tree and need to test whether user-defined gene signatures are heritable, i.e. associated with phylogenetic structure.

## When NOT to use

- You want de novo modules rather than testing user-defined signatures — use Hotspot-based module detection instead
- No lineage/phylogeny is available or the tree covers cells absent from the expression matrix — Geary's C on cophenetic distances requires matched cells on a tree
- You only need phylogeny visualization with annotation overlays (iTOL, CeLaVi) without joint expression-lineage statistics

## Inputs

- Raw UMI count matrix (cells x genes) for the clone (e.g. from NCBI GEO GSE161363 / GSM4905334)
- Cassiopeia-reconstructed single-cell phylogeny (Newick/tree object) covering the same cells
- Per-cell metadata table
- Gene signature collections in .gmt format (e.g. MSigDB Hallmark, C2, C5 (BP))

## Outputs

- signature_autocorrelation_table.csv with a C' statistic (in [0,1]) and FDR for every evaluated gene signature
- Collapsable report table of significant signatures (FDR < 0.05)
- Interactive web-based report with per-cell signature Z-scores and UMAP/tSNE projections

## How to apply

Download matched raw UMI counts and cell metadata (e.g. GEO series GSE161363, dataset GSM4905334 from Quinn et al. 2021) plus the Cassiopeia-reconstructed tumor phylogeny, keeping only cells present in both modalities. Preprocess in R: collapse unifurcations with ape's collapse.singles, library-size-normalize counts to the clone's median UMI count, and select informative genes with VISION's filterGenesFano (default parameters) for projection_genes. Build a PhyloVision object using the phylogeny as the latent space, supplying MSigDB Hallmark, C2, and C5 (BP) signatures as .gmt files plus cell metadata. Run the signature-centric analysis with num_neighbors = 30 and projection_methods = c('UMAP', 'tSNE30'); this computes per-cell signature Z-scores and evaluates Geary's C on a KNN graph built from cophenetic distances, reporting C' = 1 - C so that 1 indicates perfect autocorrelation. Empirical p-values are computed against randomly generated signatures and corrected with Benjamini-Hochberg; retain signatures with FDR < 0.05.

## Related tools

- **PhyloVision** (Runs the signature-centric phylogenetic autocorrelation analysis (Geary's C on a cophenetic KNN graph) and emits the interactive report) — https://github.com/YosefLab/VISION
- **VISION** (Front-end of the web report and source of filterGenesFano for informative-gene selection and signature-scoring procedures) — https://github.com/YosefLab/VISION
- **Cassiopeia** (Provides the reconstructed single-cell phylogenies used as the latent space)
- **R** (Implementation language; ape's collapse.singles is used to prune unifurcations before analysis)

## Examples

```
hs.create_knn_graph(weighted_graph=False, n_neighbors=30); hs_results = hs.compute_autocorrelations()
```

## Evaluation signals

- The output table contains a C' statistic and an FDR for every signature, with C' bounded in [0,1] where 1 indicates perfect autocorrelation
- Only signatures with FDR < 0.05 (Benjamini-Hochberg corrected empirical p-values against randomly generated signatures) are retained in the reported collapsable table
- Every cell appears exactly once in the analysis — the intersection of expression matrix and lineage after collapse.singles pruning
- The KNN graph is built from cophenetic distances with num_neighbors = 30, matching the workflow specification

## Limitations

- PhyloVision is implemented in R, which may provide challenges for utilizing separate software available in Python, like Cassiopeia
- Analyses of the metastasis dataset lack experimental validation to support the generated hypotheses
- The dynamic phylogeny viewer was only tested on trees of up to ~4,000 leaves

## Evidence

- [results] Geary's C computation: [evidence span withheld — anchor status: not_found]
- [methods] filterGenesFano preprocessing: "Informative genes were found using the filterGenesFano function in VISION with default parameters"
- [methods] Unifurcation collapsing: "collapsed using the collapse.singles function in the R package ape"
- [readme] Hotspot autocorrelation API: "To compute per-gene autocorrelations, run:

hs_results = hs.compute_autocorrelations()"
- [discussion] R/Python limitation: "our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python, like Cassiopeia"
