---
name: deep-embedding-baseline-method-execution
description: Use when benchmarking a new scRNA-seq clustering method against published
  baselines (scDeepCluster, DCA, DESC, scVI, scCAEs, DEC, scziDesk, scGNN, scGAE,
  GraphSCC, Seurat, SHARP) on the same normalized, HVG-filtered matrices.
when_to_use_negative:
- You are running the new method itself (scMGCA) rather than the comparison baselines
  — this skill covers only the 12 comparison methods
- The datasets lack ground-truth cell-type labels, so NMI/ARI cannot be computed against
  references
- A baseline requires method-specific preprocessing that conflicts with the shared
  top-500-HVG pipeline (e.g., SHARP's own all-zero-gene removal and conditional log-transform);
  forcing the shared pipeline would invalidate the baseline's default behavior
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scziDesk
  role: deep self-training K-means clustering baseline run from its published implementation
  repo: https://github.com/xuebaliang/scziDesk
- name: scDeepCluster
  role: model-based deep embedding clustering baseline run from its published implementation
  repo: https://github.com/ttgump/scDeepCluster
- name: DCA
  role: deep count autoencoder (ZINB loss) baseline for denoising/dimensionality reduction
  repo: https://github.com/theislab/dca
- name: DEC
  role: unsupervised deep embedding clustering baseline (DEC-keras)
  repo: https://github.com/XifengGuo/DEC-keras
- name: DESC
  role: unsupervised deep embedding algorithm baseline
  repo: https://github.com/eleozzr/desc
- name: scVI
  role: single-cell variational inference baseline
  repo: https://github.com/YosefLab/scVI
- name: scCAEs
  role: convolutional autoencoder embedding with soft K-means baseline
  repo: https://github.com/gushenweiz/scCAEs
- name: scGNN
  role: single-cell graph neural network baseline
  repo: https://github.com/juexinwang/scGNN
- name: scikit-learn
  role: computes NMI and ARI between predicted and ground-truth labels
- name: Seurat
  role: R-based single-cell clustering baseline
- name: SHARP
  role: ensemble random projection clustering baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:06145c40f6a185607d3992dfe4b5b60e498e6d9332fe682fc02da723eb7bc64f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuebaliang/scziDesk
  tools:
  - DCA
  - DEC
  - DESC
  - SCANPY
  - SHARP
  - Seurat
  - anndata
  - numpy
  - pandas
  - scCAEs
  - scDeepCluster
  - scGNN
  - scMGCA
  - scVI
  - scikit-learn
  - scipy
  - scziDesk
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# deep-embedding-baseline-method-execution

## Summary

Runs published deep-embedding and statistical clustering baselines (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE, GraphSCC, Seurat, SHARP) from their original implementations on identically preprocessed scRNA-seq count matrices so a new method (e.g., scMGCA) can be benchmarked fairly on NMI/ARI/ASW.

## When to use

Use when benchmarking a new scRNA-seq clustering method against published baselines (scDeepCluster, DCA, DESC, scVI, scCAEs, DEC, scziDesk, scGNN, scGAE, GraphSCC, Seurat, SHARP) on the same normalized, HVG-filtered matrices.

## When NOT to use

- You are running the new method itself (scMGCA) rather than the comparison baselines — this skill covers only the 12 comparison methods
- The datasets lack ground-truth cell-type labels, so NMI/ARI cannot be computed against references
- A baseline requires method-specific preprocessing that conflicts with the shared top-500-HVG pipeline (e.g., SHARP's own all-zero-gene removal and conditional log-transform); forcing the shared pipeline would invalidate the baseline's default behavior

## Inputs

- 20 real scRNA-seq count matrices from public repositories, preprocessed (genes filtered, normalized, top 500 highly variable genes retained)
- Ground-truth cell-type label vectors for each dataset
- Published baseline implementations (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE, GraphSCC, Seurat, SHARP)

## Outputs

- Per-method, per-dataset NMI, ARI, and ASW scores
- clustering_benchmark_results.csv containing all 13 methods x 20 datasets scores plus row-wise means across the 20 datasets

## How to apply

Preprocess each count matrix once — filter genes not expressed in cells, normalize the expression matrix, and retain the top d=500 highly variable genes — then feed the identical matrices to every baseline so differences reflect the methods, not preprocessing. Clone each method's published implementation (e.g., scziDesk from github.com/xuebaliang/scziDesk, scDeepCluster from github.com/ttgump/scDeepCluster, DCA from github.com/theislab/dca, DEC-keras from github.com/XifengGuo/DEC-keras, DESC from github.com/eleozzr/desc, scVI from github.com/YosefLab/scVI, scCAEs from github.com/gushenweiz/scCAEs) and run each with its default/recommended settings; note per-tool environment constraints such as scziDesk requiring Python 3.6 with TensorFlow 1.14 and Keras 2.2, or GraphSCC requiring PyTorch 1.1.0. For datasets with more than 25,000 cells, split into batches as scMGCA does, so baselines operate on the same partitioning. Compute NMI and ARI with scikit-learn against ground-truth cell-type labels, and ASW (average silhouette width) on each method's learned embeddings, then assemble per-dataset scores plus row-wise means across all 20 datasets into clustering_benchmark_results.csv.

## Related tools

- **scziDesk** (deep self-training K-means clustering baseline run from its published implementation) — https://github.com/xuebaliang/scziDesk
- **scDeepCluster** (model-based deep embedding clustering baseline run from its published implementation) — https://github.com/ttgump/scDeepCluster
- **DCA** (deep count autoencoder (ZINB loss) baseline for denoising/dimensionality reduction) — https://github.com/theislab/dca
- **DEC** (unsupervised deep embedding clustering baseline (DEC-keras)) — https://github.com/XifengGuo/DEC-keras
- **DESC** (unsupervised deep embedding algorithm baseline) — https://github.com/eleozzr/desc
- **scVI** (single-cell variational inference baseline) — https://github.com/YosefLab/scVI
- **scCAEs** (convolutional autoencoder embedding with soft K-means baseline) — https://github.com/gushenweiz/scCAEs
- **scGNN** (single-cell graph neural network baseline) — https://github.com/juexinwang/scGNN
- **scikit-learn** (computes NMI and ARI between predicted and ground-truth labels)
- **Seurat** (R-based single-cell clustering baseline)
- **SHARP** (ensemble random projection clustering baseline)

## Examples

```
library(SHARP); res = SHARP(scExp, rN.seed=2103); ARI(label, res)
```

## Evaluation signals

- Every one of the 12 baselines reports NMI and ARI against ground-truth labels on all 20 datasets, matching the scMGCA benchmark table
- Row-wise means across the 20 datasets are present for all 13 methods; in the paper's result scMGCA leads with mean NMI 0.8304, ARI 0.8278, ASW 0.5827
- All baselines were run on the same preprocessed matrices (filtered genes, normalized, top 500 HVGs), so per-dataset scores are directly comparable
- Datasets with >25,000 cells were divided into multiple batches consistently across methods
- Baseline results fall in ranges consistent with their original publications (e.g., SHARP example reports ARI 0.9675 on its reference pancreas data)

## Limitations

- No software version numbers are reported for Python, TensorFlow, SCANPY, Seurat, or R packages, so exact reproducibility of baseline environments is not guaranteed
- No specific dataset accessions or repository identifiers are provided; benchmark data are only described as coming from public repositories
- Baselines are run at default settings without per-dataset hyperparameter tuning, which may understate any baseline's achievable performance
- Per-baseline hardware/version constraints (e.g., scziDesk's TensorFlow 1.14/Keras 2.2/Python 3.6, GraphSCC's PyTorch 1.1.0) require isolated environments that can be difficult to co-install

## Evidence

- [intro] We evaluate the clustering performance of scMGCA by comparing it with the state-of-the-art clustering methods on several real scRNA-seq datasets: "We evaluate the clustering performance of scMGCA by comparing it with the state-of-the-art clustering methods on several real scRNA-seq datasets"
- [discussion] scDeepCluster reference link: [evidence span withheld — anchor status: not_found]
- [discussion] Sample data availability: "Sample data were obtained from public repositories."
- [readme] scDeepCluster README description: [evidence span withheld — anchor status: external_doc]
- [readme] scziDesk README requirements: [evidence span withheld — anchor status: external_doc]
- [readme] SHARP README benchmarking example: [evidence span withheld — anchor status: external_doc]
