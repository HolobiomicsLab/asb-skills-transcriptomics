---
name: multi-tissue-expression-summary-statistics
description: Use when multiple ribo− RNA-seq samples have been quantified with CLEAR
  (FPBcirc/FPBlinear/CIRCscore) and the goal is cross-tissue comparison, per-tissue
  summary statistics, or reproducing tissue-enrichment patterns like Figure 5A.
when_to_use_negative:
- 'Input is poly(A)−/ribo− or RNase R-treated RNA-seq: the article notes polyadenylated
  linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation'
- Only a single sample/tissue is available and no cross-tissue comparison or summary
  is intended
- The quant outputs have not been produced by CLEAR/circ_quant (no FPBcirc, FPBlinear,
  or CIRCscore columns) — e.g. raw CIRCexplorer2 back-splicing calls without quantification
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Produces the per-sample quant.txt files (FPBcirc, FPBlinear, CIRCscore) that
    are aggregated across tissues
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Generates the circular RNA calls (>=2.3.6) that circ_quant consumes before
    tissue-level summarization
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns ribo− RNA-seq reads; the mapped BAM (unmapped reads removed) is input
    to circ_quant for linear SJ quantification
- name: StringTie
  role: Installation requirement of the CLEAR pipeline used for transcript assembly
    during quantification
  repo: https://ccb.jhu.edu/software/stringtie
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:e1e4e1d582a8b285c0105580a05d4feefced77aad67ee4d6fc714c6f34cb10cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - StringTie
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-tissue-expression-summary-statistics

## Summary

Aggregates per-sample CLEAR quant.txt outputs (readNumber, FPBcirc, FPBlinear, CIRCscore) across multiple tissues into a consolidated summary table, reproducing tissue-enrichment patterns such as brain-enriched circRNAs from the CIRCexplorer3-CLEAR ENCODE 12-tissue analysis.

## When to use

Use when multiple ribo− RNA-seq samples have been quantified with CLEAR (FPBcirc/FPBlinear/CIRCscore) and the goal is cross-tissue comparison, per-tissue summary statistics, or reproducing tissue-enrichment patterns like Figure 5A.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq: the article notes polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Only a single sample/tissue is available and no cross-tissue comparison or summary is intended
- The quant outputs have not been produced by CLEAR/circ_quant (no FPBcirc, FPBlinear, or CIRCscore columns) — e.g. raw CIRCexplorer2 back-splicing calls without quantification

## Inputs

- Per-tissue CLEAR quant.txt files (BED12-like format with readNumber, FPBcirc, FPBlinear, CIRCscore columns)
- Tissue/sample metadata mapping each quant.txt to a tissue label (e.g. 12 ENCODE tissues, brain vs non-brain)
- FPB threshold (default 1) and circ-to-linear ratio (default 1) used during circ_quant

## Outputs

- Consolidated per-tissue summary table (e.g. encode_tissue_circ_quant.tsv) with circRNA counts, median FPBcirc, and CIRCscore distributions
- Fraction of FPBcirc ≥ 1 circRNAs with CIRCscore ≥ 1 per tissue
- Brain vs non-brain and within-brain (cortex/occipital/diencephalon vs cerebellum) enrichment comparisons

## How to apply

Parse each tissue's quant.txt output from circ_quant/clear_quant, extracting readNumber, FPBcirc, FPBlinear, and CIRCscore for every circRNA. Retain circRNAs with FPBcirc ≥ 1 (the default FPB threshold used to filter linear SJ) and compute per-tissue circRNA counts, median FPBcirc, and CIRCscore distributions, including the fraction of circRNAs with CIRCscore ≥ 1 (CIRCscore = FPBcirc/FPBlinear). Compare brain vs non-brain tissues, and within brain compare cortex/occipital/diencephalon vs cerebellum, to detect tissue-enrichment patterns. Write the consolidated per-tissue metrics to a single table (e.g. encode_tissue_circ_quant.tsv). Rationale: FPB normalizes fragments mapped to BSJ and SJ sites individually, so FPBcirc and FPBlinear are directly comparable within and across samples, unlike FPM vs FPKM.

## Related tools

- **CIRCexplorer3-CLEAR** (Produces the per-sample quant.txt files (FPBcirc, FPBlinear, CIRCscore) that are aggregated across tissues) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Generates the circular RNA calls (>=2.3.6) that circ_quant consumes before tissue-level summarization) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns ribo− RNA-seq reads; the mapped BAM (unmapped reads removed) is input to circ_quant for linear SJ quantification)
- **StringTie** (Installation requirement of the CLEAR pipeline used for transcript assembly during quantification) — https://ccb.jhu.edu/software/stringtie

## Examples

```
circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -t -r annotation.refFlat -o quant.txt
```

## Evaluation signals

- Each tissue's quant.txt contains the expected columns: FPBcirc ('Expression of circRNA') and CIRCscore ('Relative expression of circRNA')
- The fraction of FPBcirc ≥ 1 circRNAs with CIRCscore ≥ 1 falls in the ~10%–20% range reported in Figure 5A
- Summary table rows sum to the number of circRNAs with FPBcirc ≥ 1 in each per-tissue quant.txt (no dropped or duplicated records)
- Median FPBcirc per tissue is finite and consistent with CIRCscore = FPBcirc / FPBlinear within rounding
- Brain tissues (especially cortex, occipital, diencephalon) show higher FPB and CIRCscore values than non-brain tissues, matching the reported enrichment pattern

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- No software version numbers stated for CLEAR or CIRCexplorer2; reproducibility depends on CIRCexplorer2 >=2.3.6 per the README
- hisat_aligned.bam must not contain unmapped reads before circ_quant, or downstream quantification is invalid

## Evidence

- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [methods] Threshold of FPB for choose circRNAs to filter linear SJ.[default: 1]: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
- [readme] FPBcirc column description in quant.txt output: "FPBcirc     | Expression of circRNA                 |"
- [readme] CIRCscore column description in quant.txt output: "CIRCscore   | Relative expression of circRNA        |"
- [readme] clear_quant example invocation from README: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
