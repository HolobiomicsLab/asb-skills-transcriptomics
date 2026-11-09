---
name: classifier-auroc-evaluation
description: Use when a sample's RNA-seq BAM must be classified into skipped vs composite
  intronic terminal exon candidates and classifier performance (AUROC, TPR, F1, MCC)
  must be quantified against held-out annotated regions.
when_to_use_negative:
- The input data are 3'-end sequencing, A-seq, 3P-seq, polyA-seq, or long-read Iso-seq
  benchmarks — these serve as ground truth for benchmarking, not as classifier-training
  RNA-seq input
- The goal is only APA quantification in annotated 3'UTRs (tools like DaPars, QAPA,
  or LABRAT are designed for that, not novel IPA identification)
- The candidates lack sufficient read support (fewer than 5 spliced reads for skipped
  or 10 span reads for composite terminal exons), so classification is underdetermined
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: Runs the read module that trains random-forest classifiers on RNA-seq alignment
    features and ensembles them over ten randomized subsamples
  repo: https://github.com/YY-TMU/InPACT
- name: featureCount
  role: Computes per-region RNA-seq alignment features (region length, expression,
    spliced/unspliced read counts) used as classifier input
- name: HISAT2
  role: Aligns HEK293 replicate RNA-seq reads to the GRCh38 genome to produce the
    BAM used for feature extraction
- name: SAMtools
  role: Filters out reads flagged as secondary alignment prior to feature computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:7f6073c801abf931906c5a456ff538969277b363de1ec7c0eef82227f192255c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - SAMtools
  - featureCount
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# classifier-auroc-evaluation

## Summary

Trains and evaluates separate random-forest ensembles for skipped and composite intronic terminal exon candidates in InPACT's read module, and validates performance by reproducing per-replicate AUROCs (e.g., HEK293 rep1 skipped 0.999 / composite 0.997) on held-out testing regions.

## When to use

Use when a sample's RNA-seq BAM must be classified into skipped vs composite intronic terminal exon candidates and classifier performance (AUROC, TPR, F1, MCC) must be quantified against held-out annotated regions.

## When NOT to use

- The input data are 3'-end sequencing, A-seq, 3P-seq, polyA-seq, or long-read Iso-seq benchmarks — these serve as ground truth for benchmarking, not as classifier-training RNA-seq input
- The goal is only APA quantification in annotated 3'UTRs (tools like DaPars, QAPA, or LABRAT are designed for that, not novel IPA identification)
- The candidates lack sufficient read support (fewer than 5 spliced reads for skipped or 10 span reads for composite terminal exons), so classification is underdetermined

## Inputs

- Sorted, coordinate-ordered BAM of RNA-seq alignments (secondary alignments removed with SAMtools)
- RefSeq GTF annotation for GRCh38 (hg38.109.20211119.ncbiRefSeq.gtf)
- Per-region RNA-seq alignment features computed with featureCount
- Class labels for terminal, internal, and background regions

## Outputs

- Trained random-forest ensembles for skipped and composite terminal exon candidates
- auroc_results.csv with per-replicate, per-class AUROC and other metrics (TPR, TNR, accuracy, precision, F1, MCC)
- predict.result.txt listing predicted terminal exons, IPA type (skipped/composite), gene, and PolyAsite

## How to apply

Define three training classes from the RefSeq annotation (GRCh38/hg38.ncbiRefSeq.gtf): terminal exons (annotated last exons with at least five splice-in reads), internal exons (annotated exons between first and last with at least five splice-in reads), and background regions (annotated last exons with fewer than five splice-in reads), then randomly split regions 80:20 into training and testing sets. Compute per-region RNA-seq alignment features with featureCount, including relative region length, normalized region expression, coefficients of variation, entropy efficiency, and spliced/unspliced read counts around the 5' and 3' ends. Train separate random-forest classifiers for skipped candidates (requiring >=5 uniquely mapped spliced reads with the 3' end in the candidate region) and composite candidates (requiring >=10 uniquely mapped unspliced reads crossing the closest upstream splice site), ensembling over ten randomized subsamples of the training set. Evaluate each ensemble on the 20% held-out testing set, computing true positive rate, true negative rate, accuracy, precision, F1 score, Matthew's correlation coefficient, and AUROC, and compare the per-replicate AUROCs (rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998) within a small tolerance.

## Related tools

- **InPACT** (Runs the read module that trains random-forest classifiers on RNA-seq alignment features and ensembles them over ten randomized subsamples) — https://github.com/YY-TMU/InPACT
- **featureCount** (Computes per-region RNA-seq alignment features (region length, expression, spliced/unspliced read counts) used as classifier input)
- **HISAT2** (Aligns HEK293 replicate RNA-seq reads to the GRCh38 genome to produce the BAM used for feature extraction)
- **SAMtools** (Filters out reads flagged as secondary alignment prior to feature computation)

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf --minimum_spliced_reads_for_skipped 5 --minimum_span_reads_for_composite 10 -P 5
```

## Evaluation signals

- auroc_results.csv must match expected values within a small tolerance: rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998
- Terminal exon training class regions must each have at least five splice-in reads; background regions fewer than five
- The 80:20 train/test split of regions must be reproduced so evaluation uses the 20% held-out testing set
- Each classifier ensemble must consist of ten randomized training-set subsamples, ensembled per the methods description
- Skipped candidates must retain only regions with >=5 uniquely mapped spliced reads with the 3' end in the candidate region; composite candidates >=10 unspliced reads crossing the closest upstream splice site

## Limitations

- AUROC expectations are replicate- and class-specific; deviations may arise from read-count stochasticity at the five/ten read thresholds
- The training classes depend on the RefSeq annotation quality and GRCh38 genome build; alternative annotations change class definitions
- Random 80:20 splits and randomized subsampling mean AUROC values are not deterministic and only match expected values within a small tolerance

## Evidence

- [methods] Training class definition and read thresholds: "annotated unique last exons that have at least five splice-in reads"
- [methods] Random-forest ensembling strategy: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [readme] Default read-count filters for candidate classes: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] Composite candidate read threshold: "Minimum number of reads that cross the boundary required to consider the potential composite terminal exon. [default=10]"
- [readme] CLI invocation for IPA site identification: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
