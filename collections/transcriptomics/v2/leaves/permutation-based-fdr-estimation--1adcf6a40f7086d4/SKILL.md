---
name: permutation-based-fdr-estimation
description: Use when a SingleCellGGM GGM object has been constructed from a single-cell
  count matrix and edges need significance assessment or false discovery rate control
  before network clustering.
when_to_use_negative:
- The expression matrix has not yet been processed by SingleCellGGM — fdr_control
  requires the fitted ggm object as input
- Genes have not been filtered for low expression (e.g., expressed in fewer than 10
  cells) or lack Ensembl gene IDs, degrading permutation-based statistics
- The dataset is small-scale (fewer than ~5000 cells) — SingleCellGGM itself recommends
  large cell numbers
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: MATLAB algorithm whose fdr_control.m function implements the permutation-based
    FDR estimation and adjust_cutoff.m applies the chosen pcor cutoff
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: ModuleEnrichmentAnalysis
  role: 'Downstream: after FDR-controlled edges define GEP modules, GO/MP enrichment
    is run on them (135 of 158 Mouse Brain GEPs enriched at p <= 0.05)'
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:9ea333ad8ab7866b14bb983339fd43773651ca3971f8b03a657471e0d939a035
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# permutation-based-fdr-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses the significance of gene co-expression edges in SingleCellGGM networks by permuting the expression matrix and recomputing partial-correlation statistics, yielding an empirical FDR table across pcor cutoffs (e.g., Mouse Brain network with ~0 FDR at pcor >= 0.03).

## When to use

Use when a SingleCellGGM GGM object has been constructed from a single-cell count matrix and edges need significance assessment or false discovery rate control before network clustering.

## When NOT to use

- The expression matrix has not yet been processed by SingleCellGGM — fdr_control requires the fitted ggm object as input
- Genes have not been filtered for low expression (e.g., expressed in fewer than 10 cells) or lack Ensembl gene IDs, degrading permutation-based statistics
- The dataset is small-scale (fewer than ~5000 cells) — SingleCellGGM itself recommends large cell numbers

## Inputs

- Log-normalized single-cell gene expression matrix (double/sparse format, cells in rows, genes in columns)
- SingleCellGGM result object (ggm) containing pcor_all, coexpressed_cell_num, and SigEdges
- Optional: permutation_fraction parameter (default 1)

## Outputs

- fdr_control result object with an fdr table (Pcor cutoffs vs. SigEdgeNum, FDR, SigPermutatedEdgeNum, SigPermutatedEdgeProportion)
- Adjusted ggm object (via adjust_cutoff) with edge significance recomputed at the chosen pcor and co-expressed-cell cutoffs

## How to apply

After running SingleCellGGM (typically 20,000 iterations for pcor calculation) on a log-normalized, sparse expression matrix, call fdr_control(expression_matrix, ggm, permutation_fraction) to permute a fraction of genes (default 1, i.e., all) and re-estimate edge statistics. The function returns a table of FDR values stratified by Pcor cutoff (e.g., a 91x5 table with columns Pcor, SigEdgeNum, FDR, SigPermutatedEdgeNum, SigPermutatedEdgeProportion). Inspect fdr.fdr to choose a pcor cutoff that yields acceptable FDR; in the Mouse Brain analysis (24,080 genes x 292,495 cells), the pcor >= 0.03 and co-expressed-in-≥10-cells cutoffs produced 73,786 gene pairs with a permutation-test FDR of approximately 0. If a stricter or looser cutoff is desired post-hoc, use adjust_cutoff(ggm, pcor_cutoff, coexpressed_cell_cutoff) rather than re-running SingleCellGGM.

## Related tools

- **SingleCellGGM** (MATLAB algorithm whose fdr_control.m function implements the permutation-based FDR estimation and adjust_cutoff.m applies the chosen pcor cutoff) — https://github.com/MaShisongLab/SingleCellGGM
- **ModuleEnrichmentAnalysis** (Downstream: after FDR-controlled edges define GEP modules, GO/MP enrichment is run on them (135 of 158 Mouse Brain GEPs enriched at p <= 0.05)) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
fdr = fdr_control(expression_matrix, ggm); fdr.fdr; ggm_adjusted = adjust_cutoff(ggm, 0.03, 10);
```

## Evaluation signals

- fdr.fdr is a valid table with expected columns (Pcor, SigEdgeNum, FDR, SigPermutatedEdgeNum, SigPermutatedEdgeProportion)
- At pcor >= 0.03 and co-expressed-cell cutoff >= 10, the Mouse Brain network reproduces ~73,786 gene pairs with permutation-test FDR approximately 0
- SigPermutatedEdgeProportion at the chosen cutoff is near zero, confirming permuted data yields far fewer significant edges than real data
- adjust_cutoff applied to the same ggm object changes SigEdges consistently with the new cutoff (e.g., 0.02 instead of 0.03) without requiring re-run of SingleCellGGM

## Limitations

- FDR estimation relies on gene permutation within the same dataset; it assumes permuted gene pairs represent the null and may not capture all biases in highly structured single-cell data
- Default cutoffs (pcor >= 0.03, co-expressed in >= 10 cells) are dataset-tuned; thresholds may need adjustment for datasets with different cell numbers or depth
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities, so FDR significance reflects transcriptomic co-expression only

## Evidence

- [readme] fdr_control usage and permutation fraction: [evidence span withheld — anchor status: external_doc]
- [readme] FDR control purpose: [evidence span withheld — anchor status: external_doc]
- [readme] adjust_cutoff defaults: [evidence span withheld — anchor status: external_doc]
- [other] Mouse Brain expected result: [evidence span withheld — anchor status: not_found]
