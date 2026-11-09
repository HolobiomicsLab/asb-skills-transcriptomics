---
name: rna-seq-count-filtering-normalization
description: Use when raw or simulated RNA-seq count matrices (e.g. MOSim mosim()
  bulk output) must be filtered and normalized prior to limma/voom linear modeling,
  DE testing, or GRN-inference preprocessing.
when_to_use_negative:
- Input is already normalized (e.g. TPM/logCPM) or consists of continuous expression
  values rather than raw counts — NOISeq filtering and voom assume count data
- Downstream analysis is single-cell validation of gene–regulator relationships, where
  the article's protocol uses Kendall's Tb correlation instead of the bulk Pearson-correlation
  route
- The omics layer is not gene expression (e.g. ATAC-seq, ChIP-seq, Methyl-seq), which
  in MOSim serve as regulatory layers rather than the central RNA-seq count data this
  skill normalizes
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Generates the simulated bulk RNA-seq count matrices (and regulatory omics
    layers) that this skill filters and normalizes
  repo: github.com/ConesaLab/MOSim
- name: NOISeq
  role: Performs filtering and normalization of the simulated RNA-seq count matrices
    before linear modeling
- name: limma
  role: Consumes the voom-transformed normalized data for differential gene expression
    calls between the two groups
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:5e5728207ffbfa971ae1bc159ae7c8f556c11cd39c6f54fca35be0e6e8be6df4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rna-seq-count-filtering-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters and normalizes RNA-seq count matrices — here, MOSim-simulated bulk RNA-seq counts — before linear modeling, using NOISeq filtering/normalization plus voom transformation to feed limma differential expression calls and downstream GRN inference.

## When to use

Use when raw or simulated RNA-seq count matrices (e.g. MOSim mosim() bulk output) must be filtered and normalized prior to limma/voom linear modeling, DE testing, or GRN-inference preprocessing.

## When NOT to use

- Input is already normalized (e.g. TPM/logCPM) or consists of continuous expression values rather than raw counts — NOISeq filtering and voom assume count data
- Downstream analysis is single-cell validation of gene–regulator relationships, where the article's protocol uses Kendall's Tb correlation instead of the bulk Pearson-correlation route
- The omics layer is not gene expression (e.g. ATAC-seq, ChIP-seq, Methyl-seq), which in MOSim serve as regulatory layers rather than the central RNA-seq count data this skill normalizes

## Inputs

- MOSim-simulated bulk RNA-seq count matrix (integer counts, RNA-seq as central data type)
- Experimental design specification (e.g. two groups, 20 time points, fixed seed, 30M reads)
- Bulk multi-omics regulatory layers (miRNA-seq, TF data) to be processed in parallel

## Outputs

- NOISeq-filtered and -normalized count matrix
- voom-transformed logCPM expression matrix with precision weights
- limma DEG list between groups (benchmark expectation: 10,593 DEGs)
- log(x+1)-transformed matrices for Pearson-correlation validation of gene–regulator pairs

## How to apply

Simulate or obtain RNA-seq count data (MOSim's mosim() function is the source in this benchmark) and treat the RNA-seq count matrix as the central data type to preprocess. Apply NOISeq filtering/normalization to remove low-count features and make counts comparable across samples, then apply voom transformation to obtain logCPM expression values with precision weights suitable for linear modeling. Call differentially expressed genes between the two groups with limma on the voom output (the benchmark expects 10,593 DEGs); these filtered/normalized matrices then feed MORE's PLS1 per-gene models. Separately, for correlation-based validation of simulated gene–regulator relationships, apply a log(x+1) transformation to the data and compute Pearson's correlation on bulk data (Kendall's Tb for single-cell), comparing against 20,000 random feature pairs with no simulated regulatory effects. Choose voom/logCPM when the downstream step is limma or another linear model, and log(x+1) when the downstream step is correlation validation.

## Related tools

- **MOSim** (Generates the simulated bulk RNA-seq count matrices (and regulatory omics layers) that this skill filters and normalizes) — github.com/ConesaLab/MOSim
- **NOISeq** (Performs filtering and normalization of the simulated RNA-seq count matrices before linear modeling)
- **limma** (Consumes the voom-transformed normalized data for differential gene expression calls between the two groups)

## Examples

```
Rscript -e 'library(limma); v <- voom(counts, design); fit <- lmFit(v, design); de <- topTable(eBayes(fit), number=Inf)'
```

## Evaluation signals

- The limma DE call on the voom output yields approximately 10,593 DEGs, matching the benchmark expectation on the MOSim/STATegra configuration
- The voom output is log2-CPM-scale expression with precision weights suitable for limma linear modeling
- After normalization, count distributions are comparable across samples of the two groups (20 time points, fixed seed, 30M reads)
- Downstream MORE inference fits ~5,573 per-gene PLS1 models on the preprocessed matrices, indicating filtering did not eliminate modeling features
- Pearson correlations on log(x+1)-transformed bulk data for simulated gene–regulator pairs separate from the 20,000 random no-regulation feature pairs

## Limitations

- The benchmark design uses one replicate per group (20 time points), which constrains within-group variability estimation for voom precision weights and DE power
- This normalization/filtering protocol was demonstrated on bulk RNA-seq; single-cell simulation and validation follow a different route (Kendall's Tb correlation rather than bulk Pearson)
- Expected DEG counts and downstream metrics (10,593 DEGs, 85.5% sensitivity, 62.9% F1) are specific to the MOSim/STATegra simulation configuration and are not universal targets

## Evidence

- [methods] a log transformation (log(x+1)) [42] was applied to the data: [evidence span withheld — anchor status: not_found]
