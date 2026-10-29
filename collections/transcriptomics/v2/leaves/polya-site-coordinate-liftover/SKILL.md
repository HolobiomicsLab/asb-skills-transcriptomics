---
name: polya-site-coordinate-liftover
description: Use when benchmarking IPA site callers against GEO-deposited 3'-end sequencing
  ground truth whose polyA site coordinates were produced on an older assembly and
  must be converted to GRCh38 before interval matching.
when_to_use_negative:
- Ground-truth polyA site coordinates are already in GRCh38 coordinates — liftOver
  is unnecessary and could shift valid sites
- The comparison target is Iso-seq polyA sites, which are extracted directly from
  the isoform annotation rather than lifted
- Benchmarking quantification accuracy (e.g., |ΔPAU| on Polyester-simulated data)
  rather than site-coordinate accuracy
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: liftOver
  role: Converts GEO-downloaded ground-truth polyA site coordinates from the original
    assembly to GRCh38 before matching predictions
- name: BEDTools closest
  role: Matches lifted (or Iso-seq-derived) ground-truth polyA sites to predicted
    IPA sites, classifying hits within 50 nt as true positives
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:89c821643262a5ed1e0a9cc7429f9bd16a6f7089f4204091c8c87729a91c299a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APAIQ
  - BEDTools closest
  - IPAFinder
  - InPACT
  - Polyester
  - liftOver
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# polya-site-coordinate-liftover

## Summary

Lifts polyadenylation site coordinates from ground-truth resources (A-seq, 3P-seq, polyA-seq) to the GRCh38 assembly with liftOver so that IPA tool predictions can be benchmarked on a consistent reference genome.

## When to use

Use when benchmarking IPA site callers against GEO-deposited 3'-end sequencing ground truth whose polyA site coordinates were produced on an older assembly and must be converted to GRCh38 before interval matching.

## When NOT to use

- Ground-truth polyA site coordinates are already in GRCh38 coordinates — liftOver is unnecessary and could shift valid sites
- The comparison target is Iso-seq polyA sites, which are extracted directly from the isoform annotation rather than lifted
- Benchmarking quantification accuracy (e.g., |ΔPAU| on Polyester-simulated data) rather than site-coordinate accuracy

## Inputs

- GEO-deposited ground-truth polyA site coordinate files (A-seq, 3P-seq, polyA-seq) in BED format on the source assembly
- liftOver chain file (e.g., source assembly to hg38/GRCh38)
- Tool-predicted IPA site BED files (InPACT, APAIQ, IPAFinder) on GRCh38
- Iso-seq isoform annotation GTF (for extracting validated polyA sites without liftover)

## Outputs

- GRCh38-coordinate BED file of lifted ground-truth polyA sites plus an unmapped-intervals file
- True-positive sets from BEDTools closest matches within 50 nt
- Sensitivity = TP/Predefined and Precision = TP/(TP+FP) per tool
- Iso-seq validation rates per tool

## How to apply

Download the ground-truth polyA site coordinates (A-seq, 3P-seq, polyA-seq) from GEO and convert them to GRCh38 using liftOver with the appropriate assembly-to-assembly chain file, discarding unmapped intervals. Keep the converted coordinates in BED format so they can be matched to tool predictions (InPACT, APAIQ, IPAFinder) with BEDTools closest. Classify a predicted site as a true positive when it falls within 50 nt of a lifted, predefined polyA site, then compute Sensitivity = TP/Predefined and Precision = TP/(TP+FP). For Iso-seq validation, extract polyA sites directly from the Iso-seq isoform annotation rather than lifting, and again use BEDTools closest to obtain validation rates. Rationale: all prediction tools were run on GRCh38-matched RNA-seq datasets (HEK293, MAQC UHR/brain, small airway epithelial cells), so ground truth must share that coordinate system for the 50-nt tolerance comparison to be valid.

## Related tools

- **liftOver** (Converts GEO-downloaded ground-truth polyA site coordinates from the original assembly to GRCh38 before matching predictions)
- **BEDTools closest** (Matches lifted (or Iso-seq-derived) ground-truth polyA sites to predicted IPA sites, classifying hits within 50 nt as true positives)

## Examples

```
liftOver polyA_sites_hg19.bed hg19ToHg38.over.chain.gz polyA_sites_hg38.bed unmapped.bed; bedtools closest -a predictions_hg38.bed -b polyA_sites_hg38.bed -d
```

## Evaluation signals

- Over 60% of InPACT-identified IPA sites should fall within 50 nt of lifted ground-truth polyA sites, versus ~30% for APAIQ and ~20% for IPAFinder
- Sensitivity and Precision values are reproducible as Sensitivity = TP/Predefined and Precision = TP/(TP+FP)
- Lifted site counts plus unmapped-interval counts equal the original ground-truth site counts per dataset
- All matched comparisons use GRCh38 coordinates for both predictions and lifted ground truth (consistent genome build)

## Limitations

- Sites that fail liftover (unmapped intervals) are lost from the ground-truth set, shrinking the Sensitivity denominator
- No version number given for liftOver, BEDTools, or any compared tool in the source workflow
- Accuracy at a 50-nt tolerance is coarse; finer positional precision differences between tools may be masked

## Evidence

- [methods] Ground truth liftover to GRCh38: [evidence span withheld — anchor status: not_found]
- [methods] 50-nt matching and metrics: [evidence span withheld — anchor status: not_found]
- [methods] Iso-seq extraction without liftover: [evidence span withheld — anchor status: not_found]
- [readme] RefSeq annotation on GRCh38/hg38: "hg38.109.20211119.ncbiRefSeq.gtf.gz"
