---
name: svm-cell-type-classification
description: Use when comparing how well alternative single-cell multiomics embeddings
  support supervised cell-type identification — e.g. testing whether scME's joint
  representation beats Seurat, totalVI, MOFA+, or unimodal embeddings on labeled CITE-seq
  data.
when_to_use_negative:
- Embeddings have not yet been generated — this skill evaluates precomputed embeddings
  and does not train scME; run scME (python scme.py) first.
- Cell type labels are unavailable, since 5-fold cross-validated supervised SVM classification
  cannot be performed without ground-truth annotations.
- The goal is unsupervised discovery of new cell populations by clustering rather
  than classification against known labels.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scME
  role: Generates the joint multimodal cell embeddings (zm) whose classification performance
    is evaluated by the SVM
  repo: https://github.com/bucky527/scME
- name: scikit-learn
  role: Provides the SVM classifier, 5-fold cross-validation, and the accuracy/precision/recall/F1/MCC
    metric functions
- name: pandas
  role: Assembles and saves the combined metrics table as svm_classification_metrics.csv
- name: numpy
  role: Handles embedding array manipulation and fold-wise metric averaging
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:8467fbe74dc97d63c27459f714869f6c56b4f6757f150761ffade9eec09a24b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - PyTorch
  - numpy
  - pandas
  - scME
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# svm-cell-type-classification

## Summary

Benchmark cell-embedding quality by training 5-fold cross-validated SVM classifiers (scikit-learn.svm) on scME multimodal embeddings versus unimodal RNA-only, protein-only, and other integration-method embeddings, reporting accuracy, precision, recall, F1, and MCC on the BM2 and SLN CITE-seq datasets.

## When to use

Use when comparing how well alternative single-cell multiomics embeddings support supervised cell-type identification — e.g. testing whether scME's joint representation beats Seurat, totalVI, MOFA+, or unimodal embeddings on labeled CITE-seq data.

## When NOT to use

- Embeddings have not yet been generated — this skill evaluates precomputed embeddings and does not train scME; run scME (python scme.py) first.
- Cell type labels are unavailable, since 5-fold cross-validated supervised SVM classification cannot be performed without ground-truth annotations.
- The goal is unsupervised discovery of new cell populations by clustering rather than classification against known labels.

## Inputs

- scME multimodal cell embeddings (zm from scme.inference) for the BM2 and SLN CITE-seq datasets
- Unimodal RNA-only and protein-only cell embeddings for the same datasets
- Embeddings from comparison methods (Seurat, totalVI, MOFA+)
- Cell type labels for BM2 and SLN cells
- MOFA+ baseline F1 scores from article Table 5 (0.6337 for BM2, 0.6079 for SLN)

## Outputs

- svm_classification_metrics.csv containing accuracy, precision, recall, F1, and MCC averaged over 5 folds for each embedding and dataset
- Side-by-side comparison table of multimodal vs unimodal vs baseline-method classification performance

## How to apply

Load the precomputed cell embeddings — the scME multimodal zm output, unimodal RNA-only and protein-only embeddings, and embeddings from Seurat, totalVI, and MOFA+ — for the BM2 and SLN datasets, together with cell type labels. For each embedding, train an scikit-learn SVM classifier under 5-fold cross-validation, computing accuracy, precision, recall, F1, and Matthews correlation coefficient (MCC) on each held-out fold and averaging across folds. Assemble the metrics side by side in a comparison table, adding the MOFA+ baseline F1 scores from article Table 5 (0.6337 for BM2, 0.6079 for SLN). Save the combined table as svm_classification_metrics.csv. Validate that scME multimodal F1/MCC exceed the unimodal embeddings, that scME outperforms Seurat, totalVI, and MOFA+ on all metrics, and that values match article Fig. 4 and Table 5.

## Related tools

- **scME** (Generates the joint multimodal cell embeddings (zm) whose classification performance is evaluated by the SVM) — https://github.com/bucky527/scME
- **scikit-learn** (Provides the SVM classifier, 5-fold cross-validation, and the accuracy/precision/recall/F1/MCC metric functions)
- **pandas** (Assembles and saves the combined metrics table as svm_classification_metrics.csv)
- **numpy** (Handles embedding array manipulation and fold-wise metric averaging)

## Examples

```
zm = scme.inference(rna_data, protein_data); cross_validate(SVC(), zm, labels, cv=5, scoring=('accuracy','precision_macro','recall_macro','f1_macro','matthews_corrcoef'))
```

## Evaluation signals

- scME multimodal F1 and MCC must exceed the unimodal RNA-only and protein-only embeddings on both BM2 and SLN
- scME outperforms Seurat, totalVI, and MOFA+ on all five metrics (accuracy, precision, recall, F1, MCC)
- MOFA+ baseline F1 values reproduce the Table 5 figures: 0.6337 (BM2) and 0.6079 (SLN)
- Metrics are averaged over 5 cross-validation folds and match the values in article Fig. 4 and Table 5
- svm_classification_metrics.csv contains all five metrics for every embedding/dataset combination

## Limitations

- Requires precomputed embeddings and ground-truth cell-type labels; cannot be applied to unlabeled data or before embedding generation.
- Comparison validity depends on a consistent 5-fold cross-validated SVM setup across all methods; MOFA+ baselines are taken from Table 5 rather than re-run in this workflow.
- Validation anchors (Fig. 4, Table 5, 0.6337/0.6079 F1) are specific to the BM2 and SLN CITE-seq benchmarking datasets.

## Evidence

- [abstract] joint representation improves both clustering and classification: "the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification"
- [abstract] scME beats other integration methods: "scME can generate a better joint representation of multiple modalities than those generated by other single-cell multiomics integration algorithms"
- [methods] embeddings for CITE-seq: "You can use scme.py to easily obtain cell embeddings for CITE-seq data"
- [readme] scikit-learn in dependency stack: "pip3 install numpy scipy pandas scikit-learn pyro-ppl matplotlib scanpy anndata scvi-tools"
