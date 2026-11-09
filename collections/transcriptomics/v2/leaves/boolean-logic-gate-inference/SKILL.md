---
name: boolean-logic-gate-inference
description: Use when you have a preprocessed scRNA-seq expression matrix and a refined
  candidate TF list (or base GRN from scATAC-seq + motifs) for <200 target genes,
  and you need the combinatorial Boolean rule (not just TF-target edges) governing
  each target in a specific cell type or tissue.
when_to_use_negative:
- 'Large-scale de novo network inference over many genes: scGATE is applicable only
  to a small (<200) subset of genes with a refined candidate TF list based on external
  hints'
- Pseudotime-sorted single-cell data along differentiation trajectories or when temporal
  delays in regulation must be modeled — scGATE does not yet handle this
- Decoding Boolean logic involving more than 5 transcriptional regulators per target,
  where overfitting is a major concern
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: Core R package performing Hill-function transformation, zero-inflated mixture
    fitting, EM prior updates, and Bayes-factor-ranked Boolean gate inference (scGATE_logic
    / scGATE_gate / scGATE_edge)
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: R dependency used for fitting the zero-inflated distribution of target gene
    expression over logic combinations
- name: truncnorm
  role: R dependency for truncated normal density terms in the mixture likelihood
- name: arrow
  role: Reads .parquet base GRN files (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
    via read_parquet()
- name: BoolODE
  role: Generates the synthetic toggle-switch scRNA-seq benchmark data (ClusterI.csv)
    with ground-truth Boolean rules
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:e75512f577a2d34244433c68f39a8c7d7a3102a4f8e79b23b3182503fdcdbd67
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
  - BoolODE
  - R
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# boolean-logic-gate-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers context-specific Boolean logic gates (AND, OR, XOR, NOT combinations) among candidate TFs regulating target genes from scRNA-seq data using scGATE, which fits a zero-inflated mixture likelihood over 2^k logic combinations with EM-updated priors and ranks gates by log10 Bayes Factor.

## When to use

Use when you have a preprocessed scRNA-seq expression matrix and a refined candidate TF list (or base GRN from scATAC-seq + motifs) for <200 target genes, and you need the combinatorial Boolean rule (not just TF-target edges) governing each target in a specific cell type or tissue.

## When NOT to use

- Large-scale de novo network inference over many genes: scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list based on external hints
- Pseudotime-sorted single-cell data along differentiation trajectories or when temporal delays in regulation must be modeled — scGATE does not yet handle this
- Decoding Boolean logic involving more than 5 transcriptional regulators per target, where overfitting is a major concern

## Inputs

- Preprocessed scRNA-seq count matrix (cells x genes, normalized/rescaled to (0,1) interval), e.g. ClusterI.csv from BoolODE synthetic toggle-switch simulation
- Base GRN of candidate TF-target links in .parquet format from scATAC-seq + TF binding motifs (e.g. Buenrostro2018_base_GRN_dataframe.parquet), optional
- Candidate TF list (e.g. Gata1, Fli1, Klf1, Spi1, Zfpm1, Tal1, Gata2 for MegE differentiation)
- scGATE parameters: number_of_em_iterations, max_num_regulators, top_gates, run_mode, h_set, abs_cor, weight_threshold

## Outputs

- Logic gate table per target gene with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate (e.g. scgate_synthetic_gates.csv / gates_results.csv)
- Ranked TF-target network with Boolean logic strings (e.g. gE -> ~gF, gE1 -> gE.~gE2, Fli1 = Gata1 AND NOT Klf1)
- Directed TF-gene edges when run via scGATE_edge() without gates

## How to apply

Preprocess the count matrix with scRNA_seq_preprocessing() to rescale expression into the (0,1) interval (set library_size_normalization='False' if already normalized upstream), and remove low-variability genes (sqrt-variance SD > 0.20 for the BoolODE synthetic example). Restrict to a candidate TF set, optionally reading a .parquet base GRN via read_base_GRN() derived from scATAC-seq and TF binding motifs. Transform TF and target profiles with the Hill activation function H(tf) = (k_sat^h + 1)*tf^h/(k_sat^h + tf^h), then run scGATE_logic() (or scGATE_gate()/scGATE_edge() wrappers) with max_num_regulators <= 3-5 to enumerate 2^k logic combinations, fit the zero-inflated (dropout point mass + truncated normal) distribution with VGAM/truncnorm, and update prior weights w_v over combinations via EM (default 3 iterations; 10 used for the mouse haematopoiesis MegE analysis). Report the top gate per target by log10 BF = L1/L0, where L0 uses uniform priors w_v = 1/2^k; mark a combination active only if it generates >5% of the target gene's observations. Use run_mode='simple' for speed (e.g. the ClusterI toggle-switch example with top_gates=1) and 'complex' for more precise Hill parameter-space search (e.g. number_of_em_iterations=10 on the Gata1/Fli1/Klf1 megakaryocyte panel).

## Related tools

- **scGATE** (Core R package performing Hill-function transformation, zero-inflated mixture fitting, EM prior updates, and Bayes-factor-ranked Boolean gate inference (scGATE_logic / scGATE_gate / scGATE_edge)) — https://github.com/CompBioIPM/scGATE
- **VGAM** (R dependency used for fitting the zero-inflated distribution of target gene expression over logic combinations)
- **truncnorm** (R dependency for truncated normal density terms in the mixture likelihood)
- **arrow** (Reads .parquet base GRN files (e.g. Buenrostro2018_base_GRN_dataframe.parquet) via read_parquet())
- **BoolODE** (Generates the synthetic toggle-switch scRNA-seq benchmark data (ClusterI.csv) with ground-truth Boolean rules)

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "False"); gates <- scGATE_logic(data = normalized_counts, number_of_em_iterations = 3, max_num_regulators = 3, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- Output CSV contains the required columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate, with positive log10 BF and non-empty logic_gate string for each reported target
- Inferred gates match ground-truth BoolODE toggle-switch rules (gE -> ~gF with log10 BF 442.47; gE1 -> gE.~gE2 with log10 BF 286.50)
- On mouse haematopoiesis data, Fli1 = Gata1 AND NOT Klf1 is ranked as the top-confidence gate among the 16 candidate logic gates
- AUROC/AUPRC against the deposited ground-truth GRN (Zenodo 10.5281/zenodo.8353409) reproduces benchmark performance
- Each active logic combination generates more than 5% (weight_threshold) of the target gene's observations

## Limitations

- scGATE must currently be applied to a relatively small pre-selected set of TFs based on prior biological knowledge; high-order Boolean logic involving >5 regulators is intractable due to overfitting
- Cannot handle pseudotime-sorted single-cell data along differentiation trajectories or temporal delays in regulation
- Directed TF-gene networks alone do not convey combinatorial relationships, and interactions within large protein complexes are missed by both the model and ground-truth networks

## Evidence

- [methods] Candidate TF subset size: [evidence span withheld — anchor status: not_found]
