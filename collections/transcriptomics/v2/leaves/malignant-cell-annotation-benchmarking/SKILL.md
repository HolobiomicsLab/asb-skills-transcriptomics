---
name: malignant-cell-annotation-benchmarking
description: Use when evaluating whether a new or existing malignant-cell annotation
  method outperforms CNV-based and machine-learning baselines on scRNA-seq tumor microenvironment
  datasets.
when_to_use_negative:
- Input scRNA-seq data is from hematologic tumors — Cancer-Finder is explicitly not
  recommended for hematologic tumor data
- Data are spatial transcriptomics spots without retraining Cancer-Finder on spatial
  training data, since the spatial variant requires substituting ST training sets
- Only one annotated reference dataset is available and generalization performance
  cannot be assessed against external validation data
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: Domain generalization deep-learning method under evaluation for malignant
    cell identification (sc_pretrain_article.pkl, softmax threshold 0.5)
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: CNV-based baseline run with default parameters (id.type='S', ngene.chr=5,
    KS.cut=0.1), counting aneuploid as positive
  repo: https://github.com/navinlabcode/copykat
- name: SCEVAN
  role: Baseline run via pipelineCNA(count_mtx) with unchanged settings
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CaSee
  role: Deep-learning baseline run on GPU with default yaml, five training/inference
    repeats
  repo: https://github.com/yuansh3354/CaSee
- name: ikarus
  role: Machine-learning baseline run with the original pretrained model and additionally
    retrained five times using Cancer-Finder's training set strategy
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: SCANPY
  role: 'Preprocessing: pp.normalize_total and log1p log-transformed global scaling
    applied to all datasets before annotation'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:cbede96e0a51f48e25f3d2c1cfd0f93865504a0a9e4ce0f1219c91c622ff9696
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# malignant-cell-annotation-benchmarking

## Summary

A benchmarking protocol that compares Cancer-Finder against SCEVAN, CopyKAT, CaSee, and ikarus for malignant cell annotation on scRNA-seq gold/silver standard datasets, using cell-count-weighted accuracy (gold) and similarity (silver) as metrics.

## When to use

Use when evaluating whether a new or existing malignant-cell annotation method outperforms CNV-based and machine-learning baselines on scRNA-seq tumor microenvironment datasets.

## When NOT to use

- Input scRNA-seq data is from hematologic tumors — Cancer-Finder is explicitly not recommended for hematologic tumor data
- Data are spatial transcriptomics spots without retraining Cancer-Finder on spatial training data, since the spatial variant requires substituting ST training sets
- Only one annotated reference dataset is available and generalization performance cannot be assessed against external validation data

## Inputs

- 10 external validation scRNA-seq datasets (datasets 1-2 gold standard: 10x PBMC and cancer cell lines; datasets 3-10 silver standard)
- Raw UMI count matrices (genes x cells) for CopyKAT/SCEVAN
- Cancer-Finder pre-trained checkpoint sc_pretrain_article.pkl
- CaSee default yaml configuration (batch_size=128, max_epochs=20, lr=0.0005, tissue 'Unknown', marker 'null')

## Outputs

- Per-run accuracy/similarity values per tool per dataset
- Cell-count-weighted average accuracy/similarity per tool (e.g. fig3b_method_comparison.csv)
- Method comparison figure data (Fig. 3b style)

## How to apply

Preprocess all datasets identically with SCANPY 1.9.1 pp.normalize_total and log1p before applying each annotator. Run the pre-trained Cancer-Finder checkpoint (sc_pretrain_article.pkl) with softmax threshold 0.5; CopyKAT v1.1.0 with defaults (id.type='S', ngene.chr=5, win.size=25, KS.cut=0.1, distance='euclidean'), batching datasets with >10,000 cells by donor and excluding 'not.defined' cells; SCEVAN via pipelineCNA(count_mtx); CaSee on GPU with its default yaml (batch_size=128, max_epochs=20, lr=0.0005); and ikarus with both the original pretrained model and a version retrained five times using Cancer-Finder's training strategy. Repeat every tool five times per dataset, then compute per-dataset accuracy as (TP+TN)/N and the cell-count-weighted average sum(ratio_i*N_i)/sum(N_i), assembling results into a comparison table (e.g. fig3b_method_comparison.csv).

## Related tools

- **Cancer-Finder** (Domain generalization deep-learning method under evaluation for malignant cell identification (sc_pretrain_article.pkl, softmax threshold 0.5)) — https://github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (CNV-based baseline run with default parameters (id.type='S', ngene.chr=5, KS.cut=0.1), counting aneuploid as positive) — https://github.com/navinlabcode/copykat
- **SCEVAN** (Baseline run via pipelineCNA(count_mtx) with unchanged settings) — https://github.com/AntonioDeFalco/SCEVAN
- **CaSee** (Deep-learning baseline run on GPU with default yaml, five training/inference repeats) — https://github.com/yuansh3354/CaSee
- **ikarus** (Machine-learning baseline run with the original pretrained model and additionally retrained five times using Cancer-Finder's training set strategy) — https://github.com/BIMSBbioinfo/ikarus
- **SCANPY** (Preprocessing: pp.normalize_total and log1p log-transformed global scaling applied to all datasets before annotation)

## Examples

```
copykat.test <- copykat(rawmat=exp.rawdata, sam.name="test")
```

## Evaluation signals

- Cancer-Finder should reach 98.30% cell-count-weighted accuracy on the two gold standard datasets (PBMC and cancer cell lines), exceeding all baselines
- Cancer-Finder should reach 90.89% weighted average similarity on the silver standard datasets (3-10)
- All five tools must be run five times per dataset and the weighted average computed as sum(ratio_i*N_i)/sum(N_i) per dataset
- CopyKAT 'not.defined' cells must be excluded from accuracy computation; aneuploid counted as positive and diploid as negative
- Reproduced per-tool rankings should match the Fig. 3b comparison order (Cancer-Finder > SCEVAN, CaSee, CopyKAT, ikarus, retrained ikarus)

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks spatial relationships among spots in ST data
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set
- CopyKAT's cell.line mode uses a synthetic baseline that does not represent the published algorithm and does not guarantee success or accuracy

## Evidence

- [abstract] rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] achieving an accuracy of 98.30% in golden standard datasets: "achieving an accuracy of 98.30% in golden standard datasets"
- [readme] CopyKAT vignette invocation: [evidence span withheld — anchor status: external_doc]
- [readme] ikarus task definition: [evidence span withheld — anchor status: external_doc]
- [readme] SCEVAN capability description: [evidence span withheld — anchor status: external_doc]
