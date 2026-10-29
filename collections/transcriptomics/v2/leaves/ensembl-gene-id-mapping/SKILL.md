---
name: ensembl-gene-id-mapping
description: Use when a single-cell count matrix (e.g., the MCA 'Figure2-batch-removed.txt')
  carries gene symbols or non-Ensembl identifiers and the downstream tool requires
  Ensembl IDs, as with SingleCellGGM network construction or ModuleEnrichmentAnalysis
  GO/MP enrichment.
when_to_use_negative:
- The input matrix already carries Ensembl gene IDs (e.g., genes already matched by
  contains(gene,'ENSMUSG')) — no mapping step is needed
- The downstream analysis is tool-agnostic with respect to identifier type (e.g.,
  exploratory plotting in Seurat) and does not require Ensembl IDs
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: Consumes the Ensembl-ID-mapped expression matrix and gene name vector to construct
    the GGM co-expression network with edges labeled by ENSMUSG IDs
  repo: github.com/MaShisongLab/SingleCellGGM
- name: ModuleEnrichmentAnalysis
  role: Requires gene sets (cluster_file) in Ensembl gene IDs only, so the ID mapping
    enables GO/MP enrichment of SingleCellGGM-derived GEPs
  repo: github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:3ca1b6b97319b9d5a1ec41038a264d49888846e857b4d46ac3089d9091926324
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  tools:
  - MCL
  - ModuleEnrichmentAnalysis
  - SingleCellGGM
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ensembl-gene-id-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps gene names in a single-cell expression matrix to Ensembl gene IDs (e.g., ENSMUSG identifiers for mouse) so downstream tools such as SingleCellGGM and ModuleEnrichmentAnalysis can consume them, since these pipelines require or only support Ensembl gene IDs.

## When to use

Use when a single-cell count matrix (e.g., the MCA 'Figure2-batch-removed.txt') carries gene symbols or non-Ensembl identifiers and the downstream tool requires Ensembl IDs, as with SingleCellGGM network construction or ModuleEnrichmentAnalysis GO/MP enrichment.

## When NOT to use

- The input matrix already carries Ensembl gene IDs (e.g., genes already matched by contains(gene,'ENSMUSG')) — no mapping step is needed
- The downstream analysis is tool-agnostic with respect to identifier type (e.g., exploratory plotting in Seurat) and does not require Ensembl IDs

## Inputs

- Single-cell gene expression matrix with non-Ensembl gene names (e.g., 'Figure2-batch-removed.txt' from MCA with 25,133 genes)
- Ensembl gene ID mapping file aligned to matrix gene order (e.g., 'data/MCA.ensembl.gene.ids.txt')

## Outputs

- Expression matrix subset to columns with valid Ensembl gene IDs (24,802 genes for MCA)
- Gene name vector of Ensembl IDs (ENSMUSG*) passed to SingleCellGGM and used in outputs like 'mca.marker.geps.txt'

## How to apply

Obtain an Ensembl gene ID list matching the matrix genes, e.g., the 'data/MCA.ensembl.gene.ids.txt' file for the mouse MCA dataset. Subset the expression matrix columns to genes that carry valid Ensembl IDs using a pattern check such as contains(gene,'ENSMUSG') — in the MCA example, 24,802 of 25,133 genes in the matrix have Ensembl gene IDs and are retained. Keep the ID vector aligned column-wise with the matrix so gene names passed to SingleCellGGM(expression_matrix, 20000, gene, 'mca') are Ensembl IDs. Note that ModuleEnrichmentAnalysis explicitly supports 'Only Ensembl gene IDs', so the same mapping enables downstream GO and MP term enrichment of GEP modules. Outputs such as 'mca.marker.geps.txt' will then contain gene names in Ensembl ID format, which serves as a validation check.

## Related tools

- **SingleCellGGM** (Consumes the Ensembl-ID-mapped expression matrix and gene name vector to construct the GGM co-expression network with edges labeled by ENSMUSG IDs) — github.com/MaShisongLab/SingleCellGGM
- **ModuleEnrichmentAnalysis** (Requires gene sets (cluster_file) in Ensembl gene IDs only, so the ID mapping enables GO/MP enrichment of SingleCellGGM-derived GEPs) — github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
gene = readcell('data/MCA.ensembl.gene.ids.txt'); idx = contains(gene,'ENSMUSG'); expression_matrix = expression_matrix(:,idx); gene = gene(idx);
```

## Evaluation signals

- After mapping, the number of retained genes matches expectation (24,802 of 25,133 MCA genes have Ensembl gene IDs)
- All retained gene names match the Ensembl ID pattern (e.g., prefix 'ENSMUSG' for mouse)
- Downstream SingleCellGGM output ggm.gene_num equals 24,802 and SigEdges gene pairs are labeled with ENSMUSG identifiers
- The GEP file 'mca.marker.geps.txt' contains gene names in Ensembl ID format
- ModuleEnrichmentAnalysis accepts the cluster file without identifier errors, since only Ensembl gene IDs are supported

## Limitations

- Genes lacking Ensembl IDs are dropped from the analysis — for MCA, 331 of 25,133 genes cannot be mapped and are excluded from network construction
- The mapping depends on an external, dataset-specific ID file (e.g., 'MCA.ensembl.gene.ids.txt' from Ensembl BioMart), which must stay row/column-aligned with the expression matrix; misalignment silently corrupts gene labels

## Evidence

- [readme] ID pattern check: [evidence span withheld — anchor status: external_doc]
- [readme] Downstream Ensembl-only requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Output validation in Ensembl format: [evidence span withheld — anchor status: not_found]
- [readme] Mapped IDs passed to SingleCellGGM: [evidence span withheld — anchor status: external_doc]
