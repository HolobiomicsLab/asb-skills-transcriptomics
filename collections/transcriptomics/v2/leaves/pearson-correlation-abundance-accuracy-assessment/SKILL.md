---
name: pearson-correlation-abundance-accuracy-assessment
description: Use when simulated RNA-seq data carry known theoretical circRNA isoform
  abundances and you must score how faithfully a quantifier (CircAST FPKM/TPM) recovers
  per-locus relative isoform fractions across conditions, or benchmark it against
  junction-read counting.
when_to_use_negative:
- The dataset is real RNA-seq without simulated ground truth - no theoretical isoform
  abundances exist to correlate against, so the Pearson-based accuracy check cannot
  be performed.
- The circRNA locus has a single isoform - relative isoform fractions are trivially
  1.0 and Pearson correlation across conditions is meaningless.
- The target loci are intronic or intergenic circRNAs, which CircAST's exon-annotated
  pipeline may miss, invalidating the comparison to theoretical values.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: CircAST
  role: Assembles circular isoforms via splice graphs and outputs per-transcript junction
    read counts, assigned read counts, FPKM, and TPM that feed the Pearson correlation
    assessment
  repo: https://github.com/xiaofengsong/CircAST
- name: Tophat2
  role: Aligns simulated paired-end reads per condition to produce accepted_hits.bam,
    the alignment input CircAST consumes
- name: Samtools
  role: Name-sorts accepted_hits.bam ('samtools sort -n') and converts it to accepted_hits.sorted.sam
    via 'samtools view'
- name: CIRCexplorer2
  role: Generates the back-splice junction list (circ_out/denovo/annotated_circ.txt)
    per condition in CircAST input format
- name: CIRI2
  role: Generates the back-splice junction list (result.txt) per condition in CircAST
    input format
- name: UROBORUS
  role: Generates the back-splice junction list (circRNA_list.txt) per condition in
    CircAST input format
- name: scipy
  role: scipy.stats.pearsonr computes the Pearson correlations between theoretical
    and estimated relative isoform fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:89d8ca7b42119a1851d281259719ef77d785cca60d2c1113750c15f673625bc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - Samtools
  - Tophat2
  - UROBORUS
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pearson-correlation-abundance-accuracy-assessment

## Summary

Benchmarking skill that scores circRNA isoform quantification accuracy by computing per-locus Pearson correlations between theoretical (simulated ground-truth) and CircAST-estimated relative abundances (TPM/FPKM) across simulated conditions, then comparing against raw junction-read-count correlations. CircAST's reported accuracy range on the 4-loci, 8-condition simulation is 0.849-0.993.

## When to use

Use when simulated RNA-seq data carry known theoretical circRNA isoform abundances and you must score how faithfully a quantifier (CircAST FPKM/TPM) recovers per-locus relative isoform fractions across conditions, or benchmark it against junction-read counting.

## When NOT to use

- The dataset is real RNA-seq without simulated ground truth - no theoretical isoform abundances exist to correlate against, so the Pearson-based accuracy check cannot be performed.
- The circRNA locus has a single isoform - relative isoform fractions are trivially 1.0 and Pearson correlation across conditions is meaningless.
- The target loci are intronic or intergenic circRNAs, which CircAST's exon-annotated pipeline may miss, invalidating the comparison to theoretical values.

## Inputs

- tophat2 paired-end alignments per simulated condition (accepted_hits.bam), name-sorted and converted to accepted_hits.sorted.sam
- circRNA back-splice junction lists per condition in CircAST format from CIRCexplorer2, CIRI2, or UROBORUS (junction.txt)
- UCSC-compatible gene annotation GTF, identical to the one used by the aligner and BSJ detector
- Theoretical relative isoform abundances (ground truth) for the 4 simulated circRNA loci with two or more isoforms each, across the 8 conditions
- Simulated read length (CircAST -L parameter)

## Outputs

- Per-condition CircAST result table: 13 tab-separated columns including junction read counts, FPKM, TPM, and assigned read counts per circular transcript
- Per-locus relative isoform fractions derived from TPM/FPKM across the 8 conditions
- Pearson correlation coefficients (scipy.stats.pearsonr) between theoretical and CircAST-estimated abundances per locus
- Benchmark Pearson correlations computed from raw junction read counts alone for comparison

## How to apply

Preprocess each of the 8 simulated conditions: name-sort the tophat2 accepted_hits.bam with 'samtools sort -n' and convert it via 'samtools view' to accepted_hits.sorted.sam. Run CircAST per condition (python CircAST.py -L <read_length> -T <threshold, default 10> -G genes.gtf -F accepted_hits.sorted.sam -J junction.txt), where junction.txt is the back-splice junction list from CIRCexplorer2, CIRI2, or UROBORUS in CircAST format and the GTF is the same UCSC-compatible file used by the aligner and detector. Parse the 13-column CircAST output table to extract junction read counts, assigned read counts, FPKM, and TPM for each circular transcript. Convert TPM (or FPKM) into per-locus relative isoform fractions and compute Pearson correlation coefficients with scipy.stats.pearsonr between theoretical and estimated fractions across the 8 conditions for each of the 4 simulated loci (two or more isoforms per locus). Recompute the same correlations from raw junction read counts alone; accept the assessment when CircAST's per-locus correlations fall in the reported 0.849-0.993 range and exceed the junction-read-count correlations, confirming CircAST outperforms junction-read-based quantification.

## Related tools

- **CircAST** (Assembles circular isoforms via splice graphs and outputs per-transcript junction read counts, assigned read counts, FPKM, and TPM that feed the Pearson correlation assessment) — https://github.com/xiaofengsong/CircAST
- **Tophat2** (Aligns simulated paired-end reads per condition to produce accepted_hits.bam, the alignment input CircAST consumes)
- **Samtools** (Name-sorts accepted_hits.bam ('samtools sort -n') and converts it to accepted_hits.sorted.sam via 'samtools view')
- **CIRCexplorer2** (Generates the back-splice junction list (circ_out/denovo/annotated_circ.txt) per condition in CircAST input format)
- **CIRI2** (Generates the back-splice junction list (result.txt) per condition in CircAST input format)
- **UROBORUS** (Generates the back-splice junction list (circRNA_list.txt) per condition in CircAST input format)
- **scipy** (scipy.stats.pearsonr computes the Pearson correlations between theoretical and estimated relative isoform fractions)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- CircAST per-locus Pearson correlations between theoretical and estimated relative abundances fall within the reported 0.849-0.993 range across the 8 conditions.
- At each locus, CircAST's correlation exceeds the correlation computed from raw junction read counts alone.
- CircAST output parses to the documented 13-column, tab-separated schema containing junction read counts, FPKM, TPM, and assigned read counts.
- Per-locus relative isoform fractions derived from TPM/FPKM sum to 1.0 for each multi-isoform locus in each condition.
- The SAM input was name-sorted ('samtools sort -n') and the same UCSC-compatible GTF was used for tophat2, the BSJ detector, and CircAST.

## Limitations

- CircAST can be applied only to RNase R-treated samples; remnant linear mRNA can affect assembly performance and generate false transcripts that distort estimated fractions.
- CircAST may miss intronic or intergenic circular transcripts, limiting the benchmark to exon-annotated loci.
- The 0.849-0.993 correlation range was established on the 4-loci, 8-condition simulation with two or more isoforms per locus; it is not a general guarantee for other simulations.
- Junction lists from BSJ detectors other than CIRCexplorer2/CIRI2/UROBORUS must be converted to CircAST input file format before use.
- CircAST v1.0.x is implemented in Python 2 (Anaconda 2.5.0+, Python 2.7), which may complicate reuse of the scipy-based correlation step.

## Evidence

- [abstract] CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [methods] CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS: "CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS"
- [readme] samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [readme] CircAST is compatible with popular upstream circRNA detection tools (CIRCexplorer2, CIRI2, UROBORUS).: "CircAST is compatible with popular upstream circRNA detection tools (CIRCexplorer2, CIRI2, UROBORUS)."
- [discussion] CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over.: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
- [discussion] The current version of the CircAST pipeline can assemble full-length sequences and quantify exon-annotated circular transcripts, but may miss intronic or intergenic circular transcripts.: "The current version of the CircAST pipeline can assemble full-length sequences and quantify exon-annotated circular transcripts, but may miss intronic or intergenic circular transcripts."
