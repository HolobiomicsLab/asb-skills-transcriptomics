---
name: nmi-clustering-metric-computation
description: Use when you have predicted cluster labels from an scRNA-seq embedding
  (e.g. Louvain clusters on scGAE latent embeddings) plus ground-truth cell labels
  and need a single NMI score per condition, such as comparing clustering across dropout-rate
  variants.
when_to_use_negative:
- When ground-truth cell labels are unavailable — NMI requires reference labels, so
  unsupervised-only datasets cannot be scored this way
- When the input is a raw count matrix and no embedding/clustering has been produced
  — compute the embedding (scGAE) and Louvain clusters first; NMI is the evaluation
  step, not the clustering step
- When you need trajectory-inference quality rather than discrete-cluster separation
  — the paper notes scGAE and scGNN have similar trajectory-inference performance
  while NMI-based comparison targets clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: Produces the latent embeddings and Louvain cluster labels (get_centers_louvain,
    model.clustering_train, model.embedding) that are compared against ground truth
    via NMI
  repo: github.com/ZixiangLuo1161/scGAE
- name: scikit-learn
  role: Provides the normalized mutual information score used to compare predicted
    clusters with ground-truth labels; pinned at 0.22.1 in the scGAE environment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:f0f5c625b77ff875b0aba54c6fd4ab522fe8113a54197c093181db6c8ad10d42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmi-clustering-metric-computation

## Summary

Computes normalized mutual information (NMI) between predicted Louvain cluster labels from scGAE embeddings and ground-truth cell labels, used to quantify clustering quality — e.g. scGAE's robustness to zero inflation across scenario4 dropout variants (NMI ≈ 0.62, 0.62, 0.65, 0.61 at 12%, 17%, 23%, 30% dropout).

## When to use

Use when you have predicted cluster labels from an scRNA-seq embedding (e.g. Louvain clusters on scGAE latent embeddings) plus ground-truth cell labels and need a single NMI score per condition, such as comparing clustering across dropout-rate variants.

## When NOT to use

- When ground-truth cell labels are unavailable — NMI requires reference labels, so unsupervised-only datasets cannot be scored this way
- When the input is a raw count matrix and no embedding/clustering has been produced — compute the embedding (scGAE) and Louvain clusters first; NMI is the evaluation step, not the clustering step
- When you need trajectory-inference quality rather than discrete-cluster separation — the paper notes scGAE and scGNN have similar trajectory-inference performance while NMI-based comparison targets clustering

## Inputs

- Ground-truth cell label vector for a simulated/annotated scRNA-seq dataset (e.g. scenario4 with 12–30% dropout)
- Predicted cluster label vector from Louvain clustering of scGAE latent embeddings (obtained after model.clustering_train() / get_centers_louvain)
- Preprocessed scRNA-seq count matrices (one per condition, e.g. four scenario4 dropout variants)

## Outputs

- One NMI value per dataset variant comparing predicted vs ground-truth clusters
- A summary table (e.g. 4-row CSV) with dropout rate and NMI per row (expected ≈ 0.62, 0.62, 0.65, 0.61)

## How to apply

Run the full scGAE pipeline per dataset: preprocess the count matrix, build the KNN cell graph (default K=35), train the multitask-oriented graph autoencoder (implemented with Spektral) to obtain low-dimensional embeddings, then obtain initial centroids and predicted labels via Louvain clustering on the pre-trained embedding. Compute NMI between the Louvain-predicted cluster labels and the ground-truth labels for each dataset variant (scikit-learn 0.22.1 is available in the scGAE environment). For the scenario4 dropout-robustness check, tabulate one row per dropout rate (12±0.3%, 17±0.4%, 23±0.5%, 30±0.6%) with its NMI; a value counts as reproduced if it falls within a tolerance band around the reported number (e.g. 0.55–0.70). Because the paper reports no random seed or stochastic-variation control, expect run-to-run NMI variation and treat the tolerance band rather than exact equality as the acceptance criterion.

## Related tools

- **scGAE** (Produces the latent embeddings and Louvain cluster labels (get_centers_louvain, model.clustering_train, model.embedding) that are compared against ground truth via NMI) — github.com/ZixiangLuo1161/scGAE
- **scikit-learn** (Provides the normalized mutual information score used to compare predicted clusters with ground-truth labels; pinned at 0.22.1 in the scGAE environment)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score; nmi = normalized_mutual_info_score(ground_truth_labels, louvain_pred_labels)
```

## Evaluation signals

- Each NMI value falls within the validation tolerance band around the reported number (e.g. 0.55–0.70 for the scenario4 series; reported 0.62, 0.62, 0.65, 0.61 at 12%, 17%, 23%, 30% dropout)
- NMI is computed against the true ground-truth label vector, not against the Louvain clusters themselves (no self-comparison inflation)
- One NMI value exists per dataset variant and the output table has matching row counts (4 rows for the four dropout rates)
- The clustering labels evaluated come from the post-clustering-training model (initial centroids from Louvain on pre-trained embeddings), matching the paper's experimental setup
- NMI values remain stable across dropout rates (no monotonic collapse as dropout rises from 12% to 30%), consistent with the reported zero-inflation robustness finding

## Limitations

- No random seed or stochastic-variation control is reported in the paper, so NMI values can vary between runs; validation must use a tolerance band (e.g. 0.55–0.70) rather than exact reproduction
- The optimal K for the KNN cell graph (default 35) varies among datasets and dataset regions, which can shift clustering quality and thus NMI
- scGAE has no direct 2D embedding; intermediate-dimension embeddings plus tSNE are used, so NMI measures clustering of the intermediate/latent embedding rather than a 2D projection

## Evidence

- [discussion] louvain clustering on pre-trained embeddings: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [readme] louvain clustering training step: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [abstract] clustering performance claim supporting NMI evaluation: "Analyses of simulated data showed that scGAE accurately reconstructs developmental trajectory and separates discrete cell clusters under different scenarios, outperforming recently developed deep"
- [discussion] KNN graph default parameter: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
