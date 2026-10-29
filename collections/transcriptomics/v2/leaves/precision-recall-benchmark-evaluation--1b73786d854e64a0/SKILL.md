---
name: precision-recall-benchmark-evaluation
description: Use when a predicted sample-specific transcriptome (e.g., name_aptardi.gtf)
  must be quantitatively scored against gold-standard polyA sites to confirm cross-dataset
  generalizability.
when_to_use_negative:
- No gold-standard 3'-end polyA site calls exist for the dataset, so AP/F cannot be
  computed against known truth
- Evaluating upstream-exon coding APA events, which aptardi does not analyze and this
  benchmark does not cover
- Datasets from non-mammalian taxa, where the manually engineered DNA sequence features
  may not apply
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: aptardi
  role: Generates the predicted sample-specific transcriptomes whose polyA sites are
    benchmarked; run in Mode 1 with the pre-built model.hdf5 and scale.pk
  repo: github.com/luskry/aptardi
- name: RSEM
  role: Quantitates the aptardi transcriptome so the conservative 2/3-samples count
    filter can retain high-quality transcripts before scoring
- name: StringTie
  role: Produces the input stringtie.gtf transcriptome per dataset that aptardi refines;
    the comparison baseline whose AP aptardi roughly doubles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:fc2af3283ebb1485196cb893cee23ca482d76d7142958f8087b7d488b7c6272c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cutadapt
  - FastQC
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# precision-recall-benchmark-evaluation

## Summary

Benchmarks predicted transcript 3'/polyA sites against gold-standard 3'-end sequencing sites by computing average precision (AP) and F-measure per RNA-Seq dataset, as done for the aptardi HBR-trained model across 2nd HBR, UHR, BNLx, and SHR datasets.

## When to use

Use when a predicted sample-specific transcriptome (e.g., name_aptardi.gtf) must be quantitatively scored against gold-standard polyA sites to confirm cross-dataset generalizability.

## When NOT to use

- No gold-standard 3'-end polyA site calls exist for the dataset, so AP/F cannot be computed against known truth
- Evaluating upstream-exon coding APA events, which aptardi does not analyze and this benchmark does not cover
- Datasets from non-mammalian taxa, where the manually engineered DNA sequence features may not apply

## Inputs

- aptardi-modified transcriptomes (name_aptardi.gtf) for HBR, 2nd HBR, UHR, BNLx, and SHR datasets
- Gold-standard 3'-end sequencing polyA sites per dataset
- RSEM quantitation of aptardi transcriptomes for applying the 2/3-samples count filter

## Outputs

- Per-dataset average precision (AP) values
- Per-dataset F-measure values
- Combined benchmark table (aptardi_ap_f_benchmark.csv)

## How to apply

After generating aptardi-modified transcriptomes (name_aptardi.gtf) per dataset, quantitate each with RSEM and apply the conservative filter of removing transcripts without at least one count in 2/3 of samples to retain high-quality aptardi transcripts. Compare the predicted polyA sites against the gold-standard 3'-end sites for each dataset and compute average precision (AP) and F-measure. Target the reported benchmark values: AP = 0.65 (HBR), 0.63 (2nd HBR), 0.63 (UHR), 0.55 (BNLx), 0.55 (SHR); F = 0.56, 0.51, 0.53, 0.47, 0.48 respectively. Assemble per-dataset AP and F values into a single table (aptardi_ap_f_benchmark.csv). Matching the reported values within reasonable tolerance confirms the HBR-built model generalizes across library preparations, organisms, and sequencing depths, and — because training and testing AP estimates are similar — that aptardi is not prone to overfitting.

## Related tools

- **aptardi** (Generates the predicted sample-specific transcriptomes whose polyA sites are benchmarked; run in Mode 1 with the pre-built model.hdf5 and scale.pk) — github.com/luskry/aptardi
- **RSEM** (Quantitates the aptardi transcriptome so the conservative 2/3-samples count filter can retain high-quality transcripts before scoring)
- **StringTie** (Produces the input stringtie.gtf transcriptome per dataset that aptardi refines; the comparison baseline whose AP aptardi roughly doubles)

## Examples

```
aptardi --o out/ --f hg38.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk --g HBR_aptardi.gtf; RSEM-quantitate HBR_aptardi.gtf; compute AP/F vs gold-standard 3'-end sites
```

## Evaluation signals

- Reproduced AP for each dataset matches reported values (HBR 0.65, 2nd HBR 0.63, UHR 0.63, BNLx 0.55, SHR 0.55) within reasonable tolerance
- Reproduced F-measure matches reported values (HBR 0.56, 2nd HBR 0.51, UHR 0.53, BNLx 0.47, SHR 0.48) within reasonable tolerance
- External-dataset AP falls in the 0.55–0.63 band, consistent with consistent cross-sample generalizability relative to the 0.65 training-set AP
- Benchmark CSV (aptardi_ap_f_benchmark.csv) contains AP and F rows for all five datasets (HBR, 2nd HBR, UHR, BNLx, SHR)
- Similarity of AP between training and testing sets is preserved, evidencing no overfitting

## Limitations

- aptardi performed modestly worse on the BNLx and SHR datasets; possibly because true polyA sites were derived from Sprague Dawley rat rather than the specific strain, or owing to unstranded RNA-Seq and/or different species
- aptardi annotates 3' ends at 100-base-bin resolution (annotating the 3'-most base of the bin), while polyA sites can wiggle up to 30 nucleotides
- PolyA sites from deeper-sequenced data (e.g., PolyA-Seq) may not be detectable at shallower sequencing depths, biasing precision estimates

## Evidence

- [full_text] The HBR-trained aptardi prediction model achieved average precision between 0.55 and 0.63 on the four external data sets: [evidence span withheld — anchor status: not_found]
- [full_text] Compare predicted polyA sites against gold-standard 3'-end sites and compute AP and F-measure: [evidence span withheld — anchor status: not_found]
- [methods] Conservative filter of removing transcripts without at least one count in 2/3 of samples: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [methods] Quantitate aptardi transcriptome with RSEM: "1. Quantitate aptardi transcriptome with RSEM"
- [abstract] The average precision of the aptardi model is twice that of a standard transcriptome assembler: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [discussion] The similarity of AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting: "the similarity of the AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting"
