---
name: classifier-accuracy-evaluation
description: Use when comparing a binary cell-state classifier (malignant vs normal)
  against published baselines on labeled gold standard datasets and unlabeled silver
  standard datasets with weighted accuracy metrics.
when_to_use_negative:
- Hematologic tumor data — the authors explicitly do not recommend Cancer-Finder for
  hematologic tumor data
- Datasets where ground-truth malignant/normal labels are unavailable and no silver-standard
  proxy labels can be constructed, since (TP+TN)/N cannot be computed
- Spatial transcriptomics evaluation requiring spatial-relationship-aware methods
  — Cancer-Finder overlooks spatial relationships among spots in ST data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Primary classifier under evaluation, run from pretrained checkpoint sc_pretrain_article.pkl
    with softmax threshold 0.5
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: CNV-based baseline (v1.1.0, default parameters), aneuploid=positive, diploid=negative,
    not.defined excluded
  repo: https://github.com/navinlabcode/copykat
- name: SCEVAN
  role: Baseline run via pipelineCNA(count_mtx) with unchanged settings
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CaSee
  role: Deep-learning baseline run on GPU with default YAML, five training/inference
    repeats
  repo: https://github.com/yuansh3354/CaSee
- name: ikarus
  role: ML baseline run with original pretrained model plus five retrains using Cancer-Finder's
    training-set strategy
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: SCANPY
  role: Preprocessing (pp.normalize_total, log1p) of all datasets for Cancer-Finder
    input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:5ba3b59640e05168cc3d976395796ec8ba0cd9494fd50ab902690b72ee595389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - CaSee
  - Cancer-Finder
  - CopyKAT
  - SCANPY
  - SCEVAN
  - ikarus
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# classifier-accuracy-evaluation

## Summary

Benchmark a malignant-cell annotation classifier (Cancer-Finder) against CNV-based and ML baselines (SCEVAN, CopyKAT, CaSee, ikarus) on gold/silver standard scRNA-seq datasets, computing accuracy and cell-count-weighted similarity to establish relative performance.

## When to use

Use when comparing a binary cell-state classifier (malignant vs normal) against published baselines on labeled gold standard datasets and unlabeled silver standard datasets with weighted accuracy metrics.

## When NOT to use

- Hematologic tumor data — the authors explicitly do not recommend Cancer-Finder for hematologic tumor data
- Datasets where ground-truth malignant/normal labels are unavailable and no silver-standard proxy labels can be constructed, since (TP+TN)/N cannot be computed
- Spatial transcriptomics evaluation requiring spatial-relationship-aware methods — Cancer-Finder overlooks spatial relationships among spots in ST data

## Inputs

- 10 external validation scRNA-seq datasets (gold: 10x PBMC, cancer cell lines; silver: datasets 3-10)
- Pre-trained Cancer-Finder checkpoint sc_pretrain_article.pkl
- Raw UMI count matrices formatted for CopyKAT (genes in rows, cells in columns)
- Count matrices for SCEVAN pipelineCNA input
- CaSee default YAML configuration (batch_size=128, max_epochs=20, lr=0.0005)
- ikarus pretrained model and Cancer-Finder training-set strategy for retraining

## Outputs

- fig3b_method_comparison.csv with per-run and weighted accuracies for all tools
- Per-dataset accuracy (gold) or similarity (silver) values as (TP+TN)/N
- Cell-count-weighted averages sum(ratio_i*N_i)/sum(N_i) per tool (target: Cancer-Finder 98.30% gold accuracy, 90.89% silver similarity)

## How to apply

Load the 10 external validation datasets (datasets 1–2 gold standard: 10x PBMC and cancer cell lines; datasets 3–10 silver standard) and preprocess with SCANPY 1.9.1 'pp.normalize_total' and 'log1p' log-transformed global scaling before classifier input. Run the pre-trained Cancer-Finder checkpoint (sc_pretrain_article.pkl) with a default softmax threshold of 0.5. Run CopyKAT v1.1.0 with defaults (id.type='S', ngene.chr=5, win.size=25, KS.cut=0.1, distance='euclidean', genome='hg20'), batching datasets with >10,000 cells by donor and counting 'aneuploid' as positive, 'diploid' as negative, excluding 'not.defined' cells; run SCEVAN via 'pipelineCNA(count_mtx)' unchanged; run CaSee on GPU with its default yaml (batch_size=128, max_epochs=20, lr=0.0005, tissue 'Unknown', marker 'null') over five training/inference repeats; run ikarus with its original pretrained model plus a retrained variant (five retrains using Cancer-Finder's training-set strategy). Repeat every tool five times per dataset, compute accuracy (gold) or similarity (silver) as (TP+TN)/N per dataset, then aggregate with the cell-count-weighted average sum(ratio_i*N_i)/sum(N_i), assembling per-run and weighted values into fig3b_method_comparison.csv. The rationale is that weighted aggregation prevents large datasets from dominating and repeated runs capture stochastic variability in training-based baselines.

## Related tools

- **Cancer-Finder** (Primary classifier under evaluation, run from pretrained checkpoint sc_pretrain_article.pkl with softmax threshold 0.5) — https://github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (CNV-based baseline (v1.1.0, default parameters), aneuploid=positive, diploid=negative, not.defined excluded) — https://github.com/navinlabcode/copykat
- **SCEVAN** (Baseline run via pipelineCNA(count_mtx) with unchanged settings) — https://github.com/AntonioDeFalco/SCEVAN
- **CaSee** (Deep-learning baseline run on GPU with default YAML, five training/inference repeats) — https://github.com/yuansh3354/CaSee
- **ikarus** (ML baseline run with original pretrained model plus five retrains using Cancer-Finder's training-set strategy) — https://github.com/BIMSBbioinfo/ikarus
- **SCANPY** (Preprocessing (pp.normalize_total, log1p) of all datasets for Cancer-Finder input)

## Examples

```
copykat.test <- copykat(rawmat=exp.rawdata, sam.name="test", id.type="S", ngene.chr=5, win.size=25, KS.cut=0.1, distance="euclidean")
```

## Evaluation signals

- Cell-count-weighted gold standard accuracy reproduces 98.30% for Cancer-Finder, exceeding SCEVAN, CaSee, CopyKAT, ikarus, and retrained ikarus
- Weighted silver standard average similarity reaches 90.89% for Cancer-Finder, exceeding all baselines
- CopyKAT runs exclude 'not.defined' cells consistently, with aneuploid counted as positive and diploid as negative
- Every tool is run five times per dataset and variance across repeats is reportable in fig3b_method_comparison.csv
- Preprocessing for Cancer-Finder inputs matches SCANPY 1.9.1 normalize_total + log1p log-transformed global scaling

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks spatial relationships among spots in ST data
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set
- Prediction errors may be resolvable only by increasing sequencing depth
- CopyKAT's cell line mode 'does not guarantee the success nor the accuracy' and uses a synthetic baseline not representing the published algorithm

## Evidence

- [discussion] Cancer-Finder achieved an accuracy of 98.30% in golden standard scRNA-seq datasets: "achieving an accuracy of 98.30% in golden standard datasets"
- [discussion] Cancer-Finder achieved a similarity of 90.89% in silver standard datasets: "a similarity of 90.89% in silver standard datasets"
- [abstract] Average accuracy across datasets: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [readme] CopyKAT input requirements: [evidence span withheld — anchor status: external_doc]
- [readme] CopyKAT cell line mode caveat: [evidence span withheld — anchor status: external_doc]
- [discussion] Hematologic tumor exclusion: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
