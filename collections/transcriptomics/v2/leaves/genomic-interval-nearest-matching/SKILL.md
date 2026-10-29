---
name: genomic-interval-nearest-matching
description: Use when predicted genomic intervals (e.g. IPA sites from InPACT, APAIQ,
  IPAFinder) must be validated against reference polyA site sets (A-seq, 3P-seq, polyA-seq,
  Iso-seq) by nearest-feature distance rather than exact overlap.
when_to_use_negative:
- When the goal is to quantify IPA usage (PAU) rather than validate site coordinates
  — use Salmon transcript quantification (InPACT_quantify) instead
- When ground-truth intervals are unavailable or 3'-end/Iso-seq data are absent —
  no reference set exists to match against
- When overlap-based logic is required (e.g. annotating reads to intervals) — use
  BEDTools intersect rather than closest, which returns the nearest neighbor regardless
  of overlap
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: Produces the predicted IPA site intervals (skipped and composite terminal
    exons) that are matched against ground truth
  repo: https://github.com/YY-TMU/InPACT
- name: BEDTools closest
  role: Nearest-neighbor matching of predicted sites to ground-truth polyA sites with
    reported distances
- name: liftOver
  role: Converts GEO ground-truth polyA coordinates to the GRCh38 assembly before
    matching
- name: IPAFinder
  role: Competing IPA-site predictor whose outputs are matched to the same ground
    truth for benchmarking
- name: APAIQ
  role: Competing IPA-site predictor whose outputs are matched to the same ground
    truth for benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:e213658e1262aa6eac4384c675fb252542a5ad2866a49e321c240fda079c7936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APAIQ
  - BEDTools closest
  - IPAFinder
  - InPACT
  - Polyester
  - liftOver
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# genomic-interval-nearest-matching

## Summary

Matches predicted genomic features (here, RNA-seq-derived IPA/polyA sites) to ground-truth intervals by nearest-neighbor distance using BEDTools closest, converting coordinate systems with liftOver and applying a distance window to call true positives for tool benchmarking.

## When to use

Use when predicted genomic intervals (e.g. IPA sites from InPACT, APAIQ, IPAFinder) must be validated against reference polyA site sets (A-seq, 3P-seq, polyA-seq, Iso-seq) by nearest-feature distance rather than exact overlap.

## When NOT to use

- When the goal is to quantify IPA usage (PAU) rather than validate site coordinates — use Salmon transcript quantification (InPACT_quantify) instead
- When ground-truth intervals are unavailable or 3'-end/Iso-seq data are absent — no reference set exists to match against
- When overlap-based logic is required (e.g. annotating reads to intervals) — use BEDTools intersect rather than closest, which returns the nearest neighbor regardless of overlap

## Inputs

- Predicted IPA/polyA site coordinates from InPACT (predict.result.txt or SAF: gene, chromosome, start, end, strand)
- Predicted IPA sites from APAIQ and IPAFinder
- Ground-truth polyA site coordinates from A-seq, 3P-seq, polyA-seq (GEO accessions, e.g. GSE167486)
- PolyA sites extracted from Iso-seq isoform annotations
- GRCh38/hg38 chain file and reference GTF (hg38.109.20211119.ncbiRefSeq.gtf) for liftOver

## Outputs

- Nearest-match table (prediction, closest ground-truth polyA site, distance in nt) from BEDTools closest
- True-positive/precision/sensitivity counts per tool using the 50 nt window
- Iso-seq validation rates per tool
- Cumulative distribution plots of distance-to-ground-truth error per tool

## How to apply

Download ground-truth polyA site coordinates (A-seq, 3P-seq, polyA-seq from GEO; Iso-seq isoform annotations) and liftOver them to GRCh38 so all intervals share one assembly. Export each tool's predicted sites as BED/SAF-style intervals (gene, chromosome, start, end, strand — the InPACT SAF format spans 20 nt upstream to the potential polyA site). Run `bedtools closest -a predictions.bed -b ground_truth.bed -d` to attach the nearest reference site and its distance to every prediction, and classify predictions within 50 nt of a predefined polyA site as true positives. From the matched pairs compute Sensitivity = TP/Predefined and Precision = TP/(TP+FP); for Iso-seq, compare InPACT and IPAFinder predictions to polyA sites extracted directly from the Iso-seq isoform annotation to obtain Iso-seq validation rates. Summarize true-positive percentages per tool and plot cumulative distributions of distance/error to judge which method identifies IPA sites most precisely.

## Related tools

- **InPACT** (Produces the predicted IPA site intervals (skipped and composite terminal exons) that are matched against ground truth) — https://github.com/YY-TMU/InPACT
- **BEDTools closest** (Nearest-neighbor matching of predicted sites to ground-truth polyA sites with reported distances)
- **liftOver** (Converts GEO ground-truth polyA coordinates to the GRCh38 assembly before matching)
- **IPAFinder** (Competing IPA-site predictor whose outputs are matched to the same ground truth for benchmarking)
- **APAIQ** (Competing IPA-site predictor whose outputs are matched to the same ground truth for benchmarking)

## Examples

```
bedtools closest -a InPACT_predictions.bed -b polyA_groundtruth_GRCh38.bed -d > matched_sites.tsv
```

## Evaluation signals

- Benchmark ordering reproduced: >60% of InPACT-identified IPA sites within 50 nt of ground-truth polyA sites versus ~30% for APAIQ and ~20% for IPAFinder
- Sensitivity = TP/Predefined and Precision = TP/(TP+FP) computed only after confirming all intervals share the same assembly post-liftOver
- Iso-seq validation rates obtained for InPACT and IPAFinder on GSE167486 small airway epithelial cell data using the same BEDTools closest procedure
- Every prediction row in the match table carries a finite distance value (closest hit found), and strand/chromosome fields agree between prediction and reference

## Limitations

- The 50 nt true-positive window is a chosen benchmarking convention; results are sensitive to this threshold
- 3'-end sequencing ground truth is scarce ('they have not been widely utilized and the available data remain relatively scarce'), limiting validation sets
- No version numbers reported for InPACT or the compared tools, and no parameters/thresholds reported for the simulations, hindering exact reproduction
- Ground truth from GEO must be converted between assemblies with liftOver, introducing potential coordinate-conversion artifacts

## Evidence

- [discussion] InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq: "InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq"
- [methods] multiple datasets of cell lines and tissue samples were utilized as the ground truth for comparison: "multiple datasets of cell lines and tissue samples were utilized as the ground truth for comparison"
- [methods] evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0: "evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0"
- [readme] Regions from 20nt upstream to potential polyA sites, with columns for gene, chromosome, start, end, and strand.: "Regions from 20nt upstream to potential polyA sites, with columns for gene, chromosome, start, end, and strand."
- [intro] they have not been widely utilized and the available data remain relatively scarce, impeding further investigation: "they have not been widely utilized and the available data remain relatively scarce, impeding further investigation"
