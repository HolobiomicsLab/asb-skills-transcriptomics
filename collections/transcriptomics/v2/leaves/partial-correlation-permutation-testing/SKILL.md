---
name: partial-correlation-permutation-testing
description: Use when building a gene co-expression network (GGM) from a large single-cell
  expression matrix (>5000 cells recommended) and you need pcor estimates with permutation-based
  FDR control to select significant gene pairs.
when_to_use_negative:
- Datasets with fewer than ~5000 cells — the README recommends large cell numbers
  and the method was designed for large-scale transcriptomes
- You need multi-modal integration (ATAC-seq, single-cell proteomics) — SingleCellGGM
  relies exclusively on transcriptome data
- Small-scale datasets where simpler co-expression methods suffice — prior pcor methods
  like SILGGM and scLink were only tested on small datasets, whereas this skill targets
  large datasets such as MCA (61,637 cells) and Mouse Brain (292,495 cells)
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: MATLAB algorithm that computes pairwise partial correlation coefficients over
    ~20,000 iterations and co-expressed cell counts for GCN construction
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: fdr_control
  role: Companion function estimating the gene-pair false discovery rate from permutation-based
    edge statistics
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: adjust_cutoff
  role: Companion function adjusting pcor cutoff (default 0.03) and co-expressed cell
    cutoff (default 10) to define network edges
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:467bba076482167dcbc6b715c6b32700ec215ac6097e5e65d4694027146d72ee
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
  - MCL
  - SingleCellGGM
  - adjust_cutoff
  - fdr_control
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# partial-correlation-permutation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical inference of gene-gene partial correlations (pcors) from single-cell transcriptomes via SingleCellGGM, which samples gene pairs over ~20,000 iterations and uses permutation-based statistics to control the gene-pair false discovery rate before network construction.

## When to use

Use when building a gene co-expression network (GGM) from a large single-cell expression matrix (>5000 cells recommended) and you need pcor estimates with permutation-based FDR control to select significant gene pairs.

## When NOT to use

- Datasets with fewer than ~5000 cells — the README recommends large cell numbers and the method was designed for large-scale transcriptomes
- You need multi-modal integration (ATAC-seq, single-cell proteomics) — SingleCellGGM relies exclusively on transcriptome data
- Small-scale datasets where simpler co-expression methods suffice — prior pcor methods like SILGGM and scLink were only tested on small datasets, whereas this skill targets large datasets such as MCA (61,637 cells) and Mouse Brain (292,495 cells)

## Inputs

- Log-normalized single-cell gene expression matrix (double, cells in rows, genes in columns), e.g. MCA 'Figure2-batch-removed.txt'
- Gene names list (Ensembl gene IDs, e.g. ENSMUSG identifiers) in file 'data/MCA.ensembl.gene.ids.txt'
- Number of iterations for pcor sampling (typically 20000)
- Optional dataset name string

## Outputs

- SingleCellGGM object with pcor_all, pcor_sampling_num, coexpressed_cell_num, and SigEdges table (e.g. 127,229 gene pairs for MCA)
- Gene-pair network table (e.g. 'mca.ggm.network.txt' with GeneA, GeneB, Pcor columns)
- fdr_control object with fdr table reporting FDR at different pcor cutoffs (MCA: FDR = 4.17E-05 at pcor >= 0.03)

## How to apply

Take a log-normalized expression matrix (cells in rows, genes in columns, double format; log2(TP10K+1)) and run SingleCellGGM(expression_matrix, 20000, gene_names, dataset_name), which samples each gene pair on average 100 times over 20,000 iterations to compute partial correlations and co-expressed cell counts. Filter low-expression genes first (expressed in <10 cells removed; in the MCA example, 24,802 of 25,133 genes with Ensembl IDs were retained). Then call fdr_control(expression_matrix, ggm) to estimate the gene-pair FDR from permuted data, and optionally adjust_cutoff(ggm, 0.03, 10) to retain edges with pcor >= 0.03 co-expressed in >= 10 cells. An alternative iteration count is round(gene_number * (gene_number - 1) / 39980). Rationale: single-cell co-expression signals are weak and dropout obscures Pearson correlations, so pcor with permutation-based inference is used to define functional gene relationships.

## Related tools

- **SingleCellGGM** (MATLAB algorithm that computes pairwise partial correlation coefficients over ~20,000 iterations and co-expressed cell counts for GCN construction) — https://github.com/MaShisongLab/SingleCellGGM
- **fdr_control** (Companion function estimating the gene-pair false discovery rate from permutation-based edge statistics) — https://github.com/MaShisongLab/SingleCellGGM
- **adjust_cutoff** (Companion function adjusting pcor cutoff (default 0.03) and co-expressed cell cutoff (default 10) to define network edges) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene, 'mca'); fdr = fdr_control(expression_matrix, ggm); ggm_adjusted = adjust_cutoff(ggm, 0.03, 10);
```

## Evaluation signals

- MCA benchmark: SigEdges table contains 127,229 gene pairs after cutoffs pcor >= 0.03 and co-expression in >= 10 cells
- fdr.fdr table reports an FDR of 4.17E-05 for the retained edges on the MCA dataset
- ggm object properties match expectations: gene_num = 24802, samples_num = 61637, RoundNumber = 20000
- Network file 'mca.ggm.network.txt' is written with gene names in Ensembl ID format (ENSMUSG...)
- Sparse-matrix conversion and gene filtering (expression in >= 10 cells) complete without dimension mismatch between gene list and matrix columns

## Limitations

- Exclusive reliance on transcriptome datasets for GCN analysis, neglecting other single-cell modalities like single-cell ATAC-seq and proteomics
- Requires large cell numbers (>5000 recommended); low-expression genes must be filtered out before running
- ~20,000 iterations of pairwise sampling are memory-intensive; the README instructs conversion to a sparse matrix to save memory

## Evidence

- [readme] The algorithm takes a log-normalized gene expression matrix, the number of iterations, the names of the genes, and optionally, the name of dataset as inputs.: [evidence span withheld — anchor status: external_doc]
- [readme] An alternative for number_of_iterations is calculated as round(gene_number * (gene_number - 1) / 39980): [evidence span withheld — anchor status: external_doc]
- [other] Run the SingleCellGGM algorithm to compute pairwise partial correlation coefficients (pcor) between genes with approximately 20,000 permutations for statistical inference.: [evidence span withheld — anchor status: not_found]
- [other] Retain network edges that pass the cutoffs pcor >= 0.03 and co-expression of the gene pair in >= 10 cells: [evidence span withheld — anchor status: not_found]
