---
name: zero-shot-foundation-model-evaluation
description: Use when comparing foundation models' zero-shot embedding quality on
  batch effect correction, e.g. when a task asks to reproduce the scIB aggregated
  (avgBIO) benchmark on pancreas/lung open-problems data without fine-tuning.
when_to_use_negative:
- The task is gene network inference benchmarking (BenGRN/GRnnData suite), not embedding-based
  batch effect correction — the paper treats these as separate evaluations.
- 'Fine-tuned, task-specific model performance is required: this skill evaluates zero-shot
  embeddings only; scPRINT and scGPT embeddings are extracted without fine-tuning
  on the batch correction task.'
- The evaluation target is denoising or cell label prediction rather than batch effect
  correction measured by scIB avgBIO.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPRINT
  role: Foundation model under test; zero-shot embeddings taken from classifier output
    logits of the pretrained checkpoint
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Baseline foundation model; cell embeddings extracted from the cell embedding
    token without task fine-tuning
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Baseline; cell embeddings computed with the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522
    model
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: scFoundation
  role: Baseline; embeddings on the 5000 most variable genes, 3224-d output reduced
    to 512-d by PCA
  repo: https://github.com/biomap-research/scFoundation
- name: scIB
  role: Evaluation suite computing the avgBIO aggregate from ARI/NMI on KNN clusters
    and label-free metrics
- name: openproblems
  role: Source of the pancreas and lung batch-correction benchmark datasets, train-test
    split by batch, and preprocessing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:86c58cc0491fc73bd430777b9878755a9845163b78441e6c7943d5b814826390
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# zero-shot-foundation-model-evaluation

## Summary

Benchmark single-cell foundation models (scPRINT, scGPT, Geneformer, scFoundation) on batch effect correction by computing zero-shot cell embeddings and scoring them with the scIB aggregated avgBIO metric on the human pancreas and lung open-problems datasets.

## When to use

Use when comparing foundation models' zero-shot embedding quality on batch effect correction, e.g. when a task asks to reproduce the scIB aggregated (avgBIO) benchmark on pancreas/lung open-problems data without fine-tuning.

## When NOT to use

- The task is gene network inference benchmarking (BenGRN/GRnnData suite), not embedding-based batch effect correction — the paper treats these as separate evaluations.
- Fine-tuned, task-specific model performance is required: this skill evaluates zero-shot embeddings only; scPRINT and scGPT embeddings are extracted without fine-tuning on the batch correction task.
- The evaluation target is denoising or cell label prediction rather than batch effect correction measured by scIB avgBIO.

## Inputs

- human pancreas open-problems batch-correction dataset
- human lung open-problems batch-correction dataset
- pretrained scPRINT checkpoint (Hugging Face: huggingface.co/jkobject/scPRINT)
- pretrained scGPT checkpoint (jkobject/scGPT fork, mytests/ folder)
- Geneformer v2 fine-tuned model gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522 (huggingface.co/ctheodoris/Geneformer)
- scFoundation pretrained model (biomap-research/scFoundation)

## Outputs

- scib_avgbio_benchmark.csv containing per-dataset avgBIO scores for scPRINT and each baseline, alongside open-problems-portal state-of-the-art values
- zero-shot cell embeddings for each model on each dataset

## How to apply

Load the human pancreas and lung open-problems batch-correction datasets following the openproblems benchmark setup, keeping the same train-test split by batch and preprocessing. Compute scPRINT embeddings zero-shot from the pretrained checkpoint (Hugging Face jkobject/scPRINT) using the classifier output logits as embeddings, without fine-tuning. For baselines, extract scGPT embeddings from the cell embedding token directly, Geneformer v2 embeddings from the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' model, and scFoundation embeddings on the 5000 most variable genes per dataset, reducing the 3224-dimensional output to 512 with PCA. Evaluate all embeddings with scIB, computing the avgBIO aggregate over label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) and label-free metrics, then collate per-dataset avgBIO scores alongside open-problems-portal state-of-the-art results into scib_avgbio_benchmark.csv.

## Related tools

- **scPRINT** (Foundation model under test; zero-shot embeddings taken from classifier output logits of the pretrained checkpoint) — https://github.com/cantinilab/scPRINT
- **scGPT** (Baseline foundation model; cell embeddings extracted from the cell embedding token without task fine-tuning) — https://github.com/jkobject/scGPT
- **Geneformer** (Baseline; cell embeddings computed with the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522 model) — https://huggingface.co/ctheodoris/Geneformer
- **scFoundation** (Baseline; embeddings on the 5000 most variable genes, 3224-d output reduced to 512-d by PCA) — https://github.com/biomap-research/scFoundation
- **scIB** (Evaluation suite computing the avgBIO aggregate from ARI/NMI on KNN clusters and label-free metrics)
- **openproblems** (Source of the pancreas and lung batch-correction benchmark datasets, train-test split by batch, and preprocessing)

## Examples

```
# compute avgBIO with scIB on zero-shot embeddings for each model, then: scib_avgbio_benchmark.to_csv('scib_avgbio_benchmark.csv')
```

## Evaluation signals

- scib_avgbio_benchmark.csv contains an avgBIO score for every model on every dataset (pancreas and lung).
- scPRINT avgBIO scores indicate convincing batch effect removal but fall below state-of-the-art methods scGEN and scVI.
- Among foundation models, scPRINT and scFoundation should show strong zero-shot performances relative to Geneformer v2 and scGPT.
- avgBIO is computed from both label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) and label-free metrics, so both components must be present in the scIB run.
- scPRINT scores should be competitive with the open-problems-portal state-of-the-art values reported in the paper.

## Limitations

- scPRINT's batch effect removal, while convincing, is not on par with state-of-the-art methods scGEN and scVI.
- Zero-shot evaluation only: scPRINT embeddings come from classifier output logits without any fine-tuning, so results do not reflect fine-tuned task performance.
- No perfect ground truths exist for gene network inference benchmarking more broadly, and pre-training/integration questions remain open per the discussion.

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [other] scIB metrics, scPRINT shows convincing batch effect removal: [evidence span withheld — anchor status: not_found]
- [intro] This suggests that by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology.: "This suggests that by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology."
- [other] classifier output logits used as zero-shot embeddings: [evidence span withheld — anchor status: not_found]
- [methods] scDataloader samples cells from cellxgene release: [evidence span withheld — anchor status: not_found]
