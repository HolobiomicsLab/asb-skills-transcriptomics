---
name: mcl-network-module-clustering
description: Use when a SingleCellGGM GGM network (e.g., MCA_GGM with 127,229 gene
  pairs or MouseBrain_GGM with 73,786 pairs) must be clustered into gene co-expression
  modules (GEPs) for enrichment analysis or cell-type annotation.
when_to_use_negative:
- The input is a raw single-cell count matrix rather than a constructed gene co-expression
  network — run SingleCellGGM first to compute partial correlations
- Modules smaller than 10 genes are needed — the retained GEPs are filtered to >=
  10 genes
- The goal is cell-level clustering rather than gene-level co-expression module detection
  — MCL here clusters gene-pair networks, not cells
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: MCL
  role: Graph-clustering algorithm that partitions the SingleCellGGM network edge
    list into gene co-expression modules (GEPs)
- name: SingleCellGGM
  role: Generates the GGM co-expression network (SigEdges with pcor >= 0.03, co-expressed
    in >= 10 cells) that MCL clusters
  repo: github.com/MaShisongLab/SingleCellGGM
- name: ModuleEnrichmentAnalysis
  role: Performs GO/MP enrichment on the MCL-derived GEPs to validate module functional
    coherence (e.g., 135/158 Mouse Brain GEPs enriched at p <= 0.05)
  repo: MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:f560e12ec36f3226be8a3715b1cd0d8c852d5d24fd93d8ecc11c7743d57d0476
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# mcl-network-module-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions a SingleCellGGM gene co-expression network (GGM GCN edge list) into gene co-expression modules using the MCL graph-clustering algorithm, retaining modules of >=10 genes as gene expression programs (GEPs) for downstream GO/MP enrichment and cell annotation.

## When to use

Use when a SingleCellGGM GGM network (e.g., MCA_GGM with 127,229 gene pairs or MouseBrain_GGM with 73,786 pairs) must be clustered into gene co-expression modules (GEPs) for enrichment analysis or cell-type annotation.

## When NOT to use

- The input is a raw single-cell count matrix rather than a constructed gene co-expression network — run SingleCellGGM first to compute partial correlations
- Modules smaller than 10 genes are needed — the retained GEPs are filtered to >= 10 genes
- The goal is cell-level clustering rather than gene-level co-expression module detection — MCL here clusters gene-pair networks, not cells

## Inputs

- SingleCellGGM network edge list (e.g., 'mca.ggm.network.txt' or MouseBrain_GGM edges) with gene pairs passing pcor >= 0.03 and co-expression in >= 10 cells
- Gene identifiers in Ensembl format (e.g., ENSMUSG IDs)
- SingleCellGGM result object (ggm) from which SigEdges are exported

## Outputs

- Gene co-expression modules (GEPs) with >= 10 genes each (137 GEPs for MCA; 158 GEPs B1–B158 for Mouse Brain)
- GEP membership file (e.g., 'mca.marker.geps.txt') mapping Ensembl gene IDs to GEP IDs
- Per-module GO/MP enrichment summaries (via ModuleEnrichmentAnalysis) confirming functional coherence

## How to apply

First construct the GGM network by running SingleCellGGM (~20,000 iterations) on a log-normalized single-cell expression matrix (cells in rows, genes in columns, low-expression genes filtered, e.g., genes expressed in <10 cells removed), retaining edges with pcor >= 0.03 co-expressed in >= 10 cells, and export the significant edge list (e.g., 'mca.ggm.network.txt'). Then run the MCL graph-clustering algorithm on this edge list to partition the network into modules. Retain only modules containing >= 10 genes and designate them as GEPs — on MCA this yields 137 GEPs; on the Mouse Brain dataset it yields 158 GEPs (B1–B158). Write the GEP gene lists (Ensembl IDs) to a file such as 'mca.marker.geps.txt', then validate modules via GO/MP enrichment with ModuleEnrichmentAnalysis, where well-formed modules show highly significant enrichment (135/158 Mouse Brain GEPs at p <= 0.05).

## Related tools

- **MCL** (Graph-clustering algorithm that partitions the SingleCellGGM network edge list into gene co-expression modules (GEPs))
- **SingleCellGGM** (Generates the GGM co-expression network (SigEdges with pcor >= 0.03, co-expressed in >= 10 cells) that MCL clusters) — github.com/MaShisongLab/SingleCellGGM
- **ModuleEnrichmentAnalysis** (Performs GO/MP enrichment on the MCL-derived GEPs to validate module functional coherence (e.g., 135/158 Mouse Brain GEPs enriched at p <= 0.05)) — MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
mcl mca.ggm.network.txt --abc -o mca.mcl.modules.txt  # cluster the 127,229-edge MCA_GGM network exported by SingleCellGGM
```

## Evaluation signals

- MCA reproduction check: clustering the MCA_GGM network (127,229 gene pairs) with MCL and keeping modules >= 10 genes yields 137 GEPs
- Mouse Brain reproduction check: clustering MouseBrain_GGM (73,786 gene pairs) yields 158 GEPs (B1–B158)
- Enrichment sanity check: 135 of 158 Mouse Brain GEPs show significantly enriched GO terms (p <= 0.05) via ModuleEnrichmentAnalysis
- Schema check: 'mca.marker.geps.txt' exists and gene names are in Ensembl ID format (e.g., ENSMUSG*)
- Edge-filter check: the clustered network contains only gene pairs with pcor >= 0.03 co-expressed in >= 10 cells, with permutation-based FDR reported (e.g., 4.17E-05 for MCA)

## Limitations

- Activity and morphogenesis GEPs identified by this workflow have been largely overlooked by previous single-cell studies, so few external benchmarks exist for them
- Enriched GO/MP terms for Mouse Brain GEPs were less compelling, with limited insight into functional differences among neuronal cell subsets, possibly due to lack of granularity in brain-related GO/MP gene annotations
- MCL is applied only after edge filtering (pcor >= 0.03, >= 10 co-expressing cells); clustering unfiltered or Pearson-correlation-based networks from dropout-laden single-cell data is not supported by this workflow

## Evidence

- [results] Applying SingleCellGGM to the Mouse Brain dataset and clustering MouseBrain_GGM with MCL yielded 158 gene co-expression modules with ≥10 genes, designated GEPs B1–B158: [evidence span withheld — anchor status: not_found]
- [readme] Save all gene pairs to a file for gene co-expression network construction: [evidence span withheld — anchor status: external_doc]
