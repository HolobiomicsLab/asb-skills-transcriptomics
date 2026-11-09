---
name: silhouette-score-embedding-assessment
description: Use when a joint embedding of annotated scRNA-seq and unlabeled scATAC-seq
  cells (e.g., scBridge output or a corrupted/downsampled source run) needs a quantitative
  embedding-quality metric alongside label transfer accuracy.
when_to_use_negative:
- When only label transfer accuracy is needed and no joint embedding or cell-type
  structure assessment is required
- When cells are unlabeled for both omics (no target annotations exist to score silhouette
  against cell-type structure)
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Produces the joint scRNA-seq/scATAC-seq embedding whose quality is scored
    by the silhouette metric
  repo: https://github.com/XLearning-SCU/scBridge
- name: Scikit-learn 1.1.1
  role: Provides the Gaussian Mixture Model machinery scBridge relies on; Scikit-learn
    metrics (e.g., silhouette_score) compute the embedding-quality score
  repo: https://github.com/XLearning-SCU/scBridge
- name: Python / PyTorch 1.12.1
  role: Execution environment for running scBridge integration runs that generate
    the embeddings to be assessed
  repo: https://github.com/XLearning-SCU/scBridge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:b4018d42aa7283d2424e7307c59dc4fe2afe7c267f36487ba76266a65f13848d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch 1.12.1
  - Python
  - Python / PyTorch 1.12.1
  - Scikit-learn 1.1.1
  - scBridge
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Silhouette score embedding assessment

## Summary

Quantifies the quality of scBridge's joint scRNA-seq/scATAC-seq embeddings by computing the silhouette score on the integrated representation against target cell-type annotations, complementing label transfer accuracy. Used to confirm that integration quality (e.g., under simulated dropout noise) tracks label-transfer performance.

## When to use

Use when a joint embedding of annotated scRNA-seq and unlabeled scATAC-seq cells (e.g., scBridge output or a corrupted/downsampled source run) needs a quantitative embedding-quality metric alongside label transfer accuracy.

## When NOT to use

- When only label transfer accuracy is needed and no joint embedding or cell-type structure assessment is required
- When cells are unlabeled for both omics (no target annotations exist to score silhouette against cell-type structure)

## Inputs

- Joint embedding coordinates from a scBridge integration run (source scRNA-seq h5ad + target scATAC-seq h5ad activity matrix)
- Target scATAC-seq cell-type annotations used as reference labels
- Optionally, multiple embedding runs from perturbed inputs (e.g., 25%/50%/75% dropout-downsampled scRNA-seq source data)

## Outputs

- Silhouette score per integration run (float, higher = better-separated cell types in the joint embedding)
- Aggregate comparison table of label transfer accuracy and silhouette score across dropout levels (e.g., dropout_benchmark_results.csv)

## How to apply

After running scBridge integration ('python main.py' with defaults such as reliability_threshold=0.95, prototype_momentum=0.9, early_stop_acc=0.99, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023), take the joint embedding output and the target scATAC-seq cell-type annotations. Compute the silhouette score on the embedding coordinates with the annotation labels (e.g., via Scikit-learn's silhouette_score), and pair it with label transfer accuracy computed against the same target annotations. Apply this to a baseline (uncorrupted) run and each perturbed run (e.g., scRNA-seq downsampled with scuttle's downsampleMatrix at 25%, 50%, 75% dropout) so the silhouette scores can be aggregated into a comparison table such as dropout_benchmark_results.csv. A near-constant silhouette score across perturbation levels, like the near-unchanged label transfer accuracy up to 75% dropout, is the expected correctness signal for this skill in the dropout-robustness setting.

## Related tools

- **scBridge** (Produces the joint scRNA-seq/scATAC-seq embedding whose quality is scored by the silhouette metric) — https://github.com/XLearning-SCU/scBridge
- **Scikit-learn 1.1.1** (Provides the Gaussian Mixture Model machinery scBridge relies on; Scikit-learn metrics (e.g., silhouette_score) compute the embedding-quality score) — https://github.com/XLearning-SCU/scBridge
- **Python / PyTorch 1.12.1** (Execution environment for running scBridge integration runs that generate the embeddings to be assessed) — https://github.com/XLearning-SCU/scBridge

## Examples

```
from sklearn.metrics import silhouette_score; s = silhouette_score(embedding, labels)
```

## Evaluation signals

- Silhouette score of the baseline (uncontaminated) scBridge run should be comparable to reported integration-quality results on the same dataset
- Silhouette score should remain near-unchanged when source data are corrupted with up to 75% dropout, mirroring stable label transfer accuracy
- Score must be computed on embedding coordinates paired with the exact target annotations used for accuracy evaluation
- Aggregated metrics across dropout levels (25%, 50%, 75%) should be consistently ordered in the comparison CSV (dropout_benchmark_results.csv)
- Runs must use scBridge defaults (e.g., reliability_threshold=0.95, early_stop_acc=0.99) so silhouette scores across runs are comparable

## Limitations

- Silhouette score depends on the availability of target cell-type annotations; it cannot be computed for fully unlabeled integrations
- The dropout-robustness conclusion (stable up to 75% dropout) is specific to the simulated count-downsampling procedure; other noise types are not covered
- Novel-type discovery and other supplementary analyses are deferred to Supplementary Notes not included in the source document

## Evidence

- [intro] integration performance evaluation scope: "We evaluate the data integration performance of scBridge on seven multi-omics datasets in terms of joint embedding quality and label transfer accuracy"
- [abstract] embedding from scBridge: "scBridge iterates between i) identifying reliable scATAC-seq cells that have smaller omics differences, and ii) integrating reliable scATAC-seq cells with scRNA-seq data"
- [readme] scBridge input format: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
- [discussion] early stopping criterion: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [discussion] Scikit-learn dependency: "We adopted the Gaussian Mixture Model provided in the Scikit-learn Python package42, v.1.1.1."
