---
name: hill-function-expression-transformation
description: 'Use when inferring Boolean logic gates or TF–target edges with scGATE:
  after scRNA_seq_preprocessing() rescaling into (0,1), TF/target profiles must be
  Hill-transformed before enumerating 2^k logic combinations.'
when_to_use_negative:
- The expression matrix has not been rescaled into the (0,1) interval by scRNA_seq_preprocessing()
  — the Hill function and zero-inflated mixture assume continuous normalized values
- Pseudotime-sorted data along differentiation trajectories or temporal-delay modeling
  is the goal — scGATE's Hill-based model does not yet handle this
- The TF candidate subset exceeds ~5 regulators per target — overfitting makes high-order
  Boolean logic inference unreliable (recommended max k=5, default max_num_regulators=3)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scGATE
  role: R package whose scGATE_logic()/scGATE_edge() functions perform the Hill activation
    transformation (parameterized by h_set, run_mode, h_act) on TF and target profiles
    prior to gate/edge inference
  repo: github.com/CompBioIPM/scGATE
- name: VGAM
  role: Used downstream to fit the zero-inflated distribution over Hill-transformed
    logic-combination outputs
- name: truncnorm
  role: Provides truncated normal density used with VGAM for the normal component
    centered on each logic combination's Hill-transformed output
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:7cc58824031638374cb9c3bbae3ab84423a649c60d040e1c1da31a23ffffa4f1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - R
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hill-function-expression-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transforms preprocessed scRNA-seq TF and target-gene expression profiles with the Hill activation function H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h) so that continuous expression is modeled without binarization as a preparatory step for scGATE logic-gate inference.

## When to use

Use when inferring Boolean logic gates or TF–target edges with scGATE: after scRNA_seq_preprocessing() rescaling into (0,1), TF/target profiles must be Hill-transformed before enumerating 2^k logic combinations.

## When NOT to use

- The expression matrix has not been rescaled into the (0,1) interval by scRNA_seq_preprocessing() — the Hill function and zero-inflated mixture assume continuous normalized values
- Pseudotime-sorted data along differentiation trajectories or temporal-delay modeling is the goal — scGATE's Hill-based model does not yet handle this
- The TF candidate subset exceeds ~5 regulators per target — overfitting makes high-order Boolean logic inference unreliable (recommended max k=5, default max_num_regulators=3)

## Inputs

- scRNA-seq expression matrix preprocessed by scRNA_seq_preprocessing(), with normalized counts within the (0,1) interval (cells in rows, genes in columns)
- Optional base GRN TF–target candidate list read via read_base_GRN() from a .parquet file (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- Optional tf_list restricting which columns are treated as TFs

## Outputs

- Hill-transformed TF activity profiles H(tf) and transformed target profiles used in the zero-inflated mixture likelihood
- Per-target logic-gate strings and confidence scores (columns: gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate) from scGATE_logic()/scGATE_gate()
- BF-ranked TF–target edge list ('from', 'to', 'BF_score') from scGATE_edge()

## How to apply

First run scRNA_seq_preprocessing() (with library_size_normalization 'True' or 'False' depending on simulated vs real data) so the expression matrix lies in the (0,1) interval, since the Hill activation function requires continuous bounded expression and avoids information loss from binarization. For each target gene, transform both TF and target expression profiles with H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h), searching over the 'h' parameter space: set h_set = NA for the default search, or use run_mode = 'simple' for a faster run vs 'complex' for a more precise search over the Hill parameter space. Compute each of the 2^k logic-combination outputs (for k <= max_num_regulators, default 3, recommended up to 5) as products of H(tf) and [1 - H(tf)] terms, then feed these to the zero-inflated mixture likelihood (VGAM/truncnorm) whose priors are updated by EM. The rationale is that Hill-activated TF activities act as the covariates in the Bayesian gate-inference model, so ill-scaled or binarized inputs break the dropout-plus-normal mixture fit.

## Related tools

- **scGATE** (R package whose scGATE_logic()/scGATE_edge() functions perform the Hill activation transformation (parameterized by h_set, run_mode, h_act) on TF and target profiles prior to gate/edge inference) — github.com/CompBioIPM/scGATE
- **VGAM** (Used downstream to fit the zero-inflated distribution over Hill-transformed logic-combination outputs)
- **truncnorm** (Provides truncated normal density used with VGAM for the normal component centered on each logic combination's Hill-transformed output)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, h_set = NA, number_of_em_iterations = 3, max_num_regulators = 3, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- On BoolODE synthetic toggle-switch ClusterI.csv, inferred gates should match the designed ground truth (e.g. gE -> ~gF with log10 BF 442.47, gE1 -> gE.~gE2 with log10 BF 286.50)
- gates_results.csv contains columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate, with positive log10 BF and non-empty logic_gate for every reported target
- Transformed profiles remain within the (0,1)-bounded activation range implied by H(tf) = (k_sat^h + 1)*tf^h/(k_sat^h + tf^h) applied to rescaled expression
- max_num_regulators kept at the default 3 (at most 5) so each gate enumerates 2^k logic combinations without overfitting
- At the default top_gates=1 and weight_threshold, only the logic combination generating more than 5% of the target gene's observations is reported as active (ON)

## Limitations

- Overfitting is a major concern for decoding high-order Boolean logic involving more than 5 transcriptional regulators
- scGATE is currently applicable only to a small (<200) subset of genes with a refined list of candidate TFs from external hints, not large-scale inference from scratch
- The 'h' parameter space search is a trade-off: run_mode 'simple' is faster but less precise than 'complex'
- Large protein-complex interactions will be missed by the model and by ChIP-based ground-truth networks

## Evidence
