---
name: mixture-parameter-sweep-analysis
description: Use when transferring knowledge from a source scRNA-Seq dataset to a
  smaller target and needing to choose the convex-combination mixture parameter theta
  without ground-truth labels, or when validating that unsupervised KTA-based theta
  selection tracks supervised ARI.
when_to_use_negative:
- Source and target datasets share only a small set of transcripts — the paper notes
  a loss of vital information is inevitable since only shared genes can be used, and
  handling differing gene sets is deferred to future work
- Ground-truth target labels are available and a supervised criterion (ARI) can be
  optimized directly, making unsupervised KTA selection unnecessary
- The downstream task is not clustering of the modified target dataset (the paper's
  downstream anchor is SC3; other analyses are untested in this workflow)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SC3
  role: Clusters each theta-modified target dataset Xtrg_new to produce the label
    kernel for KTA scoring and ARI computation
- name: scRNA (cmd_generate_data.py)
  role: Generates the simulated count-level datasets and source NMF models over which
    the theta sweep is run
  repo: https://github.com/nicococo/scRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:e2e1fa14200e06b3df9e350caae496ca564eb182dda4cb824b495cc4de2b827d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mixture-parameter-sweep-analysis

## Summary

Sweeps the transfer-learning mixture parameter theta over [0,1] and selects its value automatically via unsupervised Kernel Target Alignment (KTA) scores computed on SC3 clusterings of the modified target dataset, optionally correlating KTA against supervised ARI to validate the selection.

## When to use

Use when transferring knowledge from a source scRNA-Seq dataset to a smaller target and needing to choose the convex-combination mixture parameter theta without ground-truth labels, or when validating that unsupervised KTA-based theta selection tracks supervised ARI.

## When NOT to use

- Source and target datasets share only a small set of transcripts — the paper notes a loss of vital information is inevitable since only shared genes can be used, and handling differing gene sets is deferred to future work
- Ground-truth target labels are available and a supervised criterion (ARI) can be optimized directly, making unsupervised KTA selection unnecessary
- The downstream task is not clustering of the modified target dataset (the paper's downstream anchor is SC3; other analyses are untested in this workflow)

## Inputs

- Source NMF model (.npz) containing the learned source dictionary Hsrc from the scRNA package (e.g. src_c8.npz)
- Simulated or real target scRNA-Seq count matrix (e.g. 10,000 genes x 800 cells downsampled from 1,800-cell simulations)
- Max-simplified binary target membership matrix W'trg (column-wise maxima of Wtrg)
- Pre-specified theta mixture range within [0,1] and cluster range (default 6,7,8)
- Ground-truth target labels for computing supervised ARI (optional, 0 if absent)

## Outputs

- Per-theta KTA scores for each overlap setting and cluster number
- KTA-ARI correlation summary table (e.g. kta_ari_correlation.csv) verifying argmax coincidence
- SC3-predicted cell-type labels per theta (.tsv with cell ids) and t-SNE plots (.png)
- Unsupervised (KTA) and supervised (ARI, if labels given) accuracy measures per theta

## How to apply

For each candidate theta in a pre-specified range within [0,1], construct the modified target Xtrg_new = theta * Hsrc * W'trg + (1-theta) * Xtrg, where Hsrc is the NMF-learned source dictionary (with elastic net regularization, initialized from one-hot-encoded source labels) and W'trg is the target data matrix simplified to a binary membership matrix by column-wise maxima. Cluster each Xtrg_new with SC3 and compute the KTA score between the linear kernel of Xtrg_new over cells and the linear kernel of the SC3-predicted cell-type labels; pick the theta maximizing the KTA score. To validate the selection, retrieve supervised ARI values against known simulation ground truth for the same thetas and check that argmax(KTA) coincides with argmax(ARI) per overlap setting (complete, incomplete, no overlap), as in Supplementary Section 2.4 and Supplementary Fig. S4. Persist per-theta KTA scores and the KTA-ARI correlation summary (e.g. kta_ari_correlation.csv) so the sweep is auditable.

## Related tools

- **SC3** (Clusters each theta-modified target dataset Xtrg_new to produce the label kernel for KTA scoring and ARI computation)
- **scRNA (cmd_generate_data.py)** (Generates the simulated count-level datasets and source NMF models over which the theta sweep is run) — https://github.com/nicococo/scRNA

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; python scripts/experiments/main_wrapper_generated_data.py
```

## Evaluation signals

- Per-theta KTA scores are finite and defined for every theta in the specified range and each of the three overlap settings (complete, incomplete, no overlap)
- The theta maximizing the KTA score coincides with the theta maximizing supervised ARI for each overlap setting (cf. Supplementary Fig. S4)
- KTA-ARI correlation is positive within each overlap setting, supporting unsupervised theta selection
- Supervised accuracy is 0 wherever ground-truth labels were not supplied (README: 'this will remain 0'), confirming label handling is correct
- Output CSV contains one row per theta per setting with KTA and ARI columns and no missing values

## Limitations

- KTA-based theta selection is an unsupervised proxy; its agreement with ARI was validated on simulated data and may not transfer to all real datasets
- The method's guarantee that target clustering is independent of source cell types relies on the convex-combination construction with the simplified max-based W'trg; other simplifications performed less consistently
- If source and target gene sets differ substantially, only shared genes can be used and information loss is inevitable (explicitly deferred to future work)
- The theta sweep with NMF and clustering can take a long time depending on data, cluster range, and mixture range; t-SNE plotting can be disabled with --no-tsne to speed up

## Evidence

- [methods] Automatic selection of mixture parameter theta via KTA: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [methods] Construction of the modified target dataset: [evidence span withheld — anchor status: not_found]
- [methods] SC3 as downstream clustering input: [evidence span withheld — anchor status: not_found]
- [readme] User-defined mixture sweep in the package: "augmented clustering of the target data with user defined mix-in of the source data influence"
- [readme] Unsupervised vs supervised accuracy reporting: "Command line output shows a number of results: unsupervised and supervised (if no ground truth labels are given this will remain 0.) accuracy measures."
