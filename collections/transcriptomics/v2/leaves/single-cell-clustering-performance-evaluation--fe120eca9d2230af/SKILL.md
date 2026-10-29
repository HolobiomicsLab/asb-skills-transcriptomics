---
name: single-cell-clustering-performance-evaluation
description: Use when benchmarking a clustering or imputation method (e.g., scGCL,
  scTAG) on labeled scRNA-seq h5 data and you must quantify which KNN graph settings
  (k in {5,10,15,20,25}; city-block, cosine, euclidean, L2, manhattan distances) maximize
  ARI/NMI.
when_to_use_negative:
- The dataset lacks true cell labels, since ARI/NMI cannot be computed against ground
  truth
- You are evaluating gene-imputation accuracy rather than clustering performance;
  scGCL reports these as distinct evaluations
- The method under test does not build a KNN cell graph from the expression matrix,
  making the k/distance sweep inapplicable
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scGCL
  role: Graph contrastive learning + ZINB autoencoder imputation method whose clustering
    task is evaluated under each k–distance configuration
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Runtime for executing the scGCL clustering pipeline (Python 3.8.8) from the
    command line
- name: PyTorch
  role: Deep learning backend (PyTorch 1.10.0) for training the scGCL model during
    clustering runs
- name: torch-geometric
  role: Graph library (2.2.0) used to build and process the KNN cell graphs swept
    over k and distance measures
- name: torch-cluster
  role: Provides KNN graph construction operators (version 1.6.0) for the cell–cell
    graphs
- name: faiss-cpu
  role: Fast nearest-neighbor search library (1.7.3) supporting KNN graph construction
    over cells
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:55ad9f175c0e3aecb19ea568714c2b58a68b5accf323249568d338558f2112cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - torch-cluster
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-clustering-performance-evaluation

## Summary

Evaluates single-cell clustering pipelines on scRNA-seq data by sweeping KNN cell-graph hyperparameters (number of neighbors k and distance measure) and scoring each configuration with ARI and NMI against true cell labels, as used to benchmark scGCL against scTAG.

## When to use

Use when benchmarking a clustering or imputation method (e.g., scGCL, scTAG) on labeled scRNA-seq h5 data and you must quantify which KNN graph settings (k in {5,10,15,20,25}; city-block, cosine, euclidean, L2, manhattan distances) maximize ARI/NMI.

## When NOT to use

- The dataset lacks true cell labels, since ARI/NMI cannot be computed against ground truth
- You are evaluating gene-imputation accuracy rather than clustering performance; scGCL reports these as distinct evaluations
- The method under test does not build a KNN cell graph from the expression matrix, making the k/distance sweep inapplicable

## Inputs

- h5 file (e.g., adam dataset) containing a matrix of scRNA-seq expression values, true labels, and other information
- KNN graph hyperparameter grid: k in {5, 10, 15, 20, 25} and distance measures {city-block, cosine, euclidean, L2, manhattan}
- Baseline method results (scTAG) for comparison

## Outputs

- Per-configuration ARI and NMI scores against true cell labels, averaged over repeated runs
- A sweep table of averaged ARI/NMI for all k–distance combinations with scTAG baseline comparison
- Identified optimal configuration (cosine distance, k=15)

## How to apply

Load an h5 dataset containing the scRNA-seq expression matrix and true labels (e.g., the adam Alzheimer's disease dataset). For each k in {5, 10, 15, 20, 25} and each distance measure in {city-block, cosine, euclidean, L2, manhattan}, build the KNN cell graph and run the clustering task from the command line (e.g., 'python main.py --dataset adam') with early stopping enabled. Compute ARI and NMI against the true labels for every parameter combination, averaging over repeated runs to control run-to-run variance. Compile the averaged ARI/NMI values into a single sweep table alongside the baseline (scTAG) comparison. Judge success by verifying the table reproduces the reported result: the cosine distance captures cell–cell distance most effectively on sparse scRNA-seq data, and both scGCL and scTAG achieve their best clustering at k=15, with scGCL consistently outperforming scTAG across all tested settings.

## Related tools

- **scGCL** (Graph contrastive learning + ZINB autoencoder imputation method whose clustering task is evaluated under each k–distance configuration) — https://github.com/zehaoxiong123/scGCL
- **Python** (Runtime for executing the scGCL clustering pipeline (Python 3.8.8) from the command line)
- **PyTorch** (Deep learning backend (PyTorch 1.10.0) for training the scGCL model during clustering runs)
- **torch-geometric** (Graph library (2.2.0) used to build and process the KNN cell graphs swept over k and distance measures)
- **torch-cluster** (Provides KNN graph construction operators (version 1.6.0) for the cell–cell graphs)
- **faiss-cpu** (Fast nearest-neighbor search library (1.7.3) supporting KNN graph construction over cells)

## Examples

```
python main.py --dataset adam --task clustering --es
```

## Evaluation signals

- The sweep table shows cosine distance achieving the highest average ARI/NMI across k values on sparse scRNA-seq data
- Both scGCL and scTAG peak at k=15 in the averaged ARI/NMI sweep
- scGCL's averaged ARI/NMI is consistently higher than scTAG's at every tested k–distance combination
- All ARI/NMI values are computed against the true labels embedded in the h5 file and averaged over repeated runs
- Each clustering run completes with early stopping enabled rather than running a fixed epoch count

## Limitations

- The optimal setting (cosine, k=15) was established on the adam (Alzheimer's disease) dataset among 14 scRNA-seq datasets; other datasets may shift the optimum
- Results are averaged over repeated runs, so single-run scores may vary and should not be used for the sweep table
- The sweep is restricted to k in {5,10,15,20,25} and the five listed distance measures; configurations outside this grid are untested

## Evidence

- [results] The 'cosine' distance captures cell–cell distance most effectively on sparse scRNA-seq data, and both scGCL and scTAG achieve their best clustering effect at k=15: [evidence span withheld — anchor status: not_found]
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] Downstream task. Supported tasks are: node, clustering, similarity: "Downstream task. Supported tasks are: node, clustering, similarity"
- [abstract] scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets: "scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets"
- [methods] $ python main.py --dataset adam --epochs 300: "$ python main.py --dataset adam --epochs 300"
