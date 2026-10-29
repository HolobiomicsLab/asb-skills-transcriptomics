---
name: markov-clustering-coexpression-modules
description: Use when a weighted gene co-expression or GGM network (e.g., SingleCellGGM
  SigEdges from the MCA matrix) must be partitioned into gene modules/GEPs under the
  same MCL parameters across methods for fair comparison.
when_to_use_negative:
- The analysis goal is cell-type annotation or label transfer with pre-existing marker
  GEPs (use CellGEP's average_by_gep/cellgep instead of re-clustering a network)
- Comparing methods on datasets where each method's clustering parameters differ,
  making module counts non-comparable
- The network is a bulk-transcriptome rgsGGM network rather than a single-cell co-expression
  network under test
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SingleCellGGM
  role: Constructs the GGM gene co-expression network from the MCA single-cell expression
    matrix; its modules are the reference GEPs that the MCL parameters are matched
    to
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: ppcor
  role: Baseline pcor-based method whose MCA network is clustered with MCL to yield
    224 GEPs for benchmarking
- name: SILGGM
  role: Baseline GGM method whose MCA network is clustered with MCL to yield 258 GEPs
    for benchmarking
- name: ModuleEnrichmentAnalysis
  role: Performs GO enrichment analysis on the MCL-derived modules to count GO-enriched
    GEPs per method
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:d56774b39eda215537b6b6e0002848a5e131b1d72ce73a8cd4324135f0f30f4a
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# markov-clustering-coexpression-modules

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters gene co-expression networks (e.g., the SingleCellGGM GGM network and ppcor/SILGGM benchmark networks built from the MCA dataset) into gene modules via Markov clustering (MCL), treating the modules as gene expression programs (GEPs) that are subsequently scored for gene count and GO enrichment.

## When to use

Use when a weighted gene co-expression or GGM network (e.g., SingleCellGGM SigEdges from the MCA matrix) must be partitioned into gene modules/GEPs under the same MCL parameters across methods for fair comparison.

## When NOT to use

- The analysis goal is cell-type annotation or label transfer with pre-existing marker GEPs (use CellGEP's average_by_gep/cellgep instead of re-clustering a network)
- Comparing methods on datasets where each method's clustering parameters differ, making module counts non-comparable
- The network is a bulk-transcriptome rgsGGM network rather than a single-cell co-expression network under test

## Inputs

- SingleCellGGM gene co-expression network (e.g., mca.ggm.network.txt with gene-pair pcor edges)
- ppcor partial-correlation co-expression network from the same log-transformed MCA matrix
- SILGGM Gaussian graphical model network from the same log-transformed MCA matrix
- MCL clustering parameters matched to the SingleCellGGM MCA reference run
- Ensembl gene ID background gene list for ModuleEnrichmentAnalysis

## Outputs

- Per-method GEP module assignments (e.g., 137 GEPs for SingleCellGGM, 224 for ppcor, 258 for SILGGM)
- GO-enrichment calls per GEP (98 GO-enriched GEPs for ppcor, 175 for SILGGM, 123 for SingleCellGGM)
- Significant GEP counts by criteria: >= 50 genes (64 ppcor / 1 SILGGM / 5 SingleCellGGM) and p <= 1E-20 (32 ppcor / 2 SILGGM / 6 SingleCellGGM)
- Per-method comparison table (benchmark_comparison.csv) with module, enrichment, significance, runtime, and memory figures

## How to apply

Build each co-expression network from the shared log-transformed MCA gene expression matrix (log2(TPM/10^4+1), genes expressed in >= 10 cells): run SingleCellGGM in MATLAB (~20,000 iterations), ppcor, and SILGGM on the identical matrix. Cluster every network with the Markov clustering (MCL) algorithm using parameters identical to those used for the SingleCellGGM MCA network, which yields 224 GEPs for ppcor and 258 GEPs for SILGGM. Then run the ModuleEnrichmentAnalysis GO pipeline on each method's modules with the appropriate Ensembl background gene list, and score significance under the paper's criteria: GEPs with >= 50 genes and GEPs with GO enrichment p <= 1E-20. The rationale for identical MCL parameters is that module-count differences (e.g., 137 total GEPs for SingleCellGGM with 64 having >= 50 genes and 6 at p <= 1E-20) must reflect network topology, not clustering settings.

## Related tools

- **SingleCellGGM** (Constructs the GGM gene co-expression network from the MCA single-cell expression matrix; its modules are the reference GEPs that the MCL parameters are matched to) — https://github.com/MaShisongLab/SingleCellGGM
- **ppcor** (Baseline pcor-based method whose MCA network is clustered with MCL to yield 224 GEPs for benchmarking)
- **SILGGM** (Baseline GGM method whose MCA network is clustered with MCL to yield 258 GEPs for benchmarking)
- **ModuleEnrichmentAnalysis** (Performs GO enrichment analysis on the MCL-derived modules to count GO-enriched GEPs per method) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
ggm = SingleCellGGM( expression_matrix, 20000, gene, 'mca'); writetable(ggm.SigEdges(:,1:3),'mca.ggm.network.txt','Delimiter','tab')
```

## Evaluation signals

- Total module counts match the reported values: 224 GEPs for ppcor, 258 for SILGGM, and 137/123-gene-enriched figures for SingleCellGGM under identical MCL parameters
- Significant GEP counts reproduce the paper's thresholds: >= 50 genes gives 64 (ppcor) / 1 (SILGGM) / 5 (SingleCellGGM); GO enrichment p <= 1E-20 gives 32 / 2 / 6
- GO-enriched GEP counts match: 98 for ppcor, 175 for SILGGM, 123 for SingleCellGGM from the ModuleEnrichmentAnalysis pipeline
- All three networks were clustered from the identical log-transformed MCA matrix with the same MCL parameters
- benchmark_comparison.csv contains populated runtime and memory rows for all three methods

## Limitations

- Module-count comparability depends on strictly identical MCL parameters; differing settings confound the method comparison
- Some modules (e.g., activity and morphogenesis GEPs) lack strong GO/MP enrichment due to lack of granularity in brain-related GO and MP gene annotations
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics

## Evidence

- [readme] network construction from MCA matrix: [evidence span withheld — anchor status: external_doc]
- [readme] GO enrichment of modules: [evidence span withheld — anchor status: external_doc]
