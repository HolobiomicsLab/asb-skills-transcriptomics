---
name: long-read-isoseq-validation-analysis
description: Use when benchmarking computationally predicted IPA/polyA sites from
  conventional RNA-seq (e.g., InPACT, IPAFinder) against long-read Iso-seq isoform
  annotations to obtain validation rates for a dataset such as GSE167486 small airway
  epithelial cells.
when_to_use_negative:
- No long-read Iso-seq data or isoform annotation is available for the same sample/dataset
  — use 3'-end sequencing ground truth (3P-seq, A-seq, polyA-seq) instead
- Goal is quantifying IPA usage (PAU) rather than validating site identification —
  use Salmon-based quantification and |ΔPAU| error metrics instead
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: InPACT
  role: Source of the predicted IPA sites being validated against Iso-seq-derived
    polyA sites
  repo: https://github.com/YY-TMU/InPACT
- name: IPAFinder
  role: Comparator tool whose predictions are validated against the same Iso-seq ground
    truth
- name: BEDTools closest
  role: Matches predicted IPA sites to the nearest Iso-seq-derived polyA site to compute
    validation rates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:29b7a94ddf4cbcdb13c27247434e2a4ab816b426be41a6734a676b73e053eb47
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
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# long-read-isoseq-validation-analysis

## Summary

Validates RNA-seq-derived IPA site predictions against long-read Iso-seq isoform annotations by matching predicted polyA sites to Iso-seq-extracted polyA sites and computing Iso-seq validation rates, as done in the InPACT benchmark.

## When to use

Use when benchmarking computationally predicted IPA/polyA sites from conventional RNA-seq (e.g., InPACT, IPAFinder) against long-read Iso-seq isoform annotations to obtain validation rates for a dataset such as GSE167486 small airway epithelial cells.

## When NOT to use

- No long-read Iso-seq data or isoform annotation is available for the same sample/dataset — use 3'-end sequencing ground truth (3P-seq, A-seq, polyA-seq) instead
- Goal is quantifying IPA usage (PAU) rather than validating site identification — use Salmon-based quantification and |ΔPAU| error metrics instead

## Inputs

- InPACT IPA site prediction table (terminal exon, IPA type, gene, coordinates, PolyAsite) exported as BED
- Competing tool predictions (e.g., IPAFinder, APAIQ) in BED format
- Iso-seq isoform annotation (GTF/BED) with polyA sites for matched samples, e.g., GSE167486 small airway epithelial cells
- Coordinate-sorted BAM alignments (InPACT input) on GRCh38

## Outputs

- BEDTools closest matched-pair table (predicted site, nearest Iso-seq polyA site, distance)
- Iso-seq validation rate per tool (fraction of predicted IPA sites within 50 nt of an Iso-seq polyA site)
- Cross-tool comparison of validation rates

## How to apply

Extract polyA sites directly from the Iso-seq isoform annotation of the matched dataset (e.g., the GSE167486 small airway epithelial cell dataset used in the InPACT paper). Convert both the tool-predicted IPA sites (e.g., from InPACT or IPAFinder) and the Iso-seq-derived polyA sites into BED format on the same assembly (GRCh38). Use BEDTools closest to match each predicted site to its nearest Iso-seq ground-truth polyA site, applying the same 50 nt distance criterion used for 3'-end sequencing benchmarks to classify predictions as supported. Report Iso-seq validation rates per tool (fraction of predicted sites within 50 nt of an Iso-seq polyA site) and compare tools side by side; a tool is judged superior when its validation rate exceeds the comparators.

## Related tools

- **InPACT** (Source of the predicted IPA sites being validated against Iso-seq-derived polyA sites) — https://github.com/YY-TMU/InPACT
- **IPAFinder** (Comparator tool whose predictions are validated against the same Iso-seq ground truth)
- **BEDTools closest** (Matches predicted IPA sites to the nearest Iso-seq-derived polyA site to compute validation rates)

## Examples

```
bedtools closest -a InPACT_predictions.bed -b isoseq_polyA_sites.bed -d
```

## Evaluation signals

- Every predicted IPA site has a nearest-Iso-seq-site match with a reported distance from BEDTools closest
- Validation rate is computed as the fraction of predictions within 50 nt of an Iso-seq polyA site, consistent with the 3'-end benchmark criterion
- Predictions and Iso-seq sites are lifted to the same genome assembly (GRCh38) before matching
- InPACT's Iso-seq validation rate should exceed those of IPAFinder and APAIQ on the matched dataset, per the published benchmark

## Limitations

- Iso-seq data are scarce and dataset-specific; validation requires Iso-seq isoform annotation for the same sample/cell type as the RNA-seq predictions
- The 50 nt distance window is a benchmark convention; sites farther than 50 nt from Iso-seq polyA sites are counted as unsupported even if biologically real
- The article does not report software versions or parameters for the compared tools (InPACT, IPAFinder, APAIQ) in the benchmark

## Evidence

- [discussion] InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq: "InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq"
- [methods] Ground truth datasets: "multiple datasets of cell lines and tissue samples were utilized as the ground truth for comparison"
- [readme] InPACT IPA site identification command: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
