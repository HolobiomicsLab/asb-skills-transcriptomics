---
name: fclip-seq-dsrna-region-processing
description: Use when validating in silico dsRNA predictions against experimental
  J2 fCLIP-seq data, or when computing the fraction of captured dsRNAs overlapped
  by dsRID predictions (expected 73.3%, 7803 regions).
when_to_use_negative:
- Input is raw PacBio long-read RNA-seq BAM data intended for dsRNA prediction rather
  than interval intersection with existing predicted regions
- The goal is to run dsRID's training or whole-genome prediction pipeline (dsRID.sh,
  extract_whole.py, model_predict.py) instead of validating predictions via interval
  overlap
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: dsRID
  role: Provides the combined predicted dsRNA regions (data/dsRID_novel_dsRNA.bed)
    that are intersected with the J2 fCLIP-seq HeLa dsRNA regions
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Used to load both BED interval sets and perform the genomic interval intersection
    and summary computations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:48d799e2057f5a714202599311c499196ebcf4d8ab4294f2d0e954601454603c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fclip-seq-dsrna-region-processing

## Summary

Reproduce the dsRID validation by intersecting J2 fCLIP-seq-captured HeLa dsRNA regions (Kim et al. 2018) with dsRID-predicted dsRNA regions from the gxiaolab/dsRID repository, quantifying the overlap fraction (73.3%) and stratifying hits into EER-based and novel long dsRNA categories.

## When to use

Use when validating in silico dsRNA predictions against experimental J2 fCLIP-seq data, or when computing the fraction of captured dsRNAs overlapped by dsRID predictions (expected 73.3%, 7803 regions).

## When NOT to use

- Input is raw PacBio long-read RNA-seq BAM data intended for dsRNA prediction rather than interval intersection with existing predicted regions
- The goal is to run dsRID's training or whole-genome prediction pipeline (dsRID.sh, extract_whole.py, model_predict.py) instead of validating predictions via interval overlap

## Inputs

- dsRID predicted dsRNA regions as data/dsRID_novel_dsRNA.bed from the gxiaolab/dsRID GitHub repository
- J2 fCLIP-seq HeLa dsRNA regions (Kim et al. 2018) as a BED-format genomic interval set

## Outputs

- j2_dsrid_overlap.tsv containing per-region overlap coordinates and summary counts (7803 overlapping regions; 5036 EER-based; 2767 novel; 73.3% overlap fraction)
- Overlap fraction of J2 fCLIP-seq-captured dsRNAs overlapping dsRID-predicted dsRNAs (expected 73.3%)

## How to apply

Obtain the combined dsRID predicted dsRNA regions from data/dsRID_novel_dsRNA.bed in the gxiaolab/dsRID GitHub repository and the J2 fCLIP-seq HeLa dsRNA regions (Kim et al. 2018) as a BED-format genomic interval set. Load both interval sets in Python 3.x and perform a genomic interval intersection (bedtools/pybedtools-style intersect) to identify J2-captured dsRNA regions that overlap dsRID-predicted dsRNAs. Count the overlapping regions (expected 7803) and compute the overlap fraction of J2-captured dsRNAs (expected 73.3%). Stratify the 7803 overlapping regions into EER-based overlaps (expected 5036) and novel dsRID predictions (expected 2767). Write per-region overlap coordinates plus summary counts to j2_dsrid_overlap.tsv; validation requires reproducing the 73.3% overlap fraction and the 5036/2767 stratification shown in Fig. 3F.

## Related tools

- **dsRID** (Provides the combined predicted dsRNA regions (data/dsRID_novel_dsRNA.bed) that are intersected with the J2 fCLIP-seq HeLa dsRNA regions) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Used to load both BED interval sets and perform the genomic interval intersection and summary computations)

## Examples

```
python src/j2_dsrid_overlap.py --dsrid data/dsRID_novel_dsRNA.bed --j2 hela_j2_fclipseq.bed --output j2_dsrid_overlap.tsv
```

## Evaluation signals

- Computed overlap fraction equals the reported 73.3% of J2 fCLIP-seq-captured dsRNAs overlapping dsRID dsRNAs
- Number of overlapping regions equals 7803
- EER-based stratification count equals 5036 among the 7803 overlapping regions
- Novel dsRID prediction count equals 2767 among the 7803 overlapping regions
- j2_dsrid_overlap.tsv contains per-region overlap coordinates plus the summary counts, matching Fig. 3F

## Limitations

- Validation compares across cell type differences (HeLa vs. the datasets dsRID predictions were built from), so overlap reflects cross-cell-type concordance rather than same-sample agreement
- No version of dsRID or specific package versions are stated in the methods, so reproducing exact counts may depend on the repository state of data/dsRID_novel_dsRNA.bed

## Evidence

- [results] results: dsRID predicted regions across all samples: "dsRID predicted regions across all samples are available under data/dsRID_novel_dsRNA.bed"
- [other] workflow: interval intersection: [evidence span withheld — anchor status: not_found]
- [other] workflow: expected counts: [evidence span withheld — anchor status: not_found]
- [other] workflow: stratification: [evidence span withheld — anchor status: not_found]
- [readme] readme: prerequisites: "To get started with dsRID, you will need the following prerequisites: PacBio long-read RNA-seq data; Python version 3.x or higher; Required packages listed in the requirements.txt file"
