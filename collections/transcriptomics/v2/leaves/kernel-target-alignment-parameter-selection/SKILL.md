---
name: kernel-target-alignment-parameter-selection
description: Use when a target scRNA-Seq dataset has been reconstructed via the source
  dictionary H_src (X_trg_new = theta*H_src*W'trg + (1-theta)*X_trg) and you need
  to choose theta without ground-truth labels.
when_to_use_negative:
- Ground-truth labels for the target cells are available and supervised tuning of
  theta (e.g. by maximizing ARI) is feasible — the paper's KTA route is specifically
  the unsupervised option
- The source and target share no (or an extremely small) gene set — the article notes
  loss of vital information is inevitable when only shared transcripts can be used,
  making the reconstructed target unreliable for KTA scoring
- The downstream analysis is not clustering-derived (KTA assesses cluster-structure
  alignment, so the score is not meaningful for other objectives)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scRNA
  role: Python framework implementing the transfer workflow; the target script iterates
    over the mixture (theta) range and reports unsupervised accuracy measures, including
    KTA-based selection
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream clustering method applied to X_trg_new at the KTA-selected theta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:14aa83b1f7269a6ce9a1e267cf41f4af9d33e15c6d19167a8f5e894e440c2266
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kernel-target-alignment-parameter-selection

## Summary

Automatic unsupervised selection of the mixture parameter theta for NMF-based transfer learning in scRNA-Seq clustering, by scoring Kernel Target Alignment (KTA) over a pre-specified range of convex-combination weights instead of hand-tuning.

## When to use

Use when a target scRNA-Seq dataset has been reconstructed via the source dictionary H_src (X_trg_new = theta*H_src*W'trg + (1-theta)*X_trg) and you need to choose theta without ground-truth labels.

## When NOT to use

- Ground-truth labels for the target cells are available and supervised tuning of theta (e.g. by maximizing ARI) is feasible — the paper's KTA route is specifically the unsupervised option
- The source and target share no (or an extremely small) gene set — the article notes loss of vital information is inevitable when only shared transcripts can be used, making the reconstructed target unreliable for KTA scoring
- The downstream analysis is not clustering-derived (KTA assesses cluster-structure alignment, so the score is not meaningful for other objectives)

## Inputs

- Reconstructed target data theta*H_src*W'trg and original (pre-processed) target matrix X_trg
- Binarized target clustering matrix W'trg derived from W_trg by column-wise maxima
- A pre-specified numeric range of theta candidate values

## Outputs

- Selected theta (single scalar) maximizing KTA score
- Per-theta KTA scores over the range (summarizing figure in scRNA scripts)
- Modified target dataset X_trg_new at the selected theta, ready for SC3 clustering

## How to apply

After learning the target data matrix W_trg against the source dictionary H_src and binarizing it column-wise to W'trg, define X_trg_new as a convex combination of the reconstructed and original target data with mixture parameter theta. Sweep theta over a pre-specified range, and for each candidate value compute a Kernel Target Alignment (KTA) score, an unsupervised assessment of clustering quality that needs no ground-truth labels. Select the theta with the best KTA score as the mix-in weight. The scRNA framework exposes this as a user-defined mix-in of the source data influence during augmented target clustering (the target script iterates over a mixture range). Rationale: the transfer must balance injected source structure against preservation of the original target signal, and KTA provides a label-free proxy so the parameter is chosen automatically rather than manually.

## Related tools

- **scRNA** (Python framework implementing the transfer workflow; the target script iterates over the mixture (theta) range and reports unsupervised accuracy measures, including KTA-based selection) — https://github.com/nicococo/scRNA
- **SC3** (Downstream clustering method applied to X_trg_new at the KTA-selected theta)

## Examples

```
scRNA-target.sh --src src_c8.npz --cluster-range 6,7,8 --no-tsne
```

## Evaluation signals

- KTA score landscape over the theta range should be non-degenerate (summarizing .png figure in scRNA target output) with a clear maximum
- At the selected theta, SC3 on X_trg_new should not yield ARI significantly lower than SC3 on the untouched target (TargetCluster baseline), per the paper's success criterion
- theta must lie within the convex combination constraint (0 <= theta <= 1); the command-line run should report unsupervised accuracy for each mixture value
- Predicted cluster labels .tsv files should be produced for every value in the --cluster-range at the chosen theta

## Limitations

- KTA is an unsupervised proxy; the simplified max-based W'trg was chosen empirically because it 'performed best and most consistently for all scenarios', implying other binarizations may degrade the KTA signal
- If X_src and X_trg share only a small set of transcripts, a loss of (probably) vital information is inevitable — the article defers differing gene-space handling to future work
- No software version numbers are stated for the framework or SC3, so exact reproduction of KTA score behavior may vary

## Evidence

- [methods] Automatic selection of mixture parameter via KTA: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [methods] Convex combination definition: [evidence span withheld — anchor status: not_found]
- [methods] Simplified binarization rationale: "it was found that taking the simplified version as described above performed best and most consistently for all scenarios under investigation"
- [readme] README mix-in description: "augmented clustering of the target data with user defined mix-in of the source data influence."
- [readme] README target workflow outputs: "Command line output shows a number of results: unsupervised and supervised (if no ground truth labels are given this will remain 0.) accuracy measures."
