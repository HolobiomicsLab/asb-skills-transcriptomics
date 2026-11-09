---
name: batch-effect-correction-benchmarking
description: Use when you need to compare how well foundation-model cell embeddings
  remove batch effects, measured by the scIB avgBIO aggregate, across open-problems
  batch-correction datasets such as human pancreas and lung.
when_to_use_negative:
- The task is gene network inference quality benchmarking (the paper's primary scPRINT
  evaluation) rather than batch-effect correction of cell embeddings
- The goal is fine-tuned, task-trained batch correction rather than zero-shot foundation-model
  embedding comparison — SOTA methods scGEN and scVI are trained on the task and are
  expected to outperform
- The input data is not from the open-problems batch-correction collection, so the
  shared preprocessing, train-test split by batch, and portal reference values would
  not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Produces zero-shot cell embeddings from the pretrained checkpoint using the
    classifier output logits, without fine-tuning
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Baseline foundation model; embeddings extracted directly from the model's
    cell embedding token without fine-tuning on batch correction (runs via the authors'
    fork)
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Baseline foundation model; cell embeddings computed with the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522'
    checkpoint
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: scFoundation
  role: Baseline foundation model; cell embeddings computed on the 5000 most variable
    genes per dataset, 3224-dim output reduced to 512 with PCA
  repo: https://github.com/biomap-research/scFoundation
- name: scIB
  role: Evaluates all embedding sets via the avgBIO aggregate over label-based (ARI
    and NMI on KNN clusters) and label-free metrics
- name: openproblems
  role: Provides the batch-correction datasets, shared preprocessing, and batch-based
    train-test split, plus the portal state-of-the-art reference scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:67c1d167908d139bc294570839adaadced92ddb88414e22ebf88e68079e6e75b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - Geneformer
  - Python
  - openproblems
  - scFoundation
  - scGPT
  - scIB
  - scPRINT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# batch-effect-correction-benchmarking

## Summary

Benchmarks zero-shot cell embeddings from single-cell foundation models (scPRINT, scGPT, Geneformer v2, scFoundation) on batch effect correction using the scIB aggregated avgBIO score on the human pancreas and lung open-problems datasets, collated against open-problems-portal state-of-the-art results.

## When to use

Use when you need to compare how well foundation-model cell embeddings remove batch effects, measured by the scIB avgBIO aggregate, across open-problems batch-correction datasets such as human pancreas and lung.

## When NOT to use

- The task is gene network inference quality benchmarking (the paper's primary scPRINT evaluation) rather than batch-effect correction of cell embeddings
- The goal is fine-tuned, task-trained batch correction rather than zero-shot foundation-model embedding comparison — SOTA methods scGEN and scVI are trained on the task and are expected to outperform
- The input data is not from the open-problems batch-correction collection, so the shared preprocessing, train-test split by batch, and portal reference values would not apply

## Inputs

- human pancreas open-problems batch-correction dataset (openproblems preprocessing and batch-based train-test split)
- human lung open-problems batch-correction dataset (openproblems preprocessing and batch-based train-test split)
- pretrained scPRINT checkpoint (available on Hugging Face at https://huggingface.co/jkobject/scPRINT)
- pretrained scGPT, Geneformer v2 ('gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522'), and scFoundation model weights

## Outputs

- per-dataset avgBIO scores for each foundation model on pancreas and lung
- scib_avgbio_benchmark.csv table including open-problems-portal state-of-the-art reference results

## How to apply

Load the human pancreas and lung open-problems batch-correction datasets following the openproblems benchmark setup, keeping the same train-test split by batch and preprocessing. Generate embeddings zero-shot for each model per the paper's protocol: scPRINT embeddings come from the pretrained checkpoint's classifier output logits without fine-tuning; scGPT embeddings come directly from the model's cell embedding token without fine-tuning on batch correction; Geneformer v2 embeddings use the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' checkpoint; scFoundation uses the 5000 most variable genes per dataset with the 3224-dimensional output reduced to 512 by PCA. Evaluate every embedding set with scIB, computing the avgBIO aggregate over label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) and label-free metrics. Collate per-dataset avgBIO scores for all models into a table with the open-problems-portal state-of-the-art results (e.g. scGEN and scVI) and save as scib_avgbio_benchmark.csv.

## Related tools

- **scPRINT** (Produces zero-shot cell embeddings from the pretrained checkpoint using the classifier output logits, without fine-tuning) — https://github.com/cantinilab/scPRINT
- **scGPT** (Baseline foundation model; embeddings extracted directly from the model's cell embedding token without fine-tuning on batch correction (runs via the authors' fork)) — https://github.com/jkobject/scGPT
- **Geneformer** (Baseline foundation model; cell embeddings computed with the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' checkpoint) — https://huggingface.co/ctheodoris/Geneformer
- **scFoundation** (Baseline foundation model; cell embeddings computed on the 5000 most variable genes per dataset, 3224-dim output reduced to 512 with PCA) — https://github.com/biomap-research/scFoundation
- **scIB** (Evaluates all embedding sets via the avgBIO aggregate over label-based (ARI and NMI on KNN clusters) and label-free metrics)
- **openproblems** (Provides the batch-correction datasets, shared preprocessing, and batch-based train-test split, plus the portal state-of-the-art reference scores)

## Examples

```
from sklearn.decomposition import PCA; emb_512 = PCA(n_components=512).fit_transform(scFoundation_embeddings)  # scFoundation: 5000 HVGs per dataset, 3224-dim output -> 512-dim
```

## Evaluation signals

- The output table scib_avgbio_benchmark.csv contains an avgBIO score for every model (scPRINT, scGPT, Geneformer v2, scFoundation) on each dataset (pancreas and lung)
- scPRINT's avgBIO scores are competitive with the other foundation models and with the open-problems-portal state-of-the-art values reported in the paper
- avgBIO is computed as the aggregate over label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) plus label-free metrics, matching the scIB definition
- All models use the same openproblems train-test split by batch and preprocessing, so scores are directly comparable
- scPRINT embeddings are extracted zero-shot (no fine-tuning); verify no task-specific training was applied to scPRINT or scGPT

## Limitations

- scPRINT's batch effect removal is convincing but not on par with state-of-the-art dedicated methods scGEN and scVI
- Among foundation models, results are mixed: scPRINT and scFoundation show strong zero-shot performance compared to Geneformer v2 and scGPT, so model ranking depends on the dataset
- Geneformer embeddings rely on a fine-tuned checkpoint while scPRINT and scGPT are zero-shot, which is a comparability consideration when interpreting the table

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [abstract] we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells from the cellxgene database: "we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells from the cellxgene database"
- [methods] follow the steps for cell embedding in the "model/README.md" file in their GitHub repository: [evidence span withheld — anchor status: not_found]
- [methods] All runs are in our fork: "https://github.com/jkobject/scGPT" in the "mytests/" folder.: [evidence span withheld — anchor status: not_found]
