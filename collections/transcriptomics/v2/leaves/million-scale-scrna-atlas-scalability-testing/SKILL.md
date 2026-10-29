---
name: million-scale-scrna-atlas-scalability-testing
description: Use when benchmarking whether a single-cell auto-annotation or novel-cell-detection
  model scales to million-cell atlases, using a healthy-vs-disease atlas split where
  disease-only cell types serve as ground-truth novel classes.
when_to_use_negative:
- The goal is small or medium-scale benchmarking across diverse protocols and imbalance
  degrees rather than demonstrating million-scale scalability — scNovel's broader
  benchmark covers that separately.
- The query set contains no cell types absent from the reference, so there is no ground-truth
  novel class against which FPR95/AUROC/AUPR can be computed.
- Only multimodal data (e.g. scRNA-seq plus chromatin accessibility) is available
  — the article notes scNovel currently does not incorporate other biology information.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scNovel
  role: Deep-neural-network classifier with barcode preprocessing used for training
    on the reference and novel-cell query detection at million-cell scale
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Preprocessing of the atlas matrices via sc.pp.normalize_total(target_sum=1e4)
    and sc.pp.log1p before training/query
- name: PyTorch
  role: Backend for the scNovel network; CUDAToolkit-enabled PyTorch recommended for
    GPU execution at million-cell scale
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:464937dceea26eb99fd60deeaf54dde395041be7cbdfae9f6518271b59a39802
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# million-scale-scrna-atlas-scalability-testing

## Summary

Protocol for validating the scalability of a novel rare cell discovery method (scNovel) on the 1,462,702-cell COVID-19 immune atlas, holding out the 10 cell types present only in COVID groups as the novel query set against a 54-cell-type healthy reference.

## When to use

Use when benchmarking whether a single-cell auto-annotation or novel-cell-detection model scales to million-cell atlases, using a healthy-vs-disease atlas split where disease-only cell types serve as ground-truth novel classes.

## When NOT to use

- The goal is small or medium-scale benchmarking across diverse protocols and imbalance degrees rather than demonstrating million-scale scalability — scNovel's broader benchmark covers that separately.
- The query set contains no cell types absent from the reference, so there is no ground-truth novel class against which FPR95/AUROC/AUPR can be computed.
- Only multimodal data (e.g. scRNA-seq plus chromatin accessibility) is available — the article notes scNovel currently does not incorporate other biology information.

## Inputs

- COVID-19 immune atlas of 1,462,702 cells with cell-type labels (54 healthy reference types + 10 COVID-only types)
- Reference expression matrix and label vector in pandas/Scanpy-readable format (e.g. train_adata.csv, train_label.csv as in the scNovel tutorial)
- Query (held-out novel) expression matrix

## Outputs

- Per-cell-type FPR95, AUROC and AUPR table for the 10 novel cell types
- Per-cell novelty scores / probability predictions from the query stage
- Demonstration of runtime/scalability on a million-cell atlas

## How to apply

Load the COVID-19 immune atlas (1,462,702 cells) and preprocess both reference and query matrices with Scanpy using normalize_total to target_sum=1e4 followed by log1p, as in the scNovel tutorial. Split the atlas so the healthy group's 54 cell types form the reference training set and the 10 COVID-only cell types (Epi-AT2, Epi-Ciliated, Epi-Secretory, Epi-Squamous, Mast, Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2, DC-LAMP3) are held out as the novel query set. Train the scNovel deep-neural-network classifier on the reference with cross-entropy loss to output cell-type probabilities, then run the query stage in the S^SIM mode (simultaneous T_minus and T_plus barcode preprocessing, identified as best by ablation) with novelty threshold lambda = 0.5. Because of the million-cell scale, set processing_unit='gpu'. Compute FPR95, AUROC and AUPR per novel cell type against ground truth to quantify performance at scale.

## Related tools

- **scNovel** (Deep-neural-network classifier with barcode preprocessing used for training on the reference and novel-cell query detection at million-cell scale) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing of the atlas matrices via sc.pp.normalize_total(target_sum=1e4) and sc.pp.log1p before training/query)
- **PyTorch** (Backend for the scNovel network; CUDAToolkit-enabled PyTorch recommended for GPU execution at million-cell scale)

## Examples

```
pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- Epi cell types (Epi-Squamous, Epi-Secretory, Epi-Ciliated, Epi-AT2) and the 17-cell Mast type show low FPR95 with high AUROC and high AUPR, matching the reported pattern.
- Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2 and DC-LAMP3 show relatively lower performance because closely related subtypes exist in the reference set — a deviation from this pattern suggests a split or preprocessing error.
- The atlas cell count matches 1,462,702 and the reference set contains exactly the 54 healthy cell types.
- Averaged AUROC should remain above the 94% level reported for scNovel's benchmarking, consistent with its claimed scalability.
- The pipeline completes only with processing_unit='gpu' at million-cell scale; CPU-only runs that stall or fail indicate a scalability configuration problem.

## Limitations

- Performance degrades for novel types with closely related subtypes present in the reference (Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2, DC-LAMP3).
- The article acknowledges the score function could be improved, so absolute threshold calibration (lambda = 0.5) may need revisiting.
- No version numbers for scNovel (beyond pip install scNovel==1.1.0 in the README) or dataset accession IDs are given, complicating exact reproduction.
- The method uses only scRNA-seq; it cannot leverage chromatin accessibility or other modalities.

## Evidence

- [abstract] Million-scale validation on the COVID atlas: "We validate scNovel's performance on a million-scale dataset to illustrate the scalability of scNovel further."
- [intro] Training and query stage design: "During the training stage, we adopt cross-entropy loss to train the model to output cell types’ probability prediction after the classifier. During the query stage"
- [intro] Million-level atlas is the scalability benchmark: "our work is the first and only novel rare cell discovery tool validated on a million-level atlas"
- [readme] Scanpy preprocessing steps: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] GPU recommendation for large data: "We highly recommend setting as 'gpu' if your server supports."
- [discussion] Three novel macrophage subtypes in COVID data: "Within the context of the profoundly imbalanced clinical COVID-19 dataset, a comprehensive analysis has unveiled the existence of three promising novel subtypes of macrophages."
