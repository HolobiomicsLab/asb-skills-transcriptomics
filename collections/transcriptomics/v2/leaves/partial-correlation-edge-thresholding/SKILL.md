---
name: partial-correlation-edge-thresholding
description: Use when building a GGM-based co-expression network from a large single-cell
  expression matrix where weak co-expression signals and dropouts make ordinary Pearson
  correlation unreliable.
when_to_use_negative:
- The dataset is a bulk transcriptome dataset — the modified GGM workflow for bulk
  data is rgsGGM, not SingleCellGGM
- The cell number is small (SingleCellGGM recommends >5000 cells), or low-expression
  genes have not been filtered out first
- The goal is integrative multimodal analysis (e.g. ATAC-seq or proteomics); SingleCellGGM
  relies exclusively on transcriptome data
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: MATLAB algorithm that computes single-cell partial correlations over ~20,000
    iterations and provides adjust_cutoff/fdr_control to threshold edges (pcor >=
    0.03, co-expression >= 10 cells) into a GGM network
  repo: MaShisongLab/SingleCellGGM
- name: ModuleEnrichmentAnalysis
  role: 'Downstream validation: GO/MP enrichment of the modules obtained from the
    thresholded network confirms edge quality (e.g. 135 of 158 Mouse Brain GEPs GO-enriched
    at p <= 0.05)'
  repo: MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:2a3159a7531b329a8b10171f13b2887c0009bfb73c5d45870970e68c6b6582c0
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
  - SingleCellGGM
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# partial-correlation-edge-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct a single-cell gene co-expression network by thresholding gene-pair partial correlation coefficients (pcors) computed by SingleCellGGM, retaining edges with pcor >= 0.03 between gene pairs co-expressed in at least 10 cells, and assessing edge significance with permutation-based FDR control.

## When to use

Use when building a GGM-based co-expression network from a large single-cell expression matrix where weak co-expression signals and dropouts make ordinary Pearson correlation unreliable.

## When NOT to use

- The dataset is a bulk transcriptome dataset — the modified GGM workflow for bulk data is rgsGGM, not SingleCellGGM
- The cell number is small (SingleCellGGM recommends >5000 cells), or low-expression genes have not been filtered out first
- The goal is integrative multimodal analysis (e.g. ATAC-seq or proteomics); SingleCellGGM relies exclusively on transcriptome data

## Inputs

- Log-normalized single-cell gene expression matrix (double/sparse, cells in rows, genes in columns), e.g. 10x Genomics Mouse Brain matrix of 24,080 genes x 292,495 cells
- Gene names/Ensembl IDs file (e.g. MCA.ensembl.gene.ids.txt)
- Number of iterations (typically 20000)

## Outputs

- SingleCellGGM result object with pcor_all, coexpressed_cell_num, and SigEdges table (GeneA, GeneB, Pcor, SamplingTime, r, Cell_num_A, Cell_num_B, Cell_num_coexpressed)
- Filtered network edge list (e.g. MouseBrain_GGM with 73,786 gene pairs at pcor >= 0.03, co-expressed in >= 10 cells)
- Permutation-test FDR table (fdr.fdr) reporting FDR per pcor cutoff

## How to apply

First log-normalize the count matrix (log2(CPM/10000 + 1)) with cells in rows and genes in columns, filter out genes expressed in fewer than 10 cells, and keep only genes with valid Ensembl IDs. Run SingleCellGGM with ~20,000 iterations (or round(gene_number * (gene_number - 1) / 39980) so each gene pair is sampled on average 100 times) to compute pcors, co-expressed-cell counts, and significant edges (SigEdges). Estimate the false discovery rate with fdr_control using the permutation test (default permutation_fraction of 1); the paper reports an FDR of approximately 0 at pcor >= 0.03 (e.g., the Mouse Brain network of 73,786 gene pairs). If needed, apply adjust_cutoff with pcor_cutoff=0.03 (default) and coexpressed_cell_cutoff=10 (default) to re-threshold the network, then write the resulting edge list (GeneA, GeneB, Pcor, etc.) to file for downstream MCL clustering.

## Related tools

- **SingleCellGGM** (MATLAB algorithm that computes single-cell partial correlations over ~20,000 iterations and provides adjust_cutoff/fdr_control to threshold edges (pcor >= 0.03, co-expression >= 10 cells) into a GGM network) — MaShisongLab/SingleCellGGM
- **ModuleEnrichmentAnalysis** (Downstream validation: GO/MP enrichment of the modules obtained from the thresholded network confirms edge quality (e.g. 135 of 158 Mouse Brain GEPs GO-enriched at p <= 0.05)) — MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene, 'mca'); fdr = fdr_control(expression_matrix, ggm); ggm_adjusted = adjust_cutoff(ggm, 0.03, 10);
```

## Evaluation signals

- The constructed Mouse Brain network reproduces the reported 73,786 gene pairs retained at pcor >= 0.03 with co-expression in >= 10 cells
- The permutation-test FDR (fdr.fdr table from fdr_control) is approximately 0 at the chosen pcor cutoff
- SigEdges table schema is intact: 9 columns including GeneA, GeneB, Pcor, SamplingTime, r, Cell_num_A, Cell_num_B, Cell_num_coexpressed, Dataset
- Genes expressed in fewer than 10 cells were removed before network construction, and only genes with Ensembl gene IDs were retained
- MCL clustering of the thresholded network yields 158 GEPs (B1–B158) with >= 10 genes, of which 135 show significant GO enrichment (p <= 0.05)

## Limitations

- FDR estimation via fdr_control is computationally tied to the permutation fraction and a limited set of pcor cutoffs (the reported fdr.fdr table spans 91 rows x 5 cutoff columns)
- Thresholding at pcor >= 0.03 discards weaker but potentially real co-expression signals; adjust_cutoff can only re-threshold to values like 0.02 after the full run
- Enriched GO/MP terms for the Mouse Brain GEPs were less compelling than for MCA, attributed to lack of granularity in brain-related GO and MP gene annotations

## Evidence

- [readme] SingleCellGGM then takes the gene pairs with pcor >= a selected cutoff value and co-expressed in >= a selected number of cells to construct a GGM gene co-expression network.: [evidence span withheld — anchor status: not_found]
- [readme] pcor_cutoff - cutoff for pcor to be adjusted (default is 0.03); coexpressed_cell_cutoff - cutoff for number of coexpressed cells to be adjusted (default is 10): [evidence span withheld — anchor status: not_found]
- [other] Retain gene pairs with pcor >= 0.03 that are co-expressed in at least 10 cells to build the MouseBrain_GGM network, and assess edge significance with SingleCellGGM's permutation test.: [evidence span withheld — anchor status: not_found]
- [other] Verify the constructed network reproduces the reported 73,786 gene pairs with a permutation-test FDR of approximately 0.: [evidence span withheld — anchor status: not_found]
- [readme] An alternative for number_of_iterations is calculated as round(gene_number * (gene_number - 1) / 39980), ensuring that each gene pair is sampled, on average, in 100 iterations.: [evidence span withheld — anchor status: external_doc]
