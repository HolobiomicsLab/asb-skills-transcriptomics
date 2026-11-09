---
name: pancreas-lung-dataset-preprocessing
description: Use when benchmarking batch effect removal of single-cell foundation
  model embeddings (scPRINT, scGPT, Geneformer v2, scFoundation) against open-problems
  SOTA on the human pancreas and lung datasets using scIB avgBIO.
when_to_use_negative:
- The goal is gene network inference or denoising rather than batch-effect correction
  benchmarking — this skill covers only the open-problems batch-correction task on
  pancreas and lung.
- You need state-of-the-art batch removal rather than zero-shot foundation-model embeddings
  — the article reports scPRINT is not on par with SOTA methods scGEN and scVI.
- Your datasets are spatial omics or non-human/mouse — the scPRINT preprocessing filters
  keep only primary cells from humans or mice and drop spatial omics.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: produces zero-shot cell embeddings (classifier output logits) from the pretrained
    checkpoint
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: baseline foundation model; embeddings extracted from the cell embedding token
    without fine-tuning (runs via the jkobject fork, mytests/ folder)
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: baseline; cell embeddings from the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522
    model
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: scFoundation
  role: baseline; embeddings on the 5000 most variable genes per dataset, 3224-dim
    output reduced to 512 by PCA
  repo: https://github.com/biomap-research/scFoundation
- name: scIB
  role: computes the avgBIO aggregate over label-based (ARI, NMI on KNN clusters)
    and label-free metrics
- name: openproblems
  role: provides the benchmark setup, datasets, and portal state-of-the-art values
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:083bd2749ff3dcaf385caa446f043851c35a36936a35605cbbcd6f1480953f9f
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pancreas-lung-dataset-preprocessing

## Summary

Reproduce the open-problems batch-effect correction benchmark by loading the human pancreas and lung batch-correction datasets (same train-test split by batch and preprocessing), computing embeddings from scPRINT and baseline foundation models, and scoring them with the scIB aggregated avgBIO metric.

## When to use

Use when benchmarking batch effect removal of single-cell foundation model embeddings (scPRINT, scGPT, Geneformer v2, scFoundation) against open-problems SOTA on the human pancreas and lung datasets using scIB avgBIO.

## When NOT to use

- The goal is gene network inference or denoising rather than batch-effect correction benchmarking — this skill covers only the open-problems batch-correction task on pancreas and lung.
- You need state-of-the-art batch removal rather than zero-shot foundation-model embeddings — the article reports scPRINT is not on par with SOTA methods scGEN and scVI.
- Your datasets are spatial omics or non-human/mouse — the scPRINT preprocessing filters keep only primary cells from humans or mice and drop spatial omics.

## Inputs

- human pancreas open-problems batch-correction dataset (train-test split by batch)
- human lung open-problems batch-correction dataset (train-test split by batch)
- pretrained scPRINT checkpoint (Hugging Face: jkobject/scPRINT)
- scGPT pretrained model weights
- Geneformer v2 fine-tuned checkpoint 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522'
- scFoundation pretrained model (100M parameters)

## Outputs

- cell embeddings per model per dataset (scPRINT classifier logits; scGPT cell embedding token; Geneformer v2 embeddings; scFoundation 512-dim after PCA)
- scIB avgBIO scores per model per dataset (ARI/NMI on KNN clusters + label-free metrics)
- collated comparison table with open-problems-portal SOTA results saved as scib_avgbio_benchmark.csv

## How to apply

Load the human pancreas and lung open-problems batch-correction datasets following the openproblems benchmark setup, keeping the same train-test split by batch and preprocessing. Compute scPRINT cell embeddings zero-shot from the pretrained checkpoint (available on Hugging Face at jkobject/scPRINT), using the classifier output logits as embeddings with no fine-tuning. Extract scGPT embeddings from the model's cell embedding token without fine-tuning, Geneformer v2 embeddings from the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' checkpoint, and scFoundation embeddings on the 5000 most variable genes per dataset, reducing its 3224-dimensional output to 512 with PCA. Evaluate every set of embeddings with scIB, computing the avgBIO aggregate over label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) and label-free metrics, then collate per-dataset avgBIO scores alongside open-problems-portal state-of-the-art results into scib_avgbio_benchmark.csv.

## Related tools

- **scPRINT** (produces zero-shot cell embeddings (classifier output logits) from the pretrained checkpoint) — https://github.com/cantinilab/scPRINT
- **scGPT** (baseline foundation model; embeddings extracted from the cell embedding token without fine-tuning (runs via the jkobject fork, mytests/ folder)) — https://github.com/jkobject/scGPT
- **Geneformer** (baseline; cell embeddings from the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522 model) — https://huggingface.co/ctheodoris/Geneformer
- **scFoundation** (baseline; embeddings on the 5000 most variable genes per dataset, 3224-dim output reduced to 512 by PCA) — https://github.com/biomap-research/scFoundation
- **scIB** (computes the avgBIO aggregate over label-based (ARI, NMI on KNN clusters) and label-free metrics)
- **openproblems** (provides the benchmark setup, datasets, and portal state-of-the-art values)

## Evaluation signals

- The output table scib_avgbio_benchmark.csv contains an avgBIO score for each model on each dataset (pancreas and lung).
- scPRINT's avgBIO scores are competitive with the baseline foundation models (scGPT, Geneformer v2, scFoundation) and close to the open-problems-portal SOTA values reported in the paper.
- Label-based metrics are computed as ARI and NMI on K-Nearest-Neighbor clusters, consistent with the paper's scIB setup.
- scFoundation embeddings are reduced from 3224 to 512 dimensions via PCA after subsetting to the 5000 most variable genes per dataset.
- No fine-tuning was applied for scPRINT (zero-shot) and scGPT (embedding token extracted directly), matching the paper's benchmark protocol.

## Limitations

- scPRINT's batch effect removal is convincing per scIB but not on par with state-of-the-art methods scGEN and scVI.
- scPRINT embeddings are zero-shot (classifier logits), so performance may differ under task-specific fine-tuning.
- scFoundation embeddings are limited to the 5000 most variable genes per dataset and require a PCA dimensionality-reduction step (3224 to 512).

## Evidence

- [other] scPRINT shows convincing batch effect removal (not on par with SOTA methods scGEN and scVI): [evidence span withheld — anchor status: not_found]
- [methods] zero-shot embeddings from classifier logits: [evidence span withheld — anchor status: not_found]
- [methods] avgBIO aggregate over label-based and label-free metrics: [evidence span withheld — anchor status: not_found]
- [methods] scFoundation HVG subsetting and PCA: [evidence span withheld — anchor status: not_found]
- [abstract] competitive zero-shot abilities claim: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
