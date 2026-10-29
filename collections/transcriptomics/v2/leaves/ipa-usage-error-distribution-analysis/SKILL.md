---
name: ipa-usage-error-distribution-analysis
description: 'Decision support for choosing an IPA quantification method: you have
  simulated RNA-seq (Polyester, 10X–50X coverage, ≥5 replicates) with predefined polyA
  sites and known ground-truth relative usage, and need to compare tools'' quantification
  error via |ΔPAU| distributions.'
when_to_use_negative:
- Input is real RNA-seq data without known ground-truth relative usage — |ΔPAU| cannot
  be computed, so use Iso-seq or 3'-end sequencing validation instead
- The question is IPA site identification accuracy only (sensitivity/precision of
  site calls), not quantification error of usage values
- The comparison is between general APA/3'UTR-APA quantification tools (MISO, QAPA,
  LABRAT, DaPars), which are limited to 3'UTR APA and cannot identify novel IPA isoforms
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Polyester
  role: R package used to simulate RNA-seq reads at 10X–50X coverage (10X increments,
    five replicates per level) from transcripts with predefined polyA sites and known
    ground-truth relative usage
- name: InPACT
  role: Identifies IPA sites (skipped/composite), assembles novel IPA transcripts
    (InPACT_transcript), and computes PAU usage estimates (InPACT_quantify) whose
    error against ground truth is scored
  repo: github.com/YY-TMU/InPACT
- name: IPAFinder
  role: Comparator tool run on the same simulated data to estimate IPA site usage;
    its |ΔPAU| error distribution is plotted against InPACT's
- name: APAIQ
  role: Comparator tool leveraging DNA sequence and RNA-seq read coverage to identify
    IPA sites and estimate usage in the benchmark
- name: Salmon
  role: Indexes the merged transcriptome (gffread-extracted transcripts.fa, k=31)
    and quantifies transcript abundance (quant.sf) feeding InPACT_quantify's PAU calculation
  repo: github.com/COMBINE-lab/salmon
- name: gffread
  role: Extracts transcript sequences from the merged GTF for the Salmon index
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:860e55a2951b30095236b9d0a80ce137c5b3ef690da200f455218aadda9ba275
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APAIQ
  - BEDTools closest
  - IPAFinder
  - InPACT
  - Polyester
  - Salmon
  - gffread
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ipa-usage-error-distribution-analysis

## Summary

Benchmark how accurately IPA tools (InPACT, IPAFinder, APAIQ) quantify relative usage of intronic polyadenylation sites by computing per-site absolute PAU error (|ΔPAU|) against ground-truth relative usage on Polyester-simulated RNA-seq data, then visualizing cumulative error distributions.

## When to use

Decision support for choosing an IPA quantification method: you have simulated RNA-seq (Polyester, 10X–50X coverage, ≥5 replicates) with predefined polyA sites and known ground-truth relative usage, and need to compare tools' quantification error via |ΔPAU| distributions.

## When NOT to use

- Input is real RNA-seq data without known ground-truth relative usage — |ΔPAU| cannot be computed, so use Iso-seq or 3'-end sequencing validation instead
- The question is IPA site identification accuracy only (sensitivity/precision of site calls), not quantification error of usage values
- The comparison is between general APA/3'UTR-APA quantification tools (MISO, QAPA, LABRAT, DaPars), which are limited to 3'UTR APA and cannot identify novel IPA isoforms

## Inputs

- Polyester-simulated RNA-seq FASTQ/BAM datasets at 10X–50X coverage (10X increments, five replicates per level) derived from a transcript set with predefined polyA sites and ground-truth relative usage (PAU) per IPA site
- Reference gene annotation GTF (e.g., RefSeq GRCh38/hg38.ncbiRefSeq.gtf) and genome FASTA
- Precomputed candidate IPA sites SAF file (e.g., InPACT_polyAsites.hg38.saf)

## Outputs

- Per-tool, per-site IPA usage estimates (PAU) from InPACT_quantify output (terminal exon, IPA type, gene, PolyAsite, IPA usage columns)
- Per-site quantification errors |ΔPAU| relative to ground-truth relative usage, per tool and coverage level
- Cumulative distribution plots of |ΔPAU| error across all identified IPA sites comparing InPACT and IPAFinder
- Sensitivity/precision-versus-coverage summary tables used alongside the error distributions

## How to apply

First simulate RNA-seq datasets with the R package Polyester at coverage levels from 10X to 50X in 10X increments (five replicates per level) from a transcript set with predefined polyA sites, taking the ground-truth relative usage of the predefined sites as reference. Run each tool (InPACT, APAIQ, IPAFinder) on the simulated data to both identify IPA sites and estimate relative usage of each site; for InPACT the usage pipeline goes through InPACT_transcript isoform assembly, gffread transcript extraction, Salmon index/quant, and InPACT_quantify to obtain PAU estimates from the quant.sf TPMs. Compute per-site quantification error as the absolute difference between estimated relative usage and ground truth (|ΔPAU|) for each tool at each coverage level. Plot cumulative distributions of |ΔPAU| across all identified IPA sites comparing the tools. The analysis is judged successful when the figures reproduce the reported trend that InPACT attains the lowest IPA-usage error versus IPAFinder and APAIQ across the 10X–50X range.

## Related tools

- **Polyester** (R package used to simulate RNA-seq reads at 10X–50X coverage (10X increments, five replicates per level) from transcripts with predefined polyA sites and known ground-truth relative usage)
- **InPACT** (Identifies IPA sites (skipped/composite), assembles novel IPA transcripts (InPACT_transcript), and computes PAU usage estimates (InPACT_quantify) whose error against ground truth is scored) — github.com/YY-TMU/InPACT
- **IPAFinder** (Comparator tool run on the same simulated data to estimate IPA site usage; its |ΔPAU| error distribution is plotted against InPACT's)
- **APAIQ** (Comparator tool leveraging DNA sequence and RNA-seq read coverage to identify IPA sites and estimate usage in the benchmark)
- **Salmon** (Indexes the merged transcriptome (gffread-extracted transcripts.fa, k=31) and quantifies transcript abundance (quant.sf) feeding InPACT_quantify's PAU calculation) — github.com/COMBINE-lab/salmon
- **gffread** (Extracts transcript sequences from the merged GTF for the Salmon index)

## Examples

```
salmon quant -i transcripts_index -l A -1 reads1.fq -2 reads2.fq --validateMappings -o transcripts_quant; InPACT_quantify --transcript_tpm transcripts_quant/quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt
```

## Evaluation signals

- Cumulative |ΔPAU| distribution for InPACT lies to the left of (below) IPAFinder and APAIQ curves across the full error range, reproducing the reported lowest IPA-usage error
- InPACT sensitivity and precision remain highest across all 10X–50X coverage levels (paper reports ~90% of IPA sites identified at 50X coverage)
- Predicted sites within 50 nt of a predefined polyA site are classified as true positives (Sensitivity = TP/Predefined, Precision = TP/(TP+FP)); per-coverage values should be averaged over the five replicates
- InPACT_quantify output has the expected columns (Terminal exon, IPA type, Gene, PolyAsite, IPA usage) and PAU values are comparable in scale to the ground-truth relative usage reference

## Limitations

- IPAFinder depends on read coverage fluctuations requiring adequate sequencing depth, so its quantification error is expected to grow at low coverage (10X–20X)
- No version numbers or explicit parameters for the simulations or comparative benchmarks were reported in the article, complicating exact reproduction
- APAIQ is not specifically designed for IPA analysis and cannot assemble IPA isoforms or differentiate skipped from composite IPA events, which constrains its usage estimation

## Evidence

- [methods] The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment: "The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment"
- [methods] compute the per-site quantification error as the absolute difference between estimated relative usage and ground truth (|ΔPAU|): [evidence span withheld — anchor status: not_found]
- [discussion] InPACT exhibited improved accuracy in quantifying IPA events with a lower error rate compared to IPAFinder and APAIQ: "InPACT exhibited improved accuracy in quantifying IPA events with a lower error rate compared to IPAFinder and APAIQ"
- [intro] IPAFinder relies on read coverage fluctuations, which necessitates adequate sequencing depth: "IPAFinder relies on read coverage fluctuations, which necessitates adequate sequencing depth, and the precision of identified IPA sites raises concerns"
- [readme] InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt: "InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt"
- [readme] salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
