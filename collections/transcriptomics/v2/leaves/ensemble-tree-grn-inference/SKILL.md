---
name: ensemble-tree-grn-inference
description: Use when you need a strong classical baseline for GRN inference from
  a gene-by-cell count matrix (e.g. 8000+ expressed genes in K562 perturb-seq data)
  to compare against foundation-model-derived networks like scPRINT or scGPT.
when_to_use_negative:
- The input is simulated ODE-generated expression data rather than real single-cell
  counts — the article warns many GN methods are inappropriately benchmarked on such
  mock-up data.
- You need a cell-state-specific or scalable method for very large datasets — the
  article notes most methods like GENIE3 do not scale well to cellxgene-scale cell
  and gene counts.
- The ground truth is a literature-based network where attention-head-based foundation
  models may be more appropriate; this skill is for count-matrix regression baselines.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GENIE3
  role: ensemble-tree GRN inference method run with default parameters and 100 trees
    on raw counts as the strongest classical comparator
- name: BenGRN
  role: benchmarking package used to match the inferred GENIE3 network to the gwps
    ground truth and compute AUPRC/EPR
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: gene regulatory network data structure package used alongside BenGRN for network
    matching and evaluation
  repo: https://github.com/cantinilab/GRnnData
- name: Python
  role: implementation environment for running GENIE3, BenGRN, and GRnnData in the
    benchmark workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:4d9325fa11e60926834e2c0f0bfea78a18b1956c9f0cda8de1c04ac0288c67b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jkobject/benGRN
  tools:
  - BenGRN
  - DeepSEM
  - GENIE3
  - GRnnData
  - Geneformer
  - Python
  - scGPT
  - scPRINT
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ensemble-tree-grn-inference

## Summary

Run GENIE3-style ensemble tree (Random Forest/Gradient Boosting) gene regulatory network inference on single-cell expression matrices, using regression trees across genes to infer regulator-target edges, and benchmark the resulting network against ground truths with AUPRC and EPR metrics.

## When to use

Use when you need a strong classical baseline for GRN inference from a gene-by-cell count matrix (e.g. 8000+ expressed genes in K562 perturb-seq data) to compare against foundation-model-derived networks like scPRINT or scGPT.

## When NOT to use

- The input is simulated ODE-generated expression data rather than real single-cell counts — the article warns many GN methods are inappropriately benchmarked on such mock-up data.
- You need a cell-state-specific or scalable method for very large datasets — the article notes most methods like GENIE3 do not scale well to cellxgene-scale cell and gene counts.
- The ground truth is a literature-based network where attention-head-based foundation models may be more appropriate; this skill is for count-matrix regression baselines.

## Inputs

- raw gene-by-cell count matrix from K562 perturb-seq cells (8000+ expressed genes, same cell/gene set as comparator methods)
- binarized directed gene-by-gene ground-truth network (gwps supplementary corrected p-values, 0.05 cutoff)

## Outputs

- inferred weighted gene regulatory network (regulators as columns, targets as rows)
- per-method AUPRC and EPR metrics compiled into gwps_benchmark_metrics.csv

## How to apply

Run GENIE3 with default parameters using 100 trees on the raw count matrix for the same cells and gene set used by all comparator methods (no extra normalization, unlike DeepSEM which gets 10,000-count normalization, log1p, and per-gene z-scoring). Orient the inferred network with regulators as columns and sparsely regulated targets as rows — no transposition — before matching it to the ground truth (e.g. a binarized genome-wide perturb-seq K562 network built with a 0.05 cutoff on multiple-hypothesis-testing-corrected p-values) using the benGRN and GRnnData packages. Score with AUPRC (area above the positive-class prevalence floor, composite trapezoidal rule without interpolation to perfect recall) and EPR (diagnostic odds ratio (TP x TN) / (FP x FN) at the cutoff giving K positive predictions, where K equals the number of ground-truth positives). In the scPRINT gwps benchmark, GENIE3 with this configuration was the best overall method, only outperformed by scPRINT when attention heads were selected against the gwps ground truth with a RidgeClassifier (L2 penalty 1, positive coefficients, no intercept).

## Related tools

- **GENIE3** (ensemble-tree GRN inference method run with default parameters and 100 trees on raw counts as the strongest classical comparator)
- **BenGRN** (benchmarking package used to match the inferred GENIE3 network to the gwps ground truth and compute AUPRC/EPR) — https://github.com/jkobject/benGRN
- **GRnnData** (gene regulatory network data structure package used alongside BenGRN for network matching and evaluation) — https://github.com/cantinilab/GRnnData
- **Python** (implementation environment for running GENIE3, BenGRN, and GRnnData in the benchmark workflow)

## Examples

```
from GENIE3 import GENIE3; network = GENIE3(raw_counts.T, tree_method='RF', n_trees=100)
```

## Evaluation signals

- GENIE3's inferred network is oriented with regulators as columns and sparsely regulated targets as rows before matching — a transposed network will fail the BenGRN match against the gwps ground truth.
- AUPRC is reported as area above the positive-class prevalence floor computed with the composite trapezoidal rule without interpolation to perfect recall; values should be comparable across methods on identical ground truth.
- EPR equals (TP x TN) / (FP x FN) at the score cutoff yielding exactly K positive predictions, where K is the number of ground-truth positives.
- Expected outcome: GENIE3 ranks best overall on the genome-wide perturb-seq benchmark, second only to scPRINT with gwps-selected heads; a much lower EPR (as seen for Geneformer v2) indicates misconfiguration.
- Per-method AUPRC and EPR values assemble into gwps_benchmark_metrics.csv without missing entries for GENIE3.

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; perturb-seq ground truths remain scarce and ambiguous, so benchmark results are indicative rather than definitive.
- GENIE3's best performance is partly attributed to the evaluation setting — scPRINT with head selection still outperformed it, and the article frames GENIE3 as 'still a competitive tool' rather than universally superior.
- GENIE3 was run on raw counts here; results may not transfer to normalized or batch-corrected inputs.

## Evidence

- [other] GENIE3 with default parameters and 100 trees on raw counts: [evidence span withheld — anchor status: not_found]
- [discussion] GENIE3 competitive tool: "while GENIE3 is still a competitive tool, we outperformed it on many of our benchmarks"
- [discussion] BenGRN and GRnnData benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [methods] gwps edge cutoff: "We used a cutoff of 0.05 on these values to define the directed binary connection between genes"
- [methods] gwps ground truth availability: "This is available for all 8000+ expressed genes in the K562 cell line."
