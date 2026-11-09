---
name: shannon-entropy-computation
description: Use when UMI-read clusters (reads sharing a UMI, mapping within ≤3 bp
  around a modal position) must be characterized for sequence complexity, i.e. per-cluster
  Shannon Entropy and Mutual Information are needed to test whether low-complexity/tandem-repeat
  reads predict the mapping shift artefact.
when_to_use_negative:
- As a standalone diagnostic filter for the mapping shift artefact — the article cautions
  that mapping shifts can occur with reads of normal or high entropy, so low entropy
  alone does not predict the artefact
- On small data sets where the article notes meaningful trends in these metrics and
  visible repeat content may not be observable
- If the UMI-read clusters have not yet been formed (i.e. reads have not been grouped
  by shared UMI and closely spaced mapping positions)
tools:
- name: Perl
  role: Implements the UMI-analysis pipeline scripts that produce the UMI-position
    groupings over which entropy and mutual information are computed, and can host
    statistics modules for the one-way ANOVA
  repo: https://github.com/ncgr/UMI-analysis
- name: C
  role: C programs (e.g. umi_analyzer_uthash, fastq_umi_clipper) in the pipeline generate
    the per-UMI position file used as the starting point for the cluster-level entropy
    analysis
  repo: https://github.com/ncgr/UMI-analysis
- name: R
  role: Runs the one-way ANOVA with cluster size as the factor variable and Shannon
    Entropy / Mutual Information as response variables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:654e967d1ca0bec7a9b0eb56ba6d40f13ba8354853467a7564372e16165a26c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - Perl
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# shannon-entropy-computation

## Summary

Computes information-theory metrics — Shannon Entropy over the nucleotide sequence of the modal read and Mutual Information between base positions — for UMI-read clusters in the run_171108 single-cell RNA-Seq data, to quantify sequence complexity associated with the read mapping shift artefact attributed in part to PCR stutter in low-entropy, simple tandem repeat templates.

## When to use

Use when UMI-read clusters (reads sharing a UMI, mapping within ≤3 bp around a modal position) must be characterized for sequence complexity, i.e. per-cluster Shannon Entropy and Mutual Information are needed to test whether low-complexity/tandem-repeat reads predict the mapping shift artefact.

## When NOT to use

- As a standalone diagnostic filter for the mapping shift artefact — the article cautions that mapping shifts can occur with reads of normal or high entropy, so low entropy alone does not predict the artefact
- On small data sets where the article notes meaningful trends in these metrics and visible repeat content may not be observable
- If the UMI-read clusters have not yet been formed (i.e. reads have not been grouped by shared UMI and closely spaced mapping positions)

## Inputs

- umi_positions.txt file (each UMI and the genomic positions where it maps)
- read sequences associated with each UMI (from the sorted SAM/aligned reads)
- UMI-read clusters grouped by cluster size around a modal mapping position

## Outputs

- Per-cluster Shannon Entropy values over the modal read sequence
- Per-cluster Mutual Information values between base positions of the modal read
- One-way ANOVA results (P values) for Entropy and Mutual Information versus cluster size

## How to apply

Load the run_171108 umi_positions.txt mapping file and the associated read sequences, grouping reads into UMI-read clusters of increasing cluster size around a modal position. For each cluster, identify the modal read and compute its Shannon Entropy over the nucleotide sequence. Compute Mutual Information between base positions of the modal read as a complementary sequence-complexity metric. Run a one-way ANOVA (via R or Perl statistics modules) with cluster size as the factor and Shannon Entropy / Mutual Information as response variables. Accept the result when the ANOVA yields P < 10^-16 for both metrics and the trend directions (entropy decrease, mutual information increase) match the reported Figure 8 pattern for run_171108.

## Related tools

- **Perl** (Implements the UMI-analysis pipeline scripts that produce the UMI-position groupings over which entropy and mutual information are computed, and can host statistics modules for the one-way ANOVA) — https://github.com/ncgr/UMI-analysis
- **C** (C programs (e.g. umi_analyzer_uthash, fastq_umi_clipper) in the pipeline generate the per-UMI position file used as the starting point for the cluster-level entropy analysis) — https://github.com/ncgr/UMI-analysis
- **R** (Runs the one-way ANOVA with cluster size as the factor variable and Shannon Entropy / Mutual Information as response variables)

## Evaluation signals

- One-way ANOVA with cluster size as the factor yields P < 10^-16 for both Shannon Entropy and Mutual Information
- Shannon Entropy decreases as UMI-read cluster size increases
- Mutual Information increases as UMI-read cluster size increases
- Trend directions reproduce the reported Figure 8 pattern for run_171108
- Per-cluster values are computed on the modal read of clusters grouped around a modal mapping position, not on arbitrary read subsets

## Limitations

- The lower-entropy tendency is pronounced only in some data sets (e.g. run_171108, La Manno) and less pronounced in others, despite highly significant ANOVA P values — the data defy neat generalization
- Conclusions about PCR stutter caused by simple tandem repeats must be tempered because mapping shifts can occur with reads of normal or high entropy
- Larger data sets may be needed to observe meaningful trends in these metrics and in visible repeat content
- Repeat-stabilized PCR stutter may contribute to, but not fully explain, the artefact; the alternative source is unidentified

## Evidence

- [abstract] may be partially explained by PCR stutter, which is often seen in low-entropy templates and those containing simple tandem repeats.: "may be partially explained by PCR stutter, which is often seen in low-entropy templates and those containing simple tandem repeats."
- [intro] sequences demonstrating the read mapping shift artefact tend to contain simple tandem repeats and that information theory metrics predictive of such repeats: "sequences demonstrating the read mapping shift artefact tend to contain simple tandem repeats and that information theory metrics predictive of such repeats"
- [discussion] any conclusion about the contribution of PCR stutter caused by simple tandem repeats must be tempered by the caution that the mapping shifts can occur: "any conclusion about the contribution of PCR stutter caused by simple tandem repeats must be tempered by the caution that the mapping shifts can occur"
- [discussion] Possibly, larger data sets are needed to observe meaningful trends in these metrics and in visible repeat content: "Possibly, larger data sets are needed to observe meaningful trends in these metrics and in visible repeat content"
- [readme] Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt.: "Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt."
