---
name: circrna-isoform-quantification-accuracy-analysis
description: Use when simulated RNA-seq data with known ground-truth circRNA isoforms
  have been processed by CircAST and you must quantify how BSJ overlap complexity
  (relation index) affects assembly and quantification accuracy.
when_to_use_negative:
- Data is real experimental RNA-seq without simulated ground-truth isoforms, so sensitivity/precision
  against truth cannot be computed
- Input is not CircAST output — raw FASTQ, SAM/BAM, or upstream BSJ detector output
  (CIRCexplorer2/CIRI2/UROBORUS lists) requires assembly by CircAST first
- Goal is circRNA detection/BSJ discovery rather than evaluating isoform-level assembly
  and quantification accuracy
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Assembles alternatively spliced circRNA transcripts and estimates FPKM/TPM
    expression via multiple splice graphs and EM-based read allocation on simulated
    data; its output table provides the assembled isoforms and expression estimates
    evaluated per RI bin
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:d4dda58e5f7d02fe08dbe200f7ce4613d3880a64652bdaeab93cf35a9b9633f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# circrna-isoform-quantification-accuracy-analysis

## Summary

Benchmarks CircAST's full-length circRNA assembly and isoform quantification on simulated data by stratifying back-splice junction (BSJ) sites by relation index (RI) — the size of the BSJ overlap equivalence class — and computing sensitivity, precision, and theoretical-vs-estimated expression correlation per RI bin.

## When to use

Use when simulated RNA-seq data with known ground-truth circRNA isoforms have been processed by CircAST and you must quantify how BSJ overlap complexity (relation index) affects assembly and quantification accuracy.

## When NOT to use

- Data is real experimental RNA-seq without simulated ground-truth isoforms, so sensitivity/precision against truth cannot be computed
- Input is not CircAST output — raw FASTQ, SAM/BAM, or upstream BSJ detector output (CIRCexplorer2/CIRI2/UROBORUS lists) requires assembly by CircAST first
- Goal is circRNA detection/BSJ discovery rather than evaluating isoform-level assembly and quantification accuracy

## Inputs

- CircAST output table on simulated data (chromosome, start, end, circular transcript name, junction read counts, FPKM, TPM, assigned reads)
- Simulated ground-truth circular transcript isoforms
- Genomic coordinates of simulated back-splice junction (BSJ) sites

## Outputs

- Per-site RI assignments for simulated BSJ sites
- RI-stratified metrics table (sensitivity, precision, expression correlation per RI bin) written to ri_performance_metrics.csv
- Site counts per RI bin (RI 1-4 vs RI > 4)

## How to apply

Load the CircAST output table (columns: chromosome, start, end, circular transcript name, junction read counts, FPKM, TPM, assigned reads) with pandas. Build the BSJ overlap relation between simulated back-splice junction sites from their genomic coordinates and compute each site's relation index as the cardinality of its equivalence class under this overlap relation. Stratify BSJ sites into RI bins (RI 1-4 vs RI > 4) and compare CircAST-assembled circular transcripts against the simulated ground truth per bin to compute sensitivity (fraction of true isoforms recovered) and precision (fraction of assembled isoforms that are true). Per bin, assess quantification accuracy as the correlation between theoretical and CircAST-estimated FPKM/TPM expression levels. Correct application reproduces the reported pattern: sensitivity above 93% for RI 1-4 declining to about 86% for RI > 4, within the simulation ranges (sensitivity 85.63%-94.32%, precision 81.96%-87.55%, correlation 0.85-0.99).

## Related tools

- **CircAST** (Assembles alternatively spliced circRNA transcripts and estimates FPKM/TPM expression via multiple splice graphs and EM-based read allocation on simulated data; its output table provides the assembled isoforms and expression estimates evaluated per RI bin) — https://github.com/xiaofengsong/CircAST

## Examples

```
python CircAST.py -L 100 -T 5 -G genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Sensitivity of circular transcript reconstruction is above 93% for RI bins 1-4 and drops to about 86% for RI > 4
- Overall simulation metrics fall within the reported ranges: sensitivity 85.63%-94.32% and precision 81.96%-87.55%
- Per-bin correlation between theoretical and CircAST-estimated expression (FPKM/TPM) lies in the reported 0.85-0.99 range
- ri_performance_metrics.csv contains a non-empty RI assignment for every simulated BSJ site and counts per RI bin sum to the total site count
- Each equivalence-class cardinality (RI) is consistent with the BSJ genomic coordinates: sites with identical or overlapping coordinates share the same RI value

## Limitations

- The reported sensitivity/precision ranges are simulation-based; on real RNase R-treated samples remnant linear mRNA can affect assembly and generate false transcripts
- CircAST can be applied only to RNase R-treated samples, which constrains the data regime the benchmark represents
- CircAST assembles only exon-annotated circular transcripts and may miss intronic or intergenic circular transcripts, biasing the accuracy assessment toward exonic circRNAs
- Accuracy depends on upstream BSJ detection; other circRNA junction detection tools require output format conversion to CircAST input format

## Evidence

- [abstract] Sensitivity/precision ranges: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] Quantification accuracy: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [readme] Output table format: "The first 3 columns are the same with bed file format."
- [abstract] Read allocation for quantification: "By assigning reads to specific isoforms, CircAST quantified the expression of circRNA isoforms"
