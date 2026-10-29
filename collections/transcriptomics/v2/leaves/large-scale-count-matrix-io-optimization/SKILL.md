---
name: large-scale-count-matrix-io-optimization
description: Use when benchmarking inference time or memory of scRNA-seq annotation
  tools on matrices ranging from 100 to 1,000,000 cells, and you need to isolate file-parsing
  overhead from model runtime.
when_to_use_negative:
- Data are hematologic tumor samples — Cancer-Finder is explicitly not recommended
  there
- The analysis goal is annotation accuracy rather than runtime/memory scalability
- Datasets are small (<1,000 cells) where csv-parsing overhead is negligible and binary
  storage adds no measurable benefit
tools:
- name: Cancer-Finder
  role: Direct-inference annotator whose runtime and memory are benchmarked; run via
    infer.py with the pre-trained checkpoint
  repo: Patchouli-M/SequencingCancerFinder
- name: CaSee
  role: Transfer-learning comparator run on the GPU platform with default parameters
  repo: yuansh3354/CaSee
- name: CopyKAT
  role: CNV-based comparator run on CPU; datasets above 10,000 cells are batched per
    its documentation
  repo: navinlabcode/copykat
- name: SCEVAN
  role: Clonal-structure comparator run on CPU with default parameters
  repo: AntonioDeFalco/SCEVAN
- name: ikarus
  role: Machine-learning comparator run on CPU with default parameters
  repo: BIMSBbioinfo/ikarus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:baf7596499f4ec853745c5e460e23a6ccd15bcbae36565ccf1adaec01acd26e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - CaSee
  - Cancer-Finder
  - CopyKAT
  - SCEVAN
  - ikarus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Large-scale count matrix I/O optimization

## Summary

A benchmarking skill that quantifies how text-format (tsv/csv) parsing dominates wall-clock inference time for malignant cell annotation tools at 10^2–10^6 cell scale, and uses binary-stored matrices to separate parsing overhead from model compute.

## When to use

Use when benchmarking inference time or memory of scRNA-seq annotation tools on matrices ranging from 100 to 1,000,000 cells, and you need to isolate file-parsing overhead from model runtime.

## When NOT to use

- Data are hematologic tumor samples — Cancer-Finder is explicitly not recommended there
- The analysis goal is annotation accuracy rather than runtime/memory scalability
- Datasets are small (<1,000 cells) where csv-parsing overhead is negligible and binary storage adds no measurable benefit

## Inputs

- Raw count matrices (tsv/csv, genes x cells or cells x genes as accepted by infer.py) subsampled to 100, 1,000, 10,000, 100,000 and 1,000,000 cells
- Binary-stored copies of the 10,000-cell and 100,000-cell matrices for parsing-overhead comparison
- Pre-trained Cancer-Finder checkpoint (checkpoints/sc_pretrain_article.pkl)

## Outputs

- Per-size wall-clock inference times for Cancer-Finder, CaSee, CopyKAT, SCEVAN and ikarus
- Peak memory consumption per tool and dataset size
- Combined benchmark table saved as inference_benchmark.csv (cf. Fig. 3c,d)

## How to apply

Subsample input count matrices to 100, 1,000, 10,000, 100,000 and 1,000,000 cells in tsv/csv format supported by Cancer-Finder's infer.py. Run inference on a fixed GPU platform (Intel Xeon Silver 4216 @ 2.10 GHz, 16 cores, NVIDIA RTX 3090, 512 GB RAM), recording wall-clock time from data reading to output; expect roughly 1.39 s, 2.53 s, 58.65 s, 3,903.41 s and 38,903.92 s. To quantify csv-parsing overhead, repeat the 10,000-cell and 100,000-cell runs with binary-stored matrices, which collapses times to about 4.15 s and 39.46 s, showing that text parsing dominates at scale. Benchmark competing tools on matched hardware (CaSee on GPU; CopyKAT, SCEVAN and ikarus on CPU) with default parameters, batching datasets above 10,000 cells for CopyKAT per its documentation, and record peak memory alongside time.

## Related tools

- **Cancer-Finder** (Direct-inference annotator whose runtime and memory are benchmarked; run via infer.py with the pre-trained checkpoint) — Patchouli-M/SequencingCancerFinder
- **CaSee** (Transfer-learning comparator run on the GPU platform with default parameters) — yuansh3354/CaSee
- **CopyKAT** (CNV-based comparator run on CPU; datasets above 10,000 cells are batched per its documentation) — navinlabcode/copykat
- **SCEVAN** (Clonal-structure comparator run on CPU with default parameters) — AntonioDeFalco/SCEVAN
- **ikarus** (Machine-learning comparator run on CPU with default parameters) — BIMSBbioinfo/ikarus

## Examples

```
Rscript -e 'library(devtools); install_github("navinlabcode/copykat"); copykat.test <- copykat(rawmat=exp.rawdata, sam.name="test")'
```

## Evaluation signals

- Cancer-Finder inference times fall on the order of 1.39 s / 2.53 s / 58.65 s / 3,903.41 s / 38,903.92 s for 100 / 1e3 / 1e4 / 1e5 / 1e6 cells
- Binary-stored matrices reduce the 10,000-cell and 100,000-cell runs to approximately 4.15 s and 39.46 s, confirming csv parsing as the dominant cost
- Cancer-Finder shows the fastest times and significantly reduced memory consumption versus CaSee, CopyKAT, SCEVAN and ikarus across all sizes
- Combined per-size table is saved as inference_benchmark.csv matching Fig. 3c,d structure
- CopyKAT runs on datasets above 10,000 cells were executed in batches per its documentation recommendation

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Runtimes are hardware-dependent; reported timings assume the specified GPU platform (Xeon Silver 4216, RTX 3090, 512 GB RAM)
- Cancer-Finder overlooks spatial relationships among spots when applied to ST data

## Evidence

- [other] Inference times on 100 to 1,000,000 cells with csv parsing overhead quantification: [evidence span withheld — anchor status: not_found]
- [other] Benchmark expectation across dataset sizes: [evidence span withheld — anchor status: not_found]
- [readme] Cancer-Finder as a rapid direct-inference annotator (README): "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
- [readme] CopyKAT benchmark input preparation: [evidence span withheld — anchor status: external_doc]
