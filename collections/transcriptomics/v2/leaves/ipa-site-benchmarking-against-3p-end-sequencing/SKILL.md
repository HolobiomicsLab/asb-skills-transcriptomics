---
name: ipa-site-benchmarking-against-3p-end-sequencing
description: Use when RNA-seq-derived IPA site predictions from multiple tools need
  accuracy assessment against experimentally determined polyA site coordinates, or
  when deciding which IPA caller to adopt.
when_to_use_negative:
- When the goal is quantification of already-annotated 3'UTR APA usage rather than
  identification of novel intronic IPA sites (tools like DaPars, QAPA, LABRAT are
  limited to 3'UTR APA or quantification only)
- When no experimental ground truth (3'-end sequencing or Iso-seq data) is available
  for the evaluated cell type or tissue
- When sequencing depth is very low — IPAFinder depends on read coverage fluctuations
  requiring adequate depth, distorting benchmark comparisons
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: InPACT
  role: Primary IPA site identification and quantification method under benchmark;
    run with 'InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P
    5'
  repo: https://github.com/YY-TMU/InPACT
- name: APAIQ
  role: Comparison method; leverages DNA sequence and RNA-seq read coverage to identify
    IPA sites
- name: IPAFinder
  role: Comparison method; identifies IPA sites from RNA-seq read coverage fluctuations
- name: BEDTools closest
  role: Matches predicted IPA sites to ground-truth polyA sites within 50 nt for TP
    classification
- name: liftOver
  role: Converts ground-truth polyA site coordinates to GRCh38 assembly
- name: Polyester
  role: R package for simulating RNA-seq data at 10x-50x coverage for identification
    and quantification benchmarks
- name: Salmon
  role: Transcript-level abundance quantification ('salmon quant') used to compute
    IPA usage (PAU) for |ΔPAU| error assessment
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:5b83a5008ec8f299c7677fabdfc34deaa03dcf862d2b9690a3271f2c77faa474
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
  - Salmon
  - liftOver
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ipa-site-benchmarking-against-3p-end-sequencing

## Summary

A benchmarking skill for evaluating intronic polyadenylation (IPA) site prediction accuracy by comparing tool outputs (InPACT, APAIQ, IPAFinder) against ground-truth polyA sites from 3'-end sequencing (A-seq, 3P-seq, polyA-seq) and long-read Iso-seq data.

## When to use

Use when RNA-seq-derived IPA site predictions from multiple tools need accuracy assessment against experimentally determined polyA site coordinates, or when deciding which IPA caller to adopt.

## When NOT to use

- When the goal is quantification of already-annotated 3'UTR APA usage rather than identification of novel intronic IPA sites (tools like DaPars, QAPA, LABRAT are limited to 3'UTR APA or quantification only)
- When no experimental ground truth (3'-end sequencing or Iso-seq data) is available for the evaluated cell type or tissue
- When sequencing depth is very low — IPAFinder depends on read coverage fluctuations requiring adequate depth, distorting benchmark comparisons

## Inputs

- Sorted coordinate BAM files of matched RNA-seq datasets (HEK293, MAQC UHR/brain, small airway epithelial cells)
- Ground-truth polyA site coordinates from A-seq, 3P-seq, and polyA-seq (GEO download, liftOver to GRCh38)
- Iso-seq isoform annotation (GSE167486) with directly extracted polyA sites
- GRCh38 RefSeq GTF annotation (hg38.109.20211119.ncbiRefSeq.gtf.gz)
- InPACT_polyAsites.hg38.saf candidate polyA site annotation

## Outputs

- Summary table of true-positive percentages, sensitivity/precision values, and Iso-seq validation rates per tool
- Cumulative distribution plots of IPA usage error (|ΔPAU|) per tool
- Per-tool identification sensitivity and precision at each simulated coverage level (10x-50x)

## How to apply

Download ground-truth polyA site coordinates from A-seq, 3P-seq, and polyA-seq datasets (e.g., GEO accessions) and convert them to GRCh38 with liftOver. Run InPACT, APAIQ, and IPAFinder on matched RNA-seq datasets (HEK293, MAQC UHR/brain, small airway epithelial cells). Match predicted sites to ground truth using BEDTools closest, classifying predictions within 50 nt of a predefined polyA site as true positives, then compute Sensitivity = TP/Predefined and Precision = TP/(TP+FP). For Iso-seq validation, compare predictions to polyA sites directly extracted from Iso-seq isoform annotation on GSE167486. Supplement with simulated RNA-seq (R package Polyester at 10x-50x coverage in 10x increments, five replicates per level) computing per-tool identification sensitivity/precision plus IPA usage error (|ΔPAU|), and accept a tool as superior when its metrics exceed those of the alternatives.

## Related tools

- **InPACT** (Primary IPA site identification and quantification method under benchmark; run with 'InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5') — https://github.com/YY-TMU/InPACT
- **APAIQ** (Comparison method; leverages DNA sequence and RNA-seq read coverage to identify IPA sites)
- **IPAFinder** (Comparison method; identifies IPA sites from RNA-seq read coverage fluctuations)
- **BEDTools closest** (Matches predicted IPA sites to ground-truth polyA sites within 50 nt for TP classification)
- **liftOver** (Converts ground-truth polyA site coordinates to GRCh38 assembly)
- **Polyester** (R package for simulating RNA-seq data at 10x-50x coverage for identification and quantification benchmarks)
- **Salmon** (Transcript-level abundance quantification ('salmon quant') used to compute IPA usage (PAU) for |ΔPAU| error assessment) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- Over 60% of InPACT-identified IPA sites fall within 50 nt of ground-truth polyA sites, versus ~30% for APAIQ and ~20% for IPAFinder
- Sensitivity = TP/Predefined and Precision = TP/(TP+FP) are higher for the evaluated tool than for APAIQ and IPAFinder on the same datasets
- Iso-seq validation rate (fraction of predicted sites matching polyA sites extracted from Iso-seq isoform annotation on GSE167486) exceeds that of competing tools
- Per-tool IPA usage error |ΔPAU| cumulative distribution is shifted toward lower error compared to APAIQ and IPAFinder
- Across all simulated coverage levels (10x-50x in 10x increments, five replicates), InPACT sensitivity/precision curves remain above competitor curves

## Limitations

- No version number is given for InPACT or any compared tool, complicating exact reproduction
- No accession or identifier is provided for the human fetal bone marrow scRNA-seq dataset
- No parameters or thresholds are reported in the discussion for the simulations or comparative benchmarks
- 3'-end sequencing techniques are not widely used and available ground-truth data remain relatively scarce
- APAIQ is not specifically designed for IPA analysis and cannot assemble IPA isoforms or differentiate skipped from composite IPA events, which may bias comparative results

## Evidence

- [other] Classification of true positives within 50 nt of predefined polyA sites with BEDTools closest: [evidence span withheld — anchor status: not_found]
- [intro] 3'-end and long-read ground truth used for benchmarking: "we show that InPACT outperforms IPAFinder in identifying and quantifying IPA with 3’-end sequencing data, long-read sequencing data and simulated RNA-seq data as benchmarks"
- [methods] Simulation coverage design for benchmarking: "The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment"
- [readme] InPACT_polyAsites.hg38.saf used as candidate polyA site annotation: "--ipa_sites/-s: Regions from 20nt upstream to potential polyA sites, with columns for gene, chromosome, start, end, and strand."
- [discussion] Comparative accuracy finding against multiple ground truths: "InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq"
