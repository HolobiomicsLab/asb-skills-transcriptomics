---
name: kernel-target-alignment-scoring
description: Use when you must choose the transfer learning mixture parameter theta
  in [0,1] for a modified target dataset Xtrg_new without ground-truth labels, or
  when validating that unsupervised KTA-based theta selection tracks supervised ARI
  across overlap settings.
when_to_use_negative:
- When ground-truth labels are available and a directly supervised selection of theta
  (via ARI) is preferred — KTA is the unsupervised surrogate.
- When the goal is plain SC3 clustering of the original target data without transfer
  learning — no theta sweep or KTA scoring is needed.
- When source and target datasets share only a small set of transcripts — the article
  notes a loss of vital information is inevitable since only shared genes can be used.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SC3
  role: Clusters each theta-modified target dataset Xtrg_new; its predicted labels'
    linear kernel is compared to the data kernel via KTA.
- name: scRNA (cmd_generate_data.py)
  role: Generates the simulated count-level scRNA-Seq datasets (10,000 genes, 1,800
    cells; complete/incomplete/no overlap) on which per-theta KTA and ARI are computed.
  repo: https://github.com/nicococo/scRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:21c5c30e8d91b61894f0cf044926b7f78c9f67ecd4040310b1aa04f64c594365
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Kernel Target Alignment (KTA) scoring for unsupervised theta selection

## Summary

Computes Kernel Target Alignment (KTA) scores between the linear kernel of a transfer-modified scRNA-Seq target dataset and the linear kernel of SC3-predicted cell-type labels, providing an unsupervised criterion for automatically selecting the mixture parameter theta.

## When to use

Use when you must choose the transfer learning mixture parameter theta in [0,1] for a modified target dataset Xtrg_new without ground-truth labels, or when validating that unsupervised KTA-based theta selection tracks supervised ARI across overlap settings.

## When NOT to use

- When ground-truth labels are available and a directly supervised selection of theta (via ARI) is preferred — KTA is the unsupervised surrogate.
- When the goal is plain SC3 clustering of the original target data without transfer learning — no theta sweep or KTA scoring is needed.
- When source and target datasets share only a small set of transcripts — the article notes a loss of vital information is inevitable since only shared genes can be used.

## Inputs

- Transfer-modified target dataset Xtrg_new (cells x genes matrix) for each candidate theta
- SC3-predicted cell-type label vector for each theta
- Simulation ground-truth labels (for the supervised ARI validation arm)
- Pre-specified range of theta values within [0,1]

## Outputs

- Per-theta KTA score vector
- Per-theta supervised ARI values (validation arm)
- KTA-ARI correlation summary per overlap setting
- kta_ari_correlation.csv (per-theta KTA scores plus correlation summary)

## How to apply

For each candidate theta in a pre-specified range within [0,1], construct the modified target Xtrg_new = theta * Hsrc * W'trg + (1-theta) * Xtrg using the NMF-learned source dictionary Hsrc (with elastic net regularization) and the max-simplified binary target membership matrix W'trg. Cluster each Xtrg_new with SC3, then compute the KTA score between the linear kernel of Xtrg_new over cells and the linear kernel of the SC3-predicted cluster-label kernel. Pick the theta maximizing the KTA score as the unsupervised choice; to validate, correlate per-theta KTA scores with supervised ARI values against simulation ground truth and check that the KTA-argmax theta coincides with the ARI-argmax theta (following Supplementary Section 2.4 and Supplementary Fig. S4, e.g. on simulated 10,000-gene / 1,800-cell datasets across complete, incomplete, and no overlap settings). The rationale is that KTA assesses clustering quality without labels, making the transfer learning workflow applicable to real data where ground truth is unavailable.

## Related tools

- **SC3** (Clusters each theta-modified target dataset Xtrg_new; its predicted labels' linear kernel is compared to the data kernel via KTA.)
- **scRNA (cmd_generate_data.py)** (Generates the simulated count-level scRNA-Seq datasets (10,000 genes, 1,800 cells; complete/incomplete/no overlap) on which per-theta KTA and ARI are computed.) — https://github.com/nicococo/scRNA

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; python scripts/experiments/main_wrapper_generated_data.py  # sweep theta, SC3-cluster each Xtrg_new, record KTA and ARI
```

## Evaluation signals

- The theta maximizing the KTA score coincides with the theta maximizing supervised ARI for each overlap setting (complete, incomplete, no overlap).
- KTA scores are computed for every theta in the pre-specified range and span a non-degenerate range (not all identical).
- kta_ari_correlation.csv contains per-theta KTA scores plus a correlation summary consistent with Supplementary Section 2.4 / Supplementary Fig. S4.
- Supervised accuracy measures remain 0 when ground-truth labels are absent, while unsupervised KTA measures are still produced.
- ARI validation values match the reproduced comparison table for the corresponding theta values.

## Limitations

- KTA selection is a proxy; its agreement with supervised ARI is an empirical observation demonstrated on simulated data and Supplementary Fig. S4, not guaranteed for all datasets.
- Only genes present in both source and target can be used; if Xsrc and Xtrg share few transcripts, vital information is lost before KTA scoring.
- The KTA-based theta sweep over cluster ranges and mixture range can be computationally slow on large data (per the README), though t-SNE plots can be turned off with --no-tsne.

## Evidence

- [methods] KTA is the unsupervised selection criterion: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [methods] Modified target feeds clustering: [evidence span withheld — anchor status: not_found]
- [readme] Unsupervised and supervised measures co-reported: "Command line output shows a number of results: unsupervised and supervised (if no ground truth labels are given this will remain 0.) accuracy measures."
- [methods] Simulation scale and overlap settings: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [methods] Convex combination construction of Xtrg_new: [evidence span withheld — anchor status: not_found]
