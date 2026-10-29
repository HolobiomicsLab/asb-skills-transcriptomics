---
name: published-statistics-reproduction
description: Use when a paper reports a checkable aggregate number (overlap fraction,
  region counts) and the underlying interval sets are publicly deposited, e.g. dsRID_novel_dsRNA.bed
  in a GitHub repo.
when_to_use_negative:
- You want to train or run dsRID itself on PacBio long-read RNA-seq BAM data (dsRID.sh
  / extract_whole.py / model_predict.py) rather than recompute a published overlap
  statistic from its deposited predictions.
- The comparison sets are not BED-format genomic intervals (e.g. expression matrices,
  feature tables), so no interval intersection can produce the reported statistic.
- No published numeric target or deposited interval file exists to validate against
  — the reproduction invariant would be unverifiable.
edam_operation: http://edamontology.org/operation_3234
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dsRID
  role: Source of the predicted dsRNA interval set (data/dsRID_novel_dsRNA.bed) against
    which the external J2 fCLIP-seq regions are intersected
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime for loading both BED interval sets and performing the pybedtools-style
    genomic interval intersection and count stratification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:2e103ce9ffc19f5bd47c70dd498bad36cb37d1fe06ade0a07b876b6752893aae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_3234
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# published-statistics-reproduction

## Summary

Reproduce a published summary statistic by independently recomputing it from the paper's deposited data files — here, re-deriving the 73.3% J2 fCLIP-seq vs dsRID dsRNA overlap (Fig. 3F) by BED interval intersection.

## When to use

Use when a paper reports a checkable aggregate number (overlap fraction, region counts) and the underlying interval sets are publicly deposited, e.g. dsRID_novel_dsRNA.bed in a GitHub repo.

## When NOT to use

- You want to train or run dsRID itself on PacBio long-read RNA-seq BAM data (dsRID.sh / extract_whole.py / model_predict.py) rather than recompute a published overlap statistic from its deposited predictions.
- The comparison sets are not BED-format genomic intervals (e.g. expression matrices, feature tables), so no interval intersection can produce the reported statistic.
- No published numeric target or deposited interval file exists to validate against — the reproduction invariant would be unverifiable.

## Inputs

- data/dsRID_novel_dsRNA.bed — dsRID-predicted dsRNA regions across all samples, from the gxiaolab/dsRID GitHub repository
- BED-format genomic interval set of J2 fCLIP-seq-captured dsRNA regions in HeLa cells (Kim et al. 2018)

## Outputs

- j2_dsrid_overlap.tsv — per-region overlap coordinates plus summary counts: 7803 overlapping regions; 5036 EER-based; 2767 novel; 73.3% overlap fraction

## How to apply

Clone the source repository (git clone https://github.com/gxiaolab/dsRID.git) and load the deposited interval file data/dsRID_novel_dsRNA.bed together with the external comparison set (J2 fCLIP-seq HeLa dsRNA regions from Kim et al. 2018, BED format). In Python 3.x, perform a genomic interval intersection (bedtools/pybedtools-style intersect) to find J2-captured dsRNAs overlapping dsRID predictions, then count unique overlapping regions and divide by the total J2-captured regions to get the overlap fraction. Stratify overlapping regions by prediction source into EER-based overlaps versus novel long dsRNA predictions. Validate by comparing the recomputed values against the published targets: 73.3% overlap fraction, 7803 overlapping regions, 5036 EER-based, and 2767 novel, as shown in Fig. 3F; material mismatches signal a coordinate-system, filtering, or file-version discrepancy rather than a failure of the published method.

## Related tools

- **dsRID** (Source of the predicted dsRNA interval set (data/dsRID_novel_dsRNA.bed) against which the external J2 fCLIP-seq regions are intersected) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime for loading both BED interval sets and performing the pybedtools-style genomic interval intersection and count stratification)

## Examples

```
import pybedtools; ov = pybedtools.BedTool('j2_hela_dsRNA.bed').intersect('data/dsRID_novel_dsRNA.bed'); print(len(ov))  # expect 7803; fraction = len(ov)/n_j2 == 0.733
```

## Evaluation signals

- Computed overlap fraction equals the published 73.3% of J2 fCLIP-seq-captured dsRNAs overlapping dsRID predictions.
- Number of unique overlapping regions equals 7803, matching Fig. 3F.
- Stratification of the 7803 regions yields 5036 EER-based overlaps and 2767 novel long dsRNA predictions (5036 + 2767 = 7803).
- j2_dsrid_overlap.tsv contains per-region overlap coordinates in valid BED-style genomic interval format for every counted region.
- Both input interval sets parse without errors and use a consistent genome build, so intersect returns a nonzero but well-defined result set.

## Limitations

- Cell type differences exist between the HeLa J2 fCLIP-seq data (Kim et al. 2018) and the dsRID predictions derived from Alzheimer's disease brain PacBio long-read RNA-seq, so the overlap is cross-dataset and not a like-for-like tissue comparison.
- The dsRID repository README lists requirements.txt packages but no specific software or package versions, so exact reproducibility of the deposited prediction file cannot be pinned to versions.
- dsRID predictions are model outputs from a random forest trained on PacBio AD brain data; differences in genome build or BED interval conventions will change the intersect counts.

## Evidence

- [results] 73.3% overlap fraction, 7803 overlapping regions: [evidence span withheld — anchor status: not_found]
- [readme] Deposited interval input file: "dsRID predicted regions across all samples are available under data/dsRID_novel_dsRNA.bed."
- [methods] Repository acquisition step: "git clone https://github.com/gxiaolab/dsRID.git"
- [abstract] Nature of the predicted regions being intersected: "we report dsRID (double-stranded RNA identifier), a machine-learning-based method to predict dsRNA regions in silico, leveraging the power of long-read RNA-sequencing"
