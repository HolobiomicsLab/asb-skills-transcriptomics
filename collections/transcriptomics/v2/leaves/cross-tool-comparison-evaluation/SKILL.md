---
name: cross-tool-comparison-evaluation
description: Use when evaluating a malignant cell annotation method against published
  baselines on shared gold/silver standard scRNA-seq datasets, or when benchmarking
  inference runtime and memory scaling with dataset size.
when_to_use_negative:
- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- The task is deconvolving spot-level cell composition in spatial data rather than
  annotating individual malignant cells (use cell2location or CARD instead)
- Only a single method needs to be run without comparison against the SCEVAN/CopyKAT/CaSee/ikarus
  baselines
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Primary domain generalization model under evaluation; run via infer.py with
    pre-trained checkpoint sc_pretrain_article.pkl
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: CNV-based baseline; run with default parameters, batching datasets with >10,000
    cells by donor
  repo: https://github.com/navinlabcode/copykat
- name: SCEVAN
  role: Baseline; run via pipelineCNA(count_mtx) with unchanged settings on CPU
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CaSee
  role: Deep-learning baseline; run on GPU with default yaml (batch_size=128, max_epochs=20,
    lr=0.0005)
  repo: https://github.com/yuansh3354/CaSee
- name: ikarus
  role: Baseline; run with the original pretrained model and retrained five times
    with Cancer-Finder's training-set strategy
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: SCANPY
  role: Preprocessing (pp.normalize_total and log1p) of Cancer-Finder inputs
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:fe89d687f60b88e023e7591deb87753570bb371a91ce408bf76fbbcda98899a1
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-tool-comparison-evaluation

## Summary

Benchmark protocol comparing Cancer-Finder against SCEVAN, CopyKAT, CaSee, and ikarus on gold/silver standard scRNA-seq datasets for malignant cell annotation accuracy and on subsampled cell counts (100 to 1,000,000 cells) for inference speed and memory, reproducing the Fig. 3 accuracy and runtime comparisons.

## When to use

Use when evaluating a malignant cell annotation method against published baselines on shared gold/silver standard scRNA-seq datasets, or when benchmarking inference runtime and memory scaling with dataset size.

## When NOT to use

- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- The task is deconvolving spot-level cell composition in spatial data rather than annotating individual malignant cells (use cell2location or CARD instead)
- Only a single method needs to be run without comparison against the SCEVAN/CopyKAT/CaSee/ikarus baselines

## Inputs

- 10 external validation scRNA-seq datasets (gold standard: 10x PBMC and cancer cell lines; silver standard: datasets 3-10)
- Raw UMI count matrices in tsv/csv format for CopyKAT (genes in rows, cells in columns)
- Subsampled count matrices of 100, 1,000, 10,000, 100,000 and 1,000,000 cells for the speed benchmark
- Pre-trained Cancer-Finder checkpoint (checkpoints/sc_pretrain_article.pkl)

## Outputs

- fig3b_method_comparison.csv with per-run and weighted accuracies/similarities for all tools
- inference_benchmark.csv with per-size inference times and memory consumption for all five tools
- Cell-level malignant/normal predictions per tool per dataset

## How to apply

Preprocess inputs with SCANPY 1.9.1 'pp.normalize_total' and 'log1p' scaling for Cancer-Finder, then run its pre-trained checkpoint (sc_pretrain_article.pkl) with a softmax threshold of 0.5. Run CopyKAT v1.1.0 with default parameters (id.type='S', ngene.chr=5, win.size=25, KS.cut=0.1, distance='euclidean', genome='hg20'), counting 'aneuploid' as positive and excluding 'not.defined' cells; run SCEVAN via 'pipelineCNA(count_mtx)'; run CaSee on GPU with its default yaml (batch_size=128, max_epochs=20, lr=0.0005); run ikarus with its original pretrained model and additionally retrain it five times with the same training-set strategy as Cancer-Finder. For speed benchmarking, run infer.py on subsets of 100, 1,000, 10,000, 100,000 and 1,000,000 cells on an NVIDIA RTX 3090 platform, recording wall-clock time from data reading to output plus peak memory, batching datasets above 10,000 cells for CopyKAT. Repeat every tool five times per dataset, compute accuracy (gold) or similarity (silver) as (TP+TN)/N, and aggregate with the cell-count-weighted average sum(ratio_i*N_i)/sum(N_i).

## Related tools

- **Cancer-Finder** (Primary domain generalization model under evaluation; run via infer.py with pre-trained checkpoint sc_pretrain_article.pkl) — https://github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (CNV-based baseline; run with default parameters, batching datasets with >10,000 cells by donor) — https://github.com/navinlabcode/copykat
- **SCEVAN** (Baseline; run via pipelineCNA(count_mtx) with unchanged settings on CPU) — https://github.com/AntonioDeFalco/SCEVAN
- **CaSee** (Deep-learning baseline; run on GPU with default yaml (batch_size=128, max_epochs=20, lr=0.0005)) — https://github.com/yuansh3354/CaSee
- **ikarus** (Baseline; run with the original pretrained model and retrained five times with Cancer-Finder's training-set strategy) — https://github.com/BIMSBbioinfo/ikarus
- **SCANPY** (Preprocessing (pp.normalize_total and log1p) of Cancer-Finder inputs)

## Examples

```
python infer.py --checkpoint checkpoints/sc_pretrain_article.pkl --input subset_10000.csv && Rscript -e 'copykat.test <- copykat(rawmat=exp.rawdata, sam.name="test")'
```

## Evaluation signals

- Cancer-Finder should reach 98.30% weighted overall accuracy on gold standard datasets and 90.89% weighted similarity on silver standard datasets, exceeding all baselines (Fig. 3b)
- Inference times on 100/1,000/10,000/100,000/1,000,000 cells should be on the order of 1.39 s / 2.53 s / 58.65 s / 3,903.41 s / 38,903.92 s (Fig. 3c,d)
- Binary-stored 10,000- and 100,000-cell runs should drop to ~4.15 s and ~39.46 s, quantifying csv-parsing overhead
- Cancer-Finder should show the fastest per-size inference time and significantly reduced memory consumption relative to all four baselines
- Every tool should be run five times per dataset and averages must use the cell-count-weighted formula sum(ratio_i*N_i)/sum(N_i)

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks spatial relationships among spots in ST data
- CopyKAT requires batching datasets above 10,000 cells by donor, and baselines run on different platforms (CopyKAT/SCEVAN/ikarus on CPU, CaSee/Cancer-Finder on GPU), so runtime comparisons must account for platform
- Robustness holds only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)

## Evidence

- [results] weighted accuracies with Cancer-Finder at 98.30% gold and 90.89% silver: [evidence span withheld — anchor status: not_found]
- [results] inference times across cell counts: [evidence span withheld — anchor status: not_found]
- [readme] CopyKAT default parameters: [evidence span withheld — anchor status: external_doc]
- [discussion] not recommended for hematologic tumors: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
