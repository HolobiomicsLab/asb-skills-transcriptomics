---
name: random-forest-ensemble-training
description: Use when classifying candidate intronic terminal exons (or similar genomic
  regions) from conventional RNA-seq alignments, needing a supervised model with region-level
  features and held-out AUROC-style validation.
when_to_use_negative:
- The goal is 3'-UTR APA quantification rather than novel intronic polyadenylation
  identification — tools like DaPars or QAPA are the appropriate scope
- Only 3'-end sequencing or long-read data are available and no conventional RNA-seq
  alignment BAM exists, since the read module depends on standard RNA-seq alignment
  features
- The task requires quantifying already-annotated transcript abundance only, which
  is handled downstream by Salmon/gffread rather than the classifier
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: InPACT
  role: 'Implements the read module: constructs putative terminal exons from the BAM
    and trains/ensembles random-forest classifiers on RNA-seq alignment features'
  repo: https://github.com/YY-TMU/InPACT
- name: featureCount
  role: Computes per-region RNA-seq alignment features used as classifier input
- name: SAMtools
  role: Filters reads flagged as secondary alignment before feature computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:364703078eac2ebdf24ea0f3324f99b4fe55bceba86bbcd4038e1af5ddf216e5
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# random-forest-ensemble-training

## Summary

Train ensembled random-forest classifiers on RNA-seq alignment features to distinguish true intronic terminal exons from background, as implemented in InPACT's read module for skipped and composite IPA events.

## When to use

Use when classifying candidate intronic terminal exons (or similar genomic regions) from conventional RNA-seq alignments, needing a supervised model with region-level features and held-out AUROC-style validation.

## When NOT to use

- The goal is 3'-UTR APA quantification rather than novel intronic polyadenylation identification — tools like DaPars or QAPA are the appropriate scope
- Only 3'-end sequencing or long-read data are available and no conventional RNA-seq alignment BAM exists, since the read module depends on standard RNA-seq alignment features
- The task requires quantifying already-annotated transcript abundance only, which is handled downstream by Salmon/gffread rather than the classifier

## Inputs

- Coordinate-sorted BAM file of RNA-seq alignments filtered for secondary alignments with SAMtools
- RefSeq GTF annotation (e.g., hg38.109.20211119.ncbiRefSeq.gtf from UCSC goldenPath)
- Per-region alignment feature table from featureCount (relative length, normalized expression, coefficients of variation, entropy efficiency, spliced/unspliced counts at 5' and 3' ends)

## Outputs

- Trained random-forest ensembles for skipped and composite terminal exon classes (each over ten randomized subsamples)
- Per-replicate, per-class held-out metrics: AUROC, accuracy, precision, F1, MCC, TPR, TNR (e.g., auroc_results.csv)
- Terminal exon predictions (predict.result.txt) labeling candidate regions as skipped or composite

## How to apply

Define three training classes from the RefSeq (GRCh38) annotation: terminal exons (annotated last exons with at least five splice-in reads), internal exons (exons between first and last with at least five splice-in reads), and background regions (annotated last exons with fewer than five splice-in reads), then split regions 80:20 into training and testing sets. Compute per-region RNA-seq alignment features with featureCount, including relative region length, normalized region expression, coefficients of variation, entropy efficiency, and spliced/unspliced read counts around the 5' and 3' ends. Train separate random-forest ensembles for putative skipped terminal exons (candidates requiring >=5 uniquely mapped spliced reads with the 3' end in the candidate region) and composite terminal exons (candidates requiring >=10 uniquely mapped unspliced reads crossing the closest upstream splice site), each ensembled over ten randomized subsamples of the training set. Evaluate each ensemble on the 20% held-out testing set, computing true positive rate, true negative rate, accuracy, precision, F1 score, Matthew's correlation coefficient, and AUROC.

## Related tools

- **InPACT** (Implements the read module: constructs putative terminal exons from the BAM and trains/ensembles random-forest classifiers on RNA-seq alignment features) — https://github.com/YY-TMU/InPACT
- **featureCount** (Computes per-region RNA-seq alignment features used as classifier input)
- **SAMtools** (Filters reads flagged as secondary alignment before feature computation)

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf --minimum_spliced_reads_for_skipped 5 --minimum_span_reads_for_composite 10 -P 5
```

## Evaluation signals

- Held-out (20% test set) AUROC per replicate and class must match expected values within a small tolerance: rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998
- Reported metrics (TPR, TNR, accuracy, precision, F1, MCC) should all be high and consistent across the ten randomized subsample ensembles
- Classifier output must distinguish skipped vs composite labels: skipped candidates need >=5 uniquely mapped spliced reads with the 3' end in the region; composite candidates need >=10 uniquely mapped unspliced reads crossing the closest upstream splice site
- Training class counts should reflect the three definitions (terminal/internal exons with >=5 splice-in reads; background last exons with <5 splice-in reads) on an 80:20 split

## Limitations

- The read module depends on adequate sequencing depth; coverage-dependent signal quality is a known concern for coverage-based IPA approaches such as IPAFinder
- The reference workflow is tied to human GRCh38/hg38 with RefSeq annotation and precomputed candidate IPA site SAF; other genomes require regenerating the sequence-module site annotation
- No version number is given for InPACT or its compared tools, so exact reproducibility of classifier behavior across versions is not guaranteed

## Evidence

- [methods] InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers.: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [readme] Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] Minimum number of reads that cross the boundary required to consider the potential composite terminal exon. [default=10]: "Minimum number of reads that cross the boundary required to consider the potential composite terminal exon. [default=10]"
- [readme] InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
