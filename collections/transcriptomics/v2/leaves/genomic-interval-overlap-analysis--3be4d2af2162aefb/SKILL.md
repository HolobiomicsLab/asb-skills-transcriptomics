---
name: genomic-interval-overlap-analysis
description: Use when validating predicted genomic regions (e.g., dsRID dsRNA BED
  files) against an experimentally derived interval set (e.g., J2 fCLIP-seq peaks)
  by intersecting coordinates and quantifying overlap fractions.
when_to_use_negative:
- Input regions are already a merged/joined feature table rather than two independent
  BED interval sets needing coordinate intersection
- The goal is dsRNA region prediction from long-read RNA-seq data itself (use dsRID's
  extract_whole.py/model_predict.py pipeline instead of overlap validation)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dsRID
  role: Source of the predicted dsRNA region BED file (data/dsRID_novel_dsRNA.bed)
    being validated against experimental J2 fCLIP-seq intervals
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Python 3.x runtime for loading BED interval sets and performing bedtools/pybedtools-style
    intersect to compute overlaps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:452fc2ccac6d9dd5b15ed48fc49610e378123047fb829efec22b2f8a19ffdf1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - dsRID
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genomic-interval-overlap-analysis

## Summary

Performs BED-format genomic interval intersection between experimentally captured dsRNAs (e.g., J2 fCLIP-seq from Kim et al. 2018 in HeLa) and dsRID-predicted dsRNA regions to validate the in silico predictions, computing overlap counts, stratification (EER-based vs novel), and overlap fractions.

## When to use

Use when validating predicted genomic regions (e.g., dsRID dsRNA BED files) against an experimentally derived interval set (e.g., J2 fCLIP-seq peaks) by intersecting coordinates and quantifying overlap fractions.

## When NOT to use

- Input regions are already a merged/joined feature table rather than two independent BED interval sets needing coordinate intersection
- The goal is dsRNA region prediction from long-read RNA-seq data itself (use dsRID's extract_whole.py/model_predict.py pipeline instead of overlap validation)

## Inputs

- dsRID combined predicted dsRNA regions (BED format, data/dsRID_novel_dsRNA.bed from gxiaolab/dsRID repo)
- J2 fCLIP-seq HeLa dsRNA regions (Kim et al. 2018) in BED format

## Outputs

- j2_dsrid_overlap.tsv with per-region overlap coordinates and summary counts (7803 overlapping regions; 5036 EER-based; 2767 novel; 73.3% overlap fraction)

## How to apply

Obtain the combined dsRID predicted dsRNA regions from data/dsRID_novel_dsRNA.bed in the gxiaolab/dsRID GitHub repository, and the J2 fCLIP-seq HeLa dsRNA regions (Kim et al. 2018) as a BED-format genomic interval set. Load both interval sets in Python 3.x (bedtools/pybedtools-style intersect) and perform a genomic interval intersection to identify J2-captured dsRNA regions overlapping dsRID-predicted regions. Count the overlapping regions (expected 7803) and compute the overlap fraction relative to J2-captured dsRNAs (expected 73.3%). Stratify overlaps into EER-based overlaps (expected 5036) and novel dsRID predictions (expected 2767), then write per-region overlap coordinates and summary counts to j2_dsrid_overlap.tsv.

## Related tools

- **dsRID** (Source of the predicted dsRNA region BED file (data/dsRID_novel_dsRNA.bed) being validated against experimental J2 fCLIP-seq intervals) — https://github.com/gxiaolab/dsRID
- **Python** (Python 3.x runtime for loading BED interval sets and performing bedtools/pybedtools-style intersect to compute overlaps)

## Examples

```
import pybedtools; dsrid = pybedtools.BedTool('data/dsRID_novel_dsRNA.bed'); j2 = pybedtools.BedTool('j2_fclipseq_hela.bed'); ov = j2.intersect(dsrid, u=True); print(len(ov)/len(j2))
```

## Evaluation signals

- Computed number of overlapping regions equals 7803
- Overlap fraction of J2-captured dsRNAs equals 73.3%
- Stratified counts: 5036 EER-based overlaps and 2767 novel dsRID predictions among the 7803 overlapping regions
- j2_dsrid_overlap.tsv contains per-region overlap coordinates plus the summary counts
- Both input interval sets are parsed as valid BED-format genomic intervals before intersection

## Limitations

- Cell type differences exist between the HeLa J2 fCLIP-seq data (Kim et al. 2018) and the AD-brain-derived dsRID predictions, so the 73.3% overlap is a cross-cell-type validation rather than a matched comparison
- No version of dsRID or specific package versions are stated; required packages are only listed in requirements.txt

## Evidence

- [results] Reproduce the J2 fCLIP-seq overlap validation: [evidence span withheld — anchor status: not_found]
- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [readme] data availability: "dsRID predicted regions across all samples are available under data/dsRID_novel_dsRNA.bed"
- [results] stratification: [evidence span withheld — anchor status: not_found]
