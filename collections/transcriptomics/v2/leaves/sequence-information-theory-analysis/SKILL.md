---
name: sequence-information-theory-analysis
description: Use when reads in UMI libraries show mapping shifts and you need to test
  whether cluster size correlates with sequence complexity, e.g. computing Shannon
  Entropy and Mutual Information per UMI-read cluster in run_171108.
when_to_use_negative:
- Input data lack UMI information entirely — in the absence of UMI the mapping shift
  artefact is undetectable and cluster-size stratification cannot be built
- The goal is to conclusively attribute mapping shifts to PCR stutter — the article
  warns that shifts can occur with reads of normal or high entropy, so entropy metrics
  alone cannot establish mechanism
- The data set is very small — the article notes that possibly larger data sets are
  needed to observe meaningful trends in these metrics and in visible repeat content
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3307
tools:
- name: Perl
  role: Scripting language for grouping reads into UMI-read clusters and running statistical
    tests (e.g. ANOVA via Perl statistics modules) on entropy and mutual information
    values
  repo: https://github.com/ncgr/UMI-analysis
- name: C
  role: Compiled pipeline components (umi_counterizer, fastq_umi_clipper, umi_analyzer_uthash)
    that generate the umi_positions.txt mapping file used as input to the entropy
    analysis
  repo: https://github.com/ncgr/UMI-analysis
- name: STAR
  role: Aligner used to produce the sorted SAM file of read-to-genome mappings from
    which UMI-read cluster positions are derived
  repo: https://github.com/alexdobin/STAR
- name: uthash
  role: C preprocessor-macro hash used by umi_analyzer_uthash to track genomic positions
    associated with each UMI
  repo: http://troydhanson.github.io/uthash/userguide.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:7a24371f5a371bc89346686c92916d7ea5c8cb98da3e9cbab3b6991667f8e012
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - Perl
  - STAR
  - uthash
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequence-information-theory-analysis

## Summary

Computes Shannon Entropy over nucleotide sequences and Mutual Information between base positions of modal reads in UMI-read clusters, then tests whether these information-theory metrics vary systematically with UMI-read cluster size using one-way ANOVA, to characterize low-entropy/tandem-repeat tendencies of reads exhibiting the UMI mapping shift artefact.

## When to use

Use when reads in UMI libraries show mapping shifts and you need to test whether cluster size correlates with sequence complexity, e.g. computing Shannon Entropy and Mutual Information per UMI-read cluster in run_171108.

## When NOT to use

- Input data lack UMI information entirely — in the absence of UMI the mapping shift artefact is undetectable and cluster-size stratification cannot be built
- The goal is to conclusively attribute mapping shifts to PCR stutter — the article warns that shifts can occur with reads of normal or high entropy, so entropy metrics alone cannot establish mechanism
- The data set is very small — the article notes that possibly larger data sets are needed to observe meaningful trends in these metrics and in visible repeat content

## Inputs

- umi_positions.txt file mapping each UMI to its genomic positions (output of src/c/umi_analyzer_uthash)
- Associated read sequences for run_171108 (quality-filtered, UMI-clipped FASTQ and sorted SAM alignments from STAR with no introns specified)

## Outputs

- Per-cluster Shannon Entropy values over the modal read nucleotide sequence
- Per-cluster Mutual Information values between base positions of the modal read
- One-way ANOVA P values for cluster size as factor versus each information-theory metric (expected P < 10^-16)

## How to apply

Load the run_171108 umi_positions.txt mapping file (produced by umi_analyzer_uthash from umi_counterizer.pl output) together with the associated read sequences, and group reads into UMI-read clusters of increasing cluster size around a modal position. For each cluster, identify the modal read and compute its Shannon Entropy over the nucleotide sequence, and compute Mutual Information between base positions of the modal read as a complementary sequence-complexity metric. Then run a one-way ANOVA (in R or via Perl statistics modules) with cluster size as the factor variable and Shannon Entropy / Mutual Information as response variables. Accept the result when ANOVA P values fall below 1e-16 for both metrics and the trend directions match Figure 8: Shannon Entropy decreases and Mutual Information increases as cluster size increases. Interpret significant entropy decrease as consistent with PCR stutter in low-entropy or simple-tandem-repeat templates, while remembering the article's caution that mapping shifts also occur with reads of normal or high entropy.

## Related tools

- **Perl** (Scripting language for grouping reads into UMI-read clusters and running statistical tests (e.g. ANOVA via Perl statistics modules) on entropy and mutual information values) — https://github.com/ncgr/UMI-analysis
- **C** (Compiled pipeline components (umi_counterizer, fastq_umi_clipper, umi_analyzer_uthash) that generate the umi_positions.txt mapping file used as input to the entropy analysis) — https://github.com/ncgr/UMI-analysis
- **STAR** (Aligner used to produce the sorted SAM file of read-to-genome mappings from which UMI-read cluster positions are derived) — https://github.com/alexdobin/STAR
- **uthash** (C preprocessor-macro hash used by umi_analyzer_uthash to track genomic positions associated with each UMI) — http://troydhanson.github.io/uthash/userguide.html

## Examples

```
Rscript -e 'd<-read.table("entropy_by_cluster.tsv",header=TRUE); summary(aov(shannon_entropy~factor(cluster_size),data=d)); summary(aov(mutual_information~factor(cluster_size),data=d))'
```

## Evaluation signals

- One-way ANOVA with cluster size as factor yields P < 10^-16 for both Shannon Entropy and Mutual Information response variables
- Trend directions match Figure 8 for run_171108: Shannon Entropy decreases and Mutual Information increases as UMI-read cluster size increases
- Clusters are correctly constructed around a modal position with reads grouped by shared UMI and closely-spaced mapping locations
- Per-cluster entropy/MI values are computed on the modal read only, as specified by the workflow

## Limitations

- Mapping shifts occur even with reads of normal or high entropy, so any conclusion about the contribution of PCR stutter caused by simple tandem repeats must be tempered
- The lower-entropy tendency is pronounced only in some data sets (most pronounced in run_171108 and the La Manno data set) and defies neat generalization despite highly significant ANOVA P values
- Possibly larger data sets are needed to observe meaningful trends in these metrics and in visible repeat content
- Repeat-stabilized PCR stutter may contribute to but not fully explain the artefact; the alternative source (e.g. Illumina bridge amplification) was not ruled out experimentally

## Evidence

- [results] In the run_171108 data set, one-way ANOVA using cluster size as the factor variable found that the pronounced decrease in Shannon Entropy and increase in Mutual Information as cluster size increases are both highly significant (P < 10−16).: [evidence span withheld — anchor status: not_found]
- [intro] Sequences demonstrating the artefact tend to contain simple tandem repeats.: "sequences demonstrating the read mapping shift artefact tend to contain simple tandem repeats and that information theory metrics predictive of such repeats also predict the occurrence of the"
- [discussion] Caution on PCR stutter interpretation from discussion.: "any conclusion about the contribution of PCR stutter caused by simple tandem repeats must be tempered by the caution that the mapping shifts can occur with reads of normal or high entropy"
- [readme] README description of umi_positions.txt generation.: "Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt."
