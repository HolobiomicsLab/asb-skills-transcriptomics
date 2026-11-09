---
name: gaussian-graphical-model-inference-large-scale
description: Use when a log-normalized single-cell gene expression matrix with >5000
  cells needs partial-correlation (GGM) co-expression network inference, or when benchmarking
  SingleCellGGM against pcor-based methods like ppcor and SILGGM on an identical input
  matrix.
when_to_use_negative:
- The dataset is small-scale or has fewer than ~5000 cells; existing pcor-based methods
  (ppcor, SILGGM, scLink) were only tested on small-scale datasets and SingleCellGGM
  recommends large cell numbers
- The goal requires modalities beyond transcriptomes (e.g., ATAC-seq or single-cell
  proteomics); SingleCellGGM relies exclusively on transcriptome data
- The input matrix has not been log-normalized or low-expression genes have not been
  filtered, since the algorithm expects a log-normalized matrix with low-expression
  genes removed first
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: Primary MATLAB algorithm performing GGM-based pcor inference over ~20,000
    iterations from a log-normalized single-cell expression matrix
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: ppcor
  role: Baseline pcor-based method run on the identical MCA log-transformed matrix
    to infer MCA_ppcor partial-correlation network for benchmarking
- name: SILGGM
  role: Baseline large-scale GGM inference method run on the identical MCA matrix
    to infer MCA_SILGGM network for benchmarking
- name: ModuleEnrichmentAnalysis
  role: R pipeline performing GO enrichment analysis on each method's GEPs to count
    GO-enriched modules (p <= 1E-20 significance criterion)
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:7c586595918b523fd0560b645b93587a2fd29acc566822255018739d86a119c9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/SingleCellGGM
  tools:
  - ModuleEnrichmentAnalysis
  - SILGGM
  - SingleCellGGM
  - ppcor
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gaussian-graphical-model-inference-large-scale

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers large-scale single-cell gene co-expression networks via the graphical Gaussian model (GGM): SingleCellGGM calculates partial correlation coefficients (pcors) between gene pairs over ~20,000 iterations, and the same log-transformed matrix can be benchmarked against other pcor-based methods (ppcor, SILGGM) to identify gene expression programs (GEPs).

## When to use

Use when a log-normalized single-cell gene expression matrix with >5000 cells needs partial-correlation (GGM) co-expression network inference, or when benchmarking SingleCellGGM against pcor-based methods like ppcor and SILGGM on an identical input matrix.

## When NOT to use

- The dataset is small-scale or has fewer than ~5000 cells; existing pcor-based methods (ppcor, SILGGM, scLink) were only tested on small-scale datasets and SingleCellGGM recommends large cell numbers
- The goal requires modalities beyond transcriptomes (e.g., ATAC-seq or single-cell proteomics); SingleCellGGM relies exclusively on transcriptome data
- The input matrix has not been log-normalized or low-expression genes have not been filtered, since the algorithm expects a log-normalized matrix with low-expression genes removed first

## Inputs

- Log2-normalized single-cell gene expression matrix (double, cells in rows, genes in columns; e.g. 'Figure2-batch-removed.txt' from MCA)
- Ensembl gene ID list for matrix genes (e.g. 'data/MCA.ensembl.gene.ids.txt')
- Number of iterations for pcor sampling (typically 20000)
- Optional dataset name string

## Outputs

- SingleCellGGM object with pcor_all, coexpressed_cell_num, and SigEdges table (e.g. 127229 significant edges for MCA)
- Tab-delimited edge list 'mca.ggm.network.txt' for downstream network construction
- FDR table (pcor cutoff vs. FDR) from fdr_control
- Per-method benchmark_comparison.csv of GEP module counts, GO-enrichment counts, significance counts, and runtime/memory

## How to apply

Log-transform the expression matrix as log2(expression / sum(expression) * 10000 + 1) with cells in rows and genes in columns, keep only genes with Ensembl IDs, filter out genes expressed in fewer than 10 cells, and convert to sparse format. Run the MATLAB SingleCellGGM function with number_of_iterations = 20000 (or round(gene_number * (gene_number - 1) / 39980) so each gene pair is sampled on average 100 times), then estimate false discovery rates with fdr_control (permutation_fraction default 1) and tune edge significance with adjust_cutoff using a pcor cutoff (default 0.03) and a co-expressed-cell cutoff (default 10). For the MCA benchmark, run ppcor and SILGGM on the identical shared log-transformed matrix, record elapsed runtime and peak memory, and cluster each resulting network into gene modules with the Markov clustering (MCL) algorithm using parameters identical to those used for the SingleCellGGM network. Assess each method's GEPs via ModuleEnrichmentAnalysis GO enrichment and count significant GEPs under the paper's criteria: GEPs with >= 50 genes and GEPs with GO enrichment p <= 1E-20. Assemble module counts, enrichment counts, significance counts, and runtime/memory into a per-method comparison table (benchmark_comparison.csv).

## Related tools

- **SingleCellGGM** (Primary MATLAB algorithm performing GGM-based pcor inference over ~20,000 iterations from a log-normalized single-cell expression matrix) — https://github.com/MaShisongLab/SingleCellGGM
- **ppcor** (Baseline pcor-based method run on the identical MCA log-transformed matrix to infer MCA_ppcor partial-correlation network for benchmarking)
- **SILGGM** (Baseline large-scale GGM inference method run on the identical MCA matrix to infer MCA_SILGGM network for benchmarking)
- **ModuleEnrichmentAnalysis** (R pipeline performing GO enrichment analysis on each method's GEPs to count GO-enriched modules (p <= 1E-20 significance criterion)) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene, 'mca')
```

## Evaluation signals

- For the MCA dataset, the SingleCellGGM object reports gene_num = 24802 (genes with Ensembl IDs after filtering), samples_num = 61637, and SigEdges with 127229 rows
- Reproduced per-method GEP counts must match reported values: 224 total GEPs / 98 GO-enriched for ppcor, 258 / 175 for SILGGM, 137 / 123 for SingleCellGGM
- Significant GEP counts under the paper's criteria must match: >= 50 genes gives 64 (ppcor) / 1 (SILGGM) / 5 (SingleCellGGM); GO p <= 1E-20 gives 32 / 2 / 6
- The fdr_control output's fdr table (91 rows for MCA) lists per-pcor-cutoff SigEdgeNum and FDR values that should be populated and inspectable
- The runtime/memory rows of benchmark_comparison.csv must be populated for all three methods

## Limitations

- Existing pcor-based methods have only been tested on small-scale datasets and have not been widely adopted; SingleCellGGM's cell numbers should be large (>5000 recommended)
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics
- The algorithm requires MATLAB, and pcor sampling is stochastic (based on ~20,000 iterations), so exact pcor values depend on sampling while ensuring each gene pair is sampled on average 100 times

## Evidence

- [readme] SingleCellGGM takes a single-cell gene expression matrix as input and uses a process consisting of ~20,000 iterations to calculate partial correlation coefficients (pcors) between gene pairs.: [evidence span withheld — anchor status: external_doc]
- [readme] Expression matrix normalization and filtering steps: [evidence span withheld — anchor status: external_doc]
- [readme] Alternative iteration-count formula: [evidence span withheld — anchor status: not_found]
