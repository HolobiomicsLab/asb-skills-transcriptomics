---
name: gene-body-coverage-bias-verification
description: Use when RNA-seq BAM files from multiple tissue groups (e.g. broiler
  liver, abdominal fat, breast muscle) must be checked for systematic gene-body coverage
  bias before variant or expression analyses.
when_to_use_negative:
- When the goal is overall alignment statistics rather than positional coverage —
  use STAR log metrics (unique mapping rates, 'too short' read percentages, chimeric
  reads) instead
- When input files are raw VCFs for allele-specific expression testing — that is the
  role of VADT, not gene-body coverage QC
- When reads have not yet been aligned; geneBody_coverage.py requires BAM input, not
  FASTQ (FASTQ-level QC is FastQC's role)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RSeQC 2.6.4
  role: Run geneBody_coverage.py on tissue-grouped BAM files to detect systematic
    gene-body coverage bias
- name: Picard 2.8.1
  role: Mark duplicate reads on the STAR-aligned BAMs before coverage assessment
- name: FastQC
  role: Screen all 68 FASTQ files for sequence quality prior to alignment and coverage
    checks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:e6516d5786f7b39f73f5c996347dbca262f6a1c91bb44e986a913fac8c86a7da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - Picard
  - Picard 2.8.1
  - RSeQC
  - RSeQC 2.6.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Gene body coverage bias verification

## Summary

Assess whether sequencing coverage is systematically biased along gene bodies (5'/3' skew) using RSeQC's geneBody_coverage.py, applied per tissue to deduplicated BAM files so that downstream ASE or differential analyses are not confounded by coverage artifacts.

## When to use

Use when RNA-seq BAM files from multiple tissue groups (e.g. broiler liver, abdominal fat, breast muscle) must be checked for systematic gene-body coverage bias before variant or expression analyses.

## When NOT to use

- When the goal is overall alignment statistics rather than positional coverage — use STAR log metrics (unique mapping rates, 'too short' read percentages, chimeric reads) instead
- When input files are raw VCFs for allele-specific expression testing — that is the role of VADT, not gene-body coverage QC
- When reads have not yet been aligned; geneBody_coverage.py requires BAM input, not FASTQ (FASTQ-level QC is FastQC's role)

## Inputs

- Deduplicated BAM files produced by STAR 2.5.2b two-pass alignment plus Picard 2.8.1 duplicate marking
- RSeQC-compatible BED file of gene models for Gallus_gallus_5.0 (Ensembl release 86 GTF-derived annotation)
- Per-sample tissue group labels (breast muscle n=23, abdominal fat n=22, liver n=23)
- FASTQ files for upstream FastQC quality screening

## Outputs

- Per-sample geneBody_coverage.py output plots (PDF/PNG) of normalized coverage along gene bodies
- Per-sample gene body coverage text tables usable for cross-tissue curve comparison
- A pass/fail judgment on systematic tissue-specific gene-coverage bias to document before ASE analysis

## How to apply

After two-pass STAR 2.5.2b alignment to the Gallus_gallus_5.0 build and duplicate marking with Picard 2.8.1, run RSeQC 2.6.4's geneBody_coverage.py on the resulting BAM files, grouping samples by tissue (breast muscle n=23, abdominal fat n=22, liver n=23) and supplying an RSeQC-compatible BED annotation of the genome. Compare the normalized per-sample gene-body coverage curves across tissues: overlapping profiles with no tissue-specific 5' or 3' skew indicate no systematic coverage bias. This read-level QC should be paired with upstream FastQC screening of the 68 fastq files (all samples must pass) and interpreted alongside STAR alignment logs, since muscle samples aligned poorly (avg. 15.39% 'too short' reads) and such mapping differences can masquerade as or confound coverage artifacts.

## Related tools

- **RSeQC 2.6.4** (Run geneBody_coverage.py on tissue-grouped BAM files to detect systematic gene-body coverage bias)
- **Picard 2.8.1** (Mark duplicate reads on the STAR-aligned BAMs before coverage assessment)
- **FastQC** (Screen all 68 FASTQ files for sequence quality prior to alignment and coverage checks)

## Examples

```
python geneBody_coverage.py -i muscle1.bam,muscle2.bam,fat1.bam,fat2.bam,liver1.bam,liver2.bam -r Gallus_gallus_5.0.bed -o geneBody_coverage_tissue
```

## Evaluation signals

- Normalized gene-body coverage curves for liver, abdominal fat, and breast muscle samples overlap with no tissue-specific 5' or 3' decay pattern
- All 68 FASTQ samples pass the FastQC quality control step before alignment
- geneBody_coverage.py is run on Picard-deduplicated BAMs, not raw STAR output
- Coverage assessment is interpreted jointly with STAR log metrics: breast muscle avg. 15.39% 'too short' reads vs 4.50% (fat) and 5.33% (liver) flags mapping, not coverage bias, as the dominant QC issue
- Report documents that no systematic gene-coverage bias by tissue was found before proceeding to GATK variant calling and VADT ASE analysis

## Limitations

- Breast muscle samples aligned substantially worse to Gallus_gallus_5.0 (avg. 15.39% 'too short' reads, ANOVA p ≈ 9.76e-24), which the authors attributed to genome build/annotation limitations for muscle-related genes; poor mapping can complicate interpretation of coverage metrics
- geneBody_coverage.py results depend on the annotation supplied in the BED file (Ensembl release 86 GTF vs Gallus_gallus_6.0 GTF choices affect gene definitions)
- Coverage bias assessment does not resolve reference-allele bias in ASE calling, which required genome masking with Bedtools and re-alignment

## Evidence

- [methods] geneBody_coverage.py: [evidence span withheld — anchor status: not_found]
- [methods] FastQC screening: "All fastq files were reviewed for sequence quality using FastQC24 and all samples passed this quality control step"
- [discussion] Muscle mapping issues: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
- [methods] Picard deduplication: "duplicate reads were marked using Picard (version 2.8.1)35"
