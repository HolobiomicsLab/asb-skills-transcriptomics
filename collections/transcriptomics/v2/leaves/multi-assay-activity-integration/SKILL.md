---
name: multi-assay-activity-integration
description: Use when you have candidate enhancer regions, several activity assay
  signals, a Hi-C contact matrix, and gene annotations (e.g. Gencode GTF), and need
  enhancer-gene interaction scores compared across assay/contact-matrix combinations.
when_to_use_negative:
- Your goal is TF-affinity summarisation on the gene level (the other independent
  STARE task) rather than enhancer-gene interaction ABC scoring
- You have no chromatin contact matrix available — the adapted activity adjustment
  depends on the number of contacts between enhancer and candidate target genes
- Your activity data is single-cell (e.g. scATAC-seq); that use case is handled by
  combining gABC with GAZE rather than this bulk multi-assay benchmarking skill
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: STARE
  role: Scores enhancer-gene interactions with the regular and adapted (gABC) ABC
    model across activity-assay and contact-matrix combinations (-q, -i, -d, -n, -f
    flags)
  repo: https://github.com/schulzlab/STARE
- name: pROC
  role: Computes and compares precision-recall/ROC curves for ABC vs. gABC per dataset/condition
- name: Juicer
  role: Data extraction and normalization of .hic files into the normalized chromatin
    contact files STARE consumes
  repo: https://github.com/aidenlab/juicer
- name: bedtools
  role: Genome arithmetic on candidate regions when preparing inputs
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:2177589092027ea3472a28b369473092721661a90e11937fd808b91689512f32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Juicer
  - STARE
  - bedtools
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-assay-activity-integration

## Summary

Integrates multiple enhancer-activity assays (DNase-seq, ATAC-seq, H3K27ac ChIP-seq) with chromatin-contact matrices to score enhancer-gene interactions using the ABC and adapted (gABC) Activity-By-Contact models via STARE, and benchmarks them against CRISPRi perturbation labels using AUPRC.

## When to use

Use when you have candidate enhancer regions, several activity assay signals, a Hi-C contact matrix, and gene annotations (e.g. Gencode GTF), and need enhancer-gene interaction scores compared across assay/contact-matrix combinations.

## When NOT to use

- Your goal is TF-affinity summarisation on the gene level (the other independent STARE task) rather than enhancer-gene interaction ABC scoring
- You have no chromatin contact matrix available — the adapted activity adjustment depends on the number of contacts between enhancer and candidate target genes
- Your activity data is single-cell (e.g. scATAC-seq); that use case is handled by combining gABC with GAZE rather than this bulk multi-assay benchmarking skill

## Inputs

- Candidate-enhancer BED file (regions to be scored)
- Gene annotation in Gencode GTF format
- Activity assay signal columns (DNase-seq, ATAC-seq, H3K27ac ChIP-seq)
- Chromatin contact matrix in STARE binary format: Fulco et al. 2019 average hg19 Hi-C matrix (average_hic.v2.191020) or Gschwind et al. 2023 averaged hg38 ENCODE matrix (ENCFF134PUN)
- CRISPRi perturbation-effect measurements (Gasperini 2019, Schraivogel 2020, Fulco 2019) for defining positives/negatives

## Outputs

- Enhancer-gene interaction scores per condition: regular ABC (-q False) and gABC (-q True, -i all_tss)
- Precision-recall curves per dataset/assay/contact-matrix combination (pROC)
- Comparison table auprc_comparison.csv with AUPRC per dataset and condition

## How to apply

For each CRISPRi validation dataset (Gasperini 2019, Schraivogel 2020, Fulco 2019), run STARE on the candidate-enhancer bed file with a Gencode GTF (-a), a 5 Mb window centered at the 5' TSS (-w 5000000) and a score cutoff -t 0.02. Score interactions with the regular ABC model using -q False (original activity, no adaptation) and with the generalized gABC model using -q True and -i all_tss (adapted activity averaged across all annotated TSS), enabling the contact pseudocount (-d True). Repeat scoring across every combination of activity-assay columns (-n, e.g. DNase-seq, ATAC-seq, H3K27ac ChIP-seq) and contact matrices (-f, e.g. the Fulco et al. average hg19 Hi-C matrix average_hic.v2.191020 or the Gschwind et al. hg38 ENCODE matrix ENCFF134PUN). The rationale is that the ABC score yields higher accuracy when enhancer activity is adapted according to the number of contacts the enhancer has to its candidate target genes. Finally, label predicted pairs against measured CRISPRi perturbation effects and compute precision-recall curves per dataset/assay/contact condition with pROC, tabulating AUPRC values into auprc_comparison.csv.

## Related tools

- **STARE** (Scores enhancer-gene interactions with the regular and adapted (gABC) ABC model across activity-assay and contact-matrix combinations (-q, -i, -d, -n, -f flags)) — https://github.com/schulzlab/STARE
- **pROC** (Computes and compares precision-recall/ROC curves for ABC vs. gABC per dataset/condition)
- **Juicer** (Data extraction and normalization of .hic files into the normalized chromatin contact files STARE consumes) — https://github.com/aidenlab/juicer
- **bedtools** (Genome arithmetic on candidate regions when preparing inputs) — https://github.com/arq5x/bedtools2

## Examples

```
stare-abc -b candidates.bed -a gencode.gtf -w 5000000 -t 0.02 -n DNase,ATAC,H3K27ac -f average_hic.v2.191020 -q True -i all_tss -d True
```

## Evaluation signals

- gABC should yield on average a 0.107 higher AUPRC than the regular ABC score across the three CRISPRi validation datasets (Gasperini, Schraivogel, Fulco)
- gABC should outperform the regular ABC score on every validation dataset, not just on average
- auprc_comparison.csv must contain AUPRC entries for every dataset × activity-assay × contact-matrix combination evaluated
- Precision-recall curves computed with pROC should reproduce per-condition rankings when positives are defined by measured CRISPRi perturbation effects
- Conditions using a single activity assay should remain comparably accurate, per the paper's finding on one-assay accuracy

## Limitations

- Accuracy gains come from adapting enhancer activity by contact count and using all annotated TSSs; with only one activity assay the model is 'comparably accurate', so adding assays is not guaranteed to improve results in all settings
- Contact matrices must be prepared in STARE's expected binary format; .hic files require Juicer normalization (e.g. KR) beforehand
- The benchmark is anchored to CRISPRi perturbation datasets in K562; AUPRC deltas may not transfer to other cell types or perturbation modalities

## Evidence

- [abstract] We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes"
- [abstract] Further, we show that the model is comparably accurate with only one assay to measure enhancer activity.: "Further, we show that the model is comparably accurate with only one assay to measure enhancer activity."
- [methods] the average hg19 Hi-C matrix provided by Fulco et al. (2019): "the average hg19 Hi-C matrix provided by `Fulco et al. (2019) <https://doi.org/10.1038/s41588-019-0538-0>`_"
- [supplementary] pROC: An open-source package for r and s+ to analyze and compare ROC curves: "pROC: An open-source package for r and s+ to analyze and compare ROC curves"
- [methods] ./Code/Juicebox_normalization.sh -j <path_juicer_jar_file> -h <hic_path> -n KR -d <out_path> -c <chromosomes> -b <bin_size>: "./Code/Juicebox_normalization.sh -j <path_juicer_jar_file> -h <hic_path> -n KR -d <out_path> -c <chromosomes> -b <bin_size>"
