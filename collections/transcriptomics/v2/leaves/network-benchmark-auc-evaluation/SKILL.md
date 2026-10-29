---
name: network-benchmark-auc-evaluation
description: Use when a directed TF-target network (e.g. from scGATE, CellOracle,
  Loregic) must be quantitatively scored against a known ground-truth GRN to reproduce
  or report AUROC/AUPRC benchmark performance.
when_to_use_negative:
- No ground-truth GRN is available for the tissue/cell type (no deposited GRN and
  no qualifying ChIP-seq data — remember the <50 peaks and non-physiological-condition
  exclusions)
- The network output is an unranked binary edge list with no confidence scores (e.g.
  log10 Bayes Factor), since AUROC/AUPRC require ranked predictions
- The goal is to validate Boolean logic gate structure against curated relationships
  rather than edge-level network benchmarking; scGATE itself notes that directed networks
  do not convey combinatorial relationships among regulating factors
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Produces the Bayes-Factor-ranked logic-gate network whose directed TF-target
    edges are benchmarked by AUROC/AUPRC against the ground-truth GRN
  repo: https://github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Execution environment in which scGATE inference and the network-vs-ground-truth
    AUROC/AUPRC comparison are run
- name: ChIP-Atlas
  role: Source of mouse TF ChIP-seq data used to generate the ground-truth GRNs used
    in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:f2e3057563d9d3cd4689cc5979e61e2e3be5685cdeedee64985c955ba8538006
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  repo_url: https://github.com/CompBioIPM/scGATE
  tools:
  - ChIP-Atlas
  - R
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# network-benchmark-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark an inferred directed TF-gene regulatory network against a deposited ground-truth GRN by computing AUROC and AUPRC over Bayes-Factor-ranked edges, as done for scGATE on the mouse haematopoiesis, Tabula Muris, Cusanovich scATAC-seq, and Buenrostro human haematopoiesis datasets.

## When to use

Use when a directed TF-target network (e.g. from scGATE, CellOracle, Loregic) must be quantitatively scored against a known ground-truth GRN to reproduce or report AUROC/AUPRC benchmark performance.

## When NOT to use

- No ground-truth GRN is available for the tissue/cell type (no deposited GRN and no qualifying ChIP-seq data — remember the <50 peaks and non-physiological-condition exclusions)
- The network output is an unranked binary edge list with no confidence scores (e.g. log10 Bayes Factor), since AUROC/AUPRC require ranked predictions
- The goal is to validate Boolean logic gate structure against curated relationships rather than edge-level network benchmarking; scGATE itself notes that directed networks do not convey combinatorial relationships among regulating factors

## Inputs

- Directed TF-target network with edges scored/ranked by log10 Bayes Factor confidence (scGATE_logic output)
- Ground-truth GRN file (e.g. .csv or .parquet base GRN such as Buenrostro2018_base_GRN_dataframe.parquet or the deposited ground-truth GRN from Zenodo)
- Preprocessed scRNA-seq expression matrix with normalized counts in the (0,1) interval
- TF chromatin immunoprecipitation sequencing (ChIP-seq) peak data from ChIP-Atlas when constructing ground truth de novo

## Outputs

- AUROC and AUPRC scores quantifying agreement between the inferred network and the ground-truth GRN
- Ranked list of predicted logic gates and TF-target edges ordered by log10 Bayes Factor
- Benchmark comparison table against ground truth (e.g. recovering Fli1 = Gata1 AND NOT Klf1 for MegE differentiation)

## How to apply

First produce a ranked network: for scGATE, run scGATE_logic() (e.g. number_of_em_iterations=10, top_gates=1, run_mode='complex' on the (0,1)-rescaled expression matrix; top_gates=50 with run_mode='simple' when a panel has >10 TFs) and rank each candidate TF-target edge by the log10 Bayes Factor of the most significant gate that includes it. Next obtain the ground-truth GRN: either the deposited file (e.g. base_grn_mouse_blood_cell_differentiation_toggle_switch.csv from Zenodo 10.5281/zenodo.8353409) or one built from mouse TF ChIP-seq data in the ChIP-Atlas database, excluding datasets with fewer than 50 peaks and non-physiological conditions such as gene knockouts or adeno-associated virus treatment. Finally compare the inferred directed network to the ground truth and compute AUROC and AUPRC. A correct application reproduces the reported benchmark behaviour, e.g. identifying Fli1 = Gata1 AND NOT Klf1 as the top-confidence gate for the MegE Gata1/Fli1/Klf1 toggle switch.

## Related tools

- **scGATE** (Produces the Bayes-Factor-ranked logic-gate network whose directed TF-target edges are benchmarked by AUROC/AUPRC against the ground-truth GRN) — https://github.com/CompBioIPM/scGATE
- **R 4.1.3** (Execution environment in which scGATE inference and the network-vs-ground-truth AUROC/AUPRC comparison are run)
- **ChIP-Atlas** (Source of mouse TF ChIP-seq data used to generate the ground-truth GRNs used in the benchmark)

## Examples

```
library(scGATE); gates <- scGATE_logic(data = data, base_GRN = candidate_tf_target, number_of_em_iterations = 10, top_gates = 1, run_mode = "complex")
```

## Evaluation signals

- AUROC and AUPRC are computed by comparing the inferred directed TF-target network against the deposited ground-truth GRN from Zenodo 10.5281/zenodo.8353409
- Edges are ranked by the log10 Bayes Factor of the most significant gate including each candidate TF before computing AUROC/AUPRC
- For the mouse haematopoiesis MegE panel (Gata1, Fli1, Klf1, Spi1, Zfpm1, Tal1, Gata2), the correct run assigns the highest confidence to Fli1 = Gata1 AND NOT Klf1 among the 16 candidate gates
- Ground-truth construction follows the exclusion rule: ChIP-seq data with fewer than 50 peaks and non-physiological conditions (gene knockouts, AAV treatment) are removed
- Input expression matrix must be normalized and rescaled into the (0,1) interval before scoring, since library size normalization was applied upstream

## Limitations

- Ground-truth networks themselves miss interactions such as large protein complexes, so AUROC/AUPRC understate true regulatory complexity
- scGATE (the inference being benchmarked) is applicable only to a small (<200) subset of genes with a refined candidate TF list, so benchmarks reflect small, pre-selected panels
- Decoding Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting, limiting benchmark scope to k<=5 TF subsets

## Evidence

- [methods] Bayes Factor ranking of gates and edges: [evidence span withheld — anchor status: not_found]
