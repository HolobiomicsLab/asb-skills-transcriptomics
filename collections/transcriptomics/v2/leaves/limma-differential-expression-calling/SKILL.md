---
name: limma-differential-expression-calling
description: Use when you have normalized bulk RNA-seq count matrices (e.g. MOSim-simulated
  with a two-group design) and need a DEG call to define true positives for benchmarking
  or to feed downstream GRN/regulator modeling.
when_to_use_negative:
- Single-cell simulation output (scRNA-seq/scATAC-seq from sc_mosim) — the MOSim paper
  uses Kendall's Tb correlation for single-cell data and this bulk voom/limma workflow
  is not designed for it
- Data already summarized as DEG lists or feature tables with no raw/normalized count
  matrix to fit linear models on
- When the goal is to validate the simulator's temporal or regulatory behavior rather
  than to define a DEG set (use correlation-based validation against 20,000 random
  feature pairs instead)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the bulk RNA-seq simulation with ground-truth DE features against
    which limma's DEG calls are benchmarked
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Execution environment for the limma/voom/NOISeq DE-calling workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:ccc691f0c9185165e3c0200fc655b8f58ab42856c9dacb0c8dcd9a44bb92d9d8
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
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# limma-differential-expression-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls differentially expressed genes between experimental groups on bulk RNA-seq count data using limma linear modeling after voom transformation, as done in the MOSim/MORE benchmark where limma was applied to MOSim-simulated STATegra-configured data to identify the DEG set that seeds downstream GRN inference.

## When to use

Use when you have normalized bulk RNA-seq count matrices (e.g. MOSim-simulated with a two-group design) and need a DEG call to define true positives for benchmarking or to feed downstream GRN/regulator modeling.

## When NOT to use

- Single-cell simulation output (scRNA-seq/scATAC-seq from sc_mosim) — the MOSim paper uses Kendall's Tb correlation for single-cell data and this bulk voom/limma workflow is not designed for it
- Data already summarized as DEG lists or feature tables with no raw/normalized count matrix to fit linear models on
- When the goal is to validate the simulator's temporal or regulatory behavior rather than to define a DEG set (use correlation-based validation against 20,000 random feature pairs instead)

## Inputs

- Bulk RNA-seq count matrix (e.g. MOSim-simulated with mosim using the STATegra-derived configuration, fixed seed, 30M reads)
- NOISeq-filtered/normalized count matrix ready for voom transformation
- Design matrix or sample metadata specifying the two experimental groups

## Outputs

- List of differentially expressed genes with limma statistics (benchmark expectation: 10,593 DEGs between the two groups)
- voom-transformed expression matrix and fitted limma model objects used as linear-modeling input to MORE GRN inference

## How to apply

Preprocess the simulated count matrices with NOISeq filtering and normalization, then apply voom transformation to convert counts to precision-weighted logCPM suitable for linear modeling. Build a design matrix reflecting the two simulated groups (e.g. the STATegra-derived configuration with 30M reads, two groups, 20 time points) and fit per-gene linear models with limma, calling DEGs between the groups. In the MORE benchmark this step yielded 10,593 DEGs, which define the gene universe for the subsequent MORE PLS1 regression with Jack-Knife resampling (5,573 fitted gene models). Validate the call against the simulator's ground-truth DE labels, since MOSim explicitly outputs differentially abundant features per omics layer for benchmarking true positive/negative definition.

## Related tools

- **MOSim** (Generates the bulk RNA-seq simulation with ground-truth DE features against which limma's DEG calls are benchmarked) — github.com/ConesaLab/MOSim
- **R** (Execution environment for the limma/voom/NOISeq DE-calling workflow)

## Examples

```
v <- voom(counts, design); fit <- eBayes(lmFit(v, design)); degs <- topTable(fit, number=Inf)
```

## Evaluation signals

- DEG count matches the benchmark expectation of 10,593 differentially expressed genes between the two simulated groups
- Downstream MORE inference on the DEG universe fits 5,573 per-gene PLS1 models and enumerates 370,566 potential gene–regulator interactions
- Benchmark metrics computed against simulated ground truth reach sensitivity 85.5% and F1-score 62.9%
- DEG calls are consistent with MOSim's exported differentially abundant features, which define true positive/negative feature sets
- voom transformation succeeds only after NOISeq filtering/normalization — verify counts are library-size normalized before lmFit

## Limitations

- The benchmark design used one replicate per time point in a two-group setup; limma's empirical-Bayes moderation assumptions may behave differently with richer replicate structures
- Expected DEG counts and metrics are specific to the MOSim STATegra-derived configuration (fixed seed, 30M reads, 50% DEGs, 60% significant regulations) and will not transfer to other simulation settings
- MOSim cannot simulate proteomics or metabolomics layers, so limma DE-calling in this context is limited to transcriptomic count data

## Evidence
