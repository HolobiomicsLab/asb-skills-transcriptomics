---
name: pytorch-neural-network-implementation
description: Use when building a domain-generalization deep-learning classifier (e.g.,
  Cancer-Finder) that must distinguish malignant vs. non-malignant cells from normalized
  single-cell gene-expression matrices.
when_to_use_negative:
- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumors
- Input requires modeling spatial relationships among spots in spatial transcriptomics
  data, which Cancer-Finder overlooks
- Input is a spatial transcriptomics task needing spatial-neighborhood awareness or
  a training set where mislabeling exceeds the robust percentage
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: The domain-generalization model being implemented; PyTorch is its deep-learning
    framework
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Framework used to build the two-layer bottleneck network, dropout layer, and
    2-neuron classification head, and to optimize the variance + average risk loss
- name: Seurat
  role: Provides the log-transformed global-scaling normalization (NormalizeData +
    log1p) applied to training data before the network
- name: SCANPY
  role: Single-cell processing toolkit used alongside Seurat in the workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:f50b8504c309eabe4fe7bfd0ea859c5e198b8dbec4a2ad295cfe4c81007bc3cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pytorch-neural-network-implementation

## Summary

Implement the Cancer-Finder malignant-cell classifier as a PyTorch network: two activation-free fully connected layers (4572 inputs → 512-neuron bottleneck) separated by 0.5-probability dropout, followed by a 2-neuron classification layer with a 0.5 softmax threshold.

## When to use

Use when building a domain-generalization deep-learning classifier (e.g., Cancer-Finder) that must distinguish malignant vs. non-malignant cells from normalized single-cell gene-expression matrices.

## When NOT to use

- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumors
- Input requires modeling spatial relationships among spots in spatial transcriptomics data, which Cancer-Finder overlooks
- Input is a spatial transcriptomics task needing spatial-neighborhood awareness or a training set where mislabeling exceeds the robust percentage

## Inputs

- Normalized (Seurat NormalizeData + log1p) scRNA-seq expression matrix of 4572 highly variable genes
- Per-cell malignant/non-malignant labels from 74 TISCH tumor microenvironment datasets, downsampled 1:1 per tissue
- Extra-domain validation matrix: normalized breast cancer data from the original CopyKAT study

## Outputs

- Trained PyTorch model checkpoint with maximal extra-domain validation accuracy (~95.16% average internal-validation accuracy across 13 tissues)
- Per-cell malignant/non-malignant predictions from the softmax output (threshold 0.5)

## How to apply

Construct the network in PyTorch (torch 1.13.1, CUDA 11.6 environment per the repository README): an input layer of 4572 highly variable gene features feeds two fully connected layers without activation functions (4572 → 512-neuron bottleneck) separated by a dropout layer with probability 0.5 to prevent overfitting, followed by a classification layer with two neurons (malignant/non-malignant) whose softmax output is thresholded at 0.5. Train on 328,230 downsampled cells from 13 TISCH tissues with per-domain cross-entropy risks R_e, optimizing L(θ) = β·R_var(θ) + R_avg(θ) with β = 1 for 50–70 epochs under 5-fold leave-cells-out cross-validation. After each epoch, evaluate on the internal validation set (80/20 split) and on normalized breast cancer data from the CopyKAT study as extra-domain validation, retaining only the checkpoint with maximal extra-domain accuracy.

## Related tools

- **Cancer-Finder** (The domain-generalization model being implemented; PyTorch is its deep-learning framework) — https://github.com/Patchouli-M/SequencingCancerFinder
- **PyTorch** (Framework used to build the two-layer bottleneck network, dropout layer, and 2-neuron classification head, and to optimize the variance + average risk loss)
- **Seurat** (Provides the log-transformed global-scaling normalization (NormalizeData + log1p) applied to training data before the network)
- **SCANPY** (Single-cell processing toolkit used alongside Seurat in the workflow)

## Examples

```
conda create -n scf
```

## Evaluation signals

- Average internal-validation accuracy across the 13 tissues reaches approximately 95.16%, the value reported for the article's pre-trained models
- Variance risk R_var(θ) and average risk R_avg(θ) are both monitored and decrease over the 50–70 training epochs
- Softmax outputs of the 2-neuron classification layer are thresholded at 0.5 for malignant calls
- Retained checkpoint is the one with maximal accuracy on extra-domain (CopyKAT breast cancer) validation data, not merely on internal validation
- Environment matches README prerequisites: torch 1.13.1, python 3.9.16, CUDA 11.6

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks the spatial relationships among spots in ST data
- The model is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Some prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)

## Evidence

- [abstract] we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [readme] System: Ubuntu 18.04 python: 3.9.16 CUDA: 11.6 torch: 1.13.1: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
- [readme] Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment.: "Additionally, Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment."
- [intro] a reference-free malignant cell annotation algorithm with high accuracy, good generalization performance, and easy scalability to handle multiple data types is urgently required: "a reference-free malignant cell annotation algorithm with high accuracy, good generalization performance, and easy scalability to handle multiple data types is urgently required"
