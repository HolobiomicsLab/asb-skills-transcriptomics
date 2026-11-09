---
name: single-cell-gene-co-expression-network-inference
description: Use when a large single-cell count matrix (>5000 cells recommended) needs
  gene-level co-expression network construction and identification of co-expression
  modules (GEPs), especially where weak co-expression signals and dropouts defeat
  Pearson-correlation approaches.
when_to_use_negative:
- The dataset has few cells (<5000 recommended) or genes expressed in <10 cells that
  have not been filtered out.
- The goal requires integrating non-transcriptomic modalities (ATAC-seq, single-cell
  proteomics), which SingleCellGGM does not support.
- A quick clustering-free annotation of a novel dataset is needed and marker GEPs
  already exist — use CellGEP's cellgep/average_by_gep instead of re-inferring a network.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SingleCellGGM
  role: Core MATLAB algorithm computing partial correlations, co-expressed cell counts,
    permutation-based FDR, and significant edges for the co-expression network
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: MCL
  role: Graph-clustering algorithm used to partition the GGM network into gene co-expression
    modules (GEPs) with >=10 genes
- name: ModuleEnrichmentAnalysis
  role: R tool for GO/MP term enrichment of each identified GEP using Ensembl BioMart
    GO and MGI MP annotations
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:8e9d92face6946142ec51d2c52fde24dfce0f568105792a6706eb17299e87b9f
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
  - ModuleEnrichmentAnalysis
  - SingleCellGGM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-gene-co-expression-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer single-cell gene co-expression networks with SingleCellGGM, a MATLAB graphical Gaussian model (GGM) algorithm that computes partial correlations (pcors) between gene pairs using ~20,000 iterations and permutation-based FDR estimation, then clusters the network into gene expression programs (GEPs) with MCL.

## When to use

Use when a large single-cell count matrix (>5000 cells recommended) needs gene-level co-expression network construction and identification of co-expression modules (GEPs), especially where weak co-expression signals and dropouts defeat Pearson-correlation approaches.

## When NOT to use

- The dataset has few cells (<5000 recommended) or genes expressed in <10 cells that have not been filtered out.
- The goal requires integrating non-transcriptomic modalities (ATAC-seq, single-cell proteomics), which SingleCellGGM does not support.
- A quick clustering-free annotation of a novel dataset is needed and marker GEPs already exist — use CellGEP's cellgep/average_by_gep instead of re-inferring a network.

## Inputs

- log-normalized single-cell gene expression matrix (double format, cells in rows, genes in columns; e.g., 'Figure2-batch-removed.txt' from MCA)
- gene names list in Ensembl ID format (e.g., 'data/MCA.ensembl.gene.ids.txt')
- number of iterations for pcor calculation (typically 20000)
- optional dataset name string

## Outputs

- SingleCellGGM result object with pcor_all, coexpressed_cell_num, and SigEdges table (e.g., 127,229 gene pairs for MCA)
- network edge list file (e.g., 'mca.ggm.network.txt', tab-delimited)
- permutation-based FDR table from fdr_control (e.g., FDR = 4.17E-05 for MCA)
- GEP membership lists after MCL clustering (e.g., 137 modules for MCA; 'mca.marker.geps.txt')
- GO/MP enrichment summaries per GEP from ModuleEnrichmentAnalysis

## How to apply

Load a log-normalized single-cell expression matrix (cells in rows, genes in columns, double format) and filter to genes with IDs and expression in >=10 cells. Run SingleCellGGM(expression_matrix, 20000, gene_names, dataset_name), which iteratively samples gene pairs (~20,000 iterations, or round(gene_number*(gene_number-1)/39980)) to compute pairwise partial correlations and co-expressed cell counts. Estimate the false discovery rate with fdr_control using permutation of genes, then apply edge cutoffs pcor >= 0.03 and co-expression in >= 10 cells via adjust_cutoff to build the GGM network. Cluster the resulting network with MCL and retain modules with >=10 genes as GEPs; optionally run GO/MP enrichment via ModuleEnrichmentAnalysis on Ensembl gene IDs. Validation benchmarks from the paper: MCA yielded 127,229 gene pairs at FDR 4.17E-05 with 137 GEPs, and the Mouse Brain dataset (292,495 cells) yielded 73,786 gene pairs at FDR ~0 with 158 GEPs (B1–B158).

## Related tools

- **SingleCellGGM** (Core MATLAB algorithm computing partial correlations, co-expressed cell counts, permutation-based FDR, and significant edges for the co-expression network) — https://github.com/MaShisongLab/SingleCellGGM
- **MCL** (Graph-clustering algorithm used to partition the GGM network into gene co-expression modules (GEPs) with >=10 genes)
- **ModuleEnrichmentAnalysis** (R tool for GO/MP term enrichment of each identified GEP using Ensembl BioMart GO and MGI MP annotations) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene, 'mca'); fdr = fdr_control(expression_matrix, ggm); ggm_adjusted = adjust_cutoff(ggm, 0.03, 10);
```

## Evaluation signals

- MCA benchmark: SigEdges table contains ~127,229 gene pairs and fdr_control reports FDR = 4.17E-05 at pcor >= 0.03 / >= 10 co-expressed cells.
- Mouse Brain benchmark: ~73,786 gene pairs at permutation-test FDR ~0, and MCL yields 158 GEPs (B1–B158) of which 135 have GO-enriched terms (p <= 0.05).
- SigEdges columns (GeneA, GeneB, Pcor, SamplingTime, r, Cell_num_A, Cell_num_B, Cell_num_coexpressed, Dataset) are populated and gene IDs match Ensembl format (ENSMUSG...).
- 'mca.marker.geps.txt' exists with genes in Ensembl ID format and all modules contain >= 10 genes.
- ggm object properties (gene_num = 24802, RoundNumber = 20000, samples_num = 61637) reflect expected matrix dimensions and iteration count.

## Limitations

- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics.
- Cell-type label transfer is constrained by the limited cell-type coverage of the source dataset; e.g., annotating TMS with MCA GEPs leaves some cells as 'unknown', and Mouse Brain annotation would be compromised by absent high-resolution neuronal cell types in MCA.
- Enriched GO and MP terms for the Mouse Brain dataset were less compelling due to lack of granularity in brain-related GO/MP gene annotations; activity and morphogenesis GEPs have been largely overlooked by previous studies.
- The algorithm requires MATLAB and a large cell number (>5000 recommended); low-expression genes must be filtered out beforehand.

## Evidence

- [readme] SingleCellGGM takes a single-cell gene expression matrix as input and uses a process consisting of ~20,000 iterations to calculate partial correlation coefficients (pcors) between gene pairs.: [evidence span withheld — anchor status: not_found]
- [readme] MCA pipeline example invocation with 24,802 genes and cutoffs: [evidence span withheld — anchor status: external_doc]
- [readme] MCA results reproduced: 127,229 significant edges: [evidence span withheld — anchor status: external_doc]
