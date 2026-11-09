---
name: imbalanced-classification-metric-evaluation
description: Use when evaluating a cell-type auto-annotation or novelty-detection
  model on scRNA-seq query sets where novel classes are rare, so accuracy is mis and
  AUROC (main metric), AUPR and FPR95 must be reported per held-out class.
when_to_use_negative:
- Balanced classification tasks with no held-out novel classes, where plain accuracy
  or macro-F1 is sufficient and novelty-scoring overhead adds nothing
- Evaluation of standard top-1 cell annotation accuracy only — this skill measures
  novel-class detection (AUROC/AUPR/FPR95), not annotation correctness of known types
- Datasets where the query contains no rare or novel populations, making FPR95 on
  near-absent positives undefined or unstable
tools:
- name: scNovel
  role: Two-stage (training + query) deep-neural-network model whose novelty scores
    are the inputs to the per-class AUROC/AUPR/FPR95 evaluation; pip-installable as
    scNovel==1.1.0
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Preprocessing of reference/query matrices (sc.pp.normalize_total to target_sum=1e4
    followed by sc.pp.log1p) so metrics are computed on comparably normalized data
- name: PyTorch
  role: Backend for training the cross-entropy classifier that produces the probability
    predictions evaluated by the metrics; GPU recommended at million-cell scale
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:3eaf472736779ffb6e7f81316acfeda9fcaee0d8c7579f5f79b6492965eb066d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# imbalanced-classification-metric-evaluation

## Summary

Evaluates novel rare cell-type detection under severe class imbalance by computing per-novel-class AUROC, AUPR and FPR95 rather than overall accuracy, as done for scNovel across benchmark datasets and the million-cell COVID-19 atlas.

## When to use

Use when evaluating a cell-type auto-annotation or novelty-detection model on scRNA-seq query sets where novel classes are rare, so accuracy is misleading and AUROC (main metric), AUPR and FPR95 must be reported per held-out class.

## When NOT to use

- Balanced classification tasks with no held-out novel classes, where plain accuracy or macro-F1 is sufficient and novelty-scoring overhead adds nothing
- Evaluation of standard top-1 cell annotation accuracy only — this skill measures novel-class detection (AUROC/AUPR/FPR95), not annotation correctness of known types
- Datasets where the query contains no rare or novel populations, making FPR95 on near-absent positives undefined or unstable

## Inputs

- Reference (training) expression matrix plus label vector, e.g. CSV files read via sc.read_csv and preprocessed with sc.pp.normalize_total(target_sum=1e4) then sc.pp.log1p
- Query expression matrix containing held-out novel cell types with ground-truth labels
- Novelty scores / cell-type probability predictions from scNovel (e.g. the S^MSP, S^ODIN, S^SEQ or S^SIM detection scores)

## Outputs

- Per-novel-class metric table of FPR95, AUROC and AUPR against ground-truth labels
- Averaged AUROC (plus AUPR and FPR95) across datasets and C^novel = 1, 2, 3, 4 settings, comparable to Appendix Table 4 and Figure 3A
- Mode-comparison result confirming which detection score mode (S^MSP, S^ODIN, S^SEQ, S^SIM) attains the highest averaged AUROC

## How to apply

Hold out C^novel = 1–4 classes as the novel query set, train the scNovel classifier on reference cells with cross-entropy loss (query threshold lambda = 0.5, best mode S^SIM per the Appendix Table 4 / Figure 3A ablation), then score query cells and compute AUROC (the main metric), AUPR and FPR95 separately for each held-out novel class against ground-truth labels. For million-scale imbalanced data (e.g. the 1,462,702-cell COVID-19 atlas with 10 COVID-only cell types), compute the three metrics per class — Epi-Squamous, Epi-Secretory, Epi-Ciliated, Epi-AT2 and the 17-cell Mast should show low FPR95 with high AUROC/AUPR, whereas Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2 and DC-LAMP3 show lower values. When comparing modes or methods, average AUROC across datasets and C^novel settings and use that as the decision criterion. Sanity-check the metric pipeline by reproducing the reported C^novel=1 numbers: 96.33% AUROC, 99.76% AUPR, 22.82% FPR95.

## Related tools

- **scNovel** (Two-stage (training + query) deep-neural-network model whose novelty scores are the inputs to the per-class AUROC/AUPR/FPR95 evaluation; pip-installable as scNovel==1.1.0) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing of reference/query matrices (sc.pp.normalize_total to target_sum=1e4 followed by sc.pp.log1p) so metrics are computed on comparably normalized data)
- **PyTorch** (Backend for training the cross-entropy classifier that produces the probability predictions evaluated by the metrics; GPU recommended at million-cell scale)

## Examples

```
import scNovel as sr; pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit='gpu')
```

## Evaluation signals

- The C^novel=1 benchmark must reproduce 96.33% AUROC, 99.76% AUPR and 22.82% FPR95
- Averaged AUROC across benchmarks should exceed 94%, up to 16.26% higher than the second-best method
- In the mode ablation, S^SIM must attain the highest averaged AUROC among S^MSP, S^ODIN, S^SEQ and S^SIM
- On the COVID-19 atlas, Epi-Squamous, Epi-Secretory, Epi-Ciliated, Epi-AT2 and the 17-cell Mast should show low FPR95 with high AUROC and high AUPR, while Macro-DNAJB1, Macro-EREG, Neu-FGF23, Neu-RSAD2 and DC-LAMP3 show relatively lower performance
- Metrics must be computed separately for each of the 10 novel cell types against ground-truth labels, never as a single pooled accuracy

## Limitations

- Novel classes with closely related subtypes present in the reference (macrophage, neutrophil and dendritic subtypes such as DC-LAMP3) yield relatively lower AUROC/AUPR and higher FPR95, so metric values are class-composition dependent
- The score function design is acknowledged as improvable, so absolute metric values may change with future scNovel versions
- Results depend on the query threshold lambda = 0.5 and on the chosen detection mode (S^SIM is the best mode per the ablation); other settings will shift FPR95/AUROC trade-offs

## Evidence

- [abstract] scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance(the only one method whose averaged AUROC results are above 94%: "scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance(the only one method whose averaged AUROC results are above 94%"
- [intro] it is essential for a tool to not only precisely categorize cell types for query cells but also identify novel cell types that are not present in the reference dataset: "it is essential for a tool to not only precisely categorize cell types for query cells but also identify novel cell types that are not present in the reference dataset"
- [readme] scRare especially obtains an outperformance on novel rare cell detection((up to 16.26% more AUROC compared to the second-best method): "scRare especially obtains an outperformance on novel rare cell detection((up to 16.26% more AUROC compared to the second-best method)"
