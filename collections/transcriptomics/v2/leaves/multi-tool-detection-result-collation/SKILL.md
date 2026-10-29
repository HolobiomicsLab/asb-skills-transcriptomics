---
name: multi-tool-detection-result-collation
description: Use when benchmarking fusion gene callers on the same input datasets
  under graded conditions (e.g., successive read subsampling) and you need a unified
  detection matrix to compare which tool calls which fusions at which depth.
when_to_use_negative:
- You have a single tool on a single full-depth dataset with no graded conditions
  — a simple call list suffices, no cross-tool matrix is needed
- Comparing fusions that do not include the anchored target gene — Anchored-fusion
  only reports fusions containing the user-specified gene, so unrestricted de novo
  comparison would be invalid
- Inputs are single-end RNA-seq or whole-genome/whole-exome data — Anchored-fusion
  requires paired-end RNA-seq and cannot contribute calls to the matrix in those settings
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Anchored-fusion
  role: Targeted fusion caller whose per-depth calls are collated; only detects fusions
    including the anchored gene of interest
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion
  role: bRNA-seq competing fusion detection method whose calls are entered into the
    detection matrix
  repo: https://github.com/STAR-Fusion
- name: FusionInspector
  role: bRNA-seq competing fusion detection method whose calls are entered into the
    detection matrix
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher
  role: bRNA-seq competing fusion detection method whose calls are entered into the
    detection matrix
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba
  role: bRNA-seq competing fusion detection method whose calls are entered into the
    detection matrix
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:14bcc1e047f34bd718c517ea570636cfdd5592e1283a9df28cf4aa5d5696cc59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - STAR-Fusion
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# multi-tool-detection-result-collation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collates fusion gene calls from multiple RNA-seq detection tools (e.g., Anchored-fusion, STAR-Fusion, FusionInspector, FusionCatcher, Arriba) across graded conditions such as sequencing-depth subsampling levels into a tool-by-condition detection matrix, enabling direct sensitivity comparison.

## When to use

Use when benchmarking fusion gene callers on the same input datasets under graded conditions (e.g., successive read subsampling) and you need a unified detection matrix to compare which tool calls which fusions at which depth.

## When NOT to use

- You have a single tool on a single full-depth dataset with no graded conditions — a simple call list suffices, no cross-tool matrix is needed
- Comparing fusions that do not include the anchored target gene — Anchored-fusion only reports fusions containing the user-specified gene, so unrestricted de novo comparison would be invalid
- Inputs are single-end RNA-seq or whole-genome/whole-exome data — Anchored-fusion requires paired-end RNA-seq and cannot contribute calls to the matrix in those settings

## Inputs

- Per-tool fusion call outputs (fusion gene lists/caller report files) for STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 runs
- Per-condition Anchored-fusion output files listing detected fusions containing the anchored target gene
- Condition labels for each run (e.g., 8 sequencing depths from 7 successive halvings of CCLE cell-line read pairs, ending below 1 M reads)
- Known/expected fusion gene list per sample (e.g., CCLE cell lines K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12) for scoring calls as true detections

## Outputs

- Tool-by-depth detection matrix recording which fusion genes each caller detects at each subsampling depth
- Sensitivity comparison summary (e.g., detection-at-depth curves or Figure-4-style pattern showing calls where competing tools fail)
- Collated list of fusion gene calls per tool per condition with true/false detection status against known fusions

## How to apply

After running each caller on every sample/condition — e.g., STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 on bRNA-seq data, plus Anchored-fusion via 'python Anchored_Fusion.py' with the anchored-gene fasta, genome fa, and GTF — collect the detected fusion genes from each tool's output. Build a tool-by-depth detection matrix where rows are callers and columns are the subsampling conditions (e.g., 8 depths generated by 7 successive random halvings of read pairs ending below 1 million reads), recording which fusion genes each caller detects at each depth. Because Anchored-fusion only reports fusions that include the user-anchored target gene, restrict or flag comparisons accordingly so that its targeted calls are scored against the same fusion events for the competing tools. Validate the collation against expected patterns, e.g., that Anchored-fusion detects the fusions at every low depth where all competing tools fail, as shown in the article's Figure 4.

## Related tools

- **Anchored-fusion** (Targeted fusion caller whose per-depth calls are collated; only detects fusions including the anchored gene of interest) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion** (bRNA-seq competing fusion detection method whose calls are entered into the detection matrix) — https://github.com/STAR-Fusion
- **FusionInspector** (bRNA-seq competing fusion detection method whose calls are entered into the detection matrix) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher** (bRNA-seq competing fusion detection method whose calls are entered into the detection matrix) — https://github.com/ndaniel/fusioncatcher
- **Arriba** (bRNA-seq competing fusion detection method whose calls are entered into the detection matrix) — https://github.com/suhrig/arriba

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=sample_R1.fastq.gz --fastq2=sample_R2.fastq.gz --out_folder=output_dir --file_ref_seq=genome.fa --file_ref_ann=annotation.gtf
```

## Evaluation signals

- Detection matrix dimensions match tools x conditions (e.g., 5 callers x 8 subsampling depths for the six CCLE cell lines), with every cell filled as detected/not-detected per fusion
- Fusion calls in each cell match the corresponding tool's raw output file for that sample/depth run
- Anchored-fusion cells are restricted to fusions containing the anchored target gene; no non-target fusions are scored for it
- Reproduction check: Anchored-fusion should show detections at all depths below 1 M reads for fusions where all four competing tools show no detections (Figure 4 pattern)
- True-detection labels agree with the known fusion genes for each cell line (e.g., BCR-ABL1 in K562, EML4-ALK in NCI-H3122)

## Limitations

- Anchored-fusion only reports fusions that include the user-provided target gene; fusions not involving the anchored gene will not appear in the collated results
- Anchored-fusion requires paired-end RNA-seq data and cannot be applied to single-end RNA-seq or whole-genome/whole-exome data
- If the input dataset is too small, the article recommends using the default HVLD model parameters from GitHub rather than retraining, which may affect call consistency across depths
- Using Anchored-fusion's pre-trained filter instead of retraining ('--not_train_filter_model') may cause result deviations since it was not trained on the input data

## Evidence

- [other] Collate all detected fusion genes into a tool-by-depth detection matrix, recording for each caller which fusions are called at each subsampling depth.: [evidence span withheld — anchor status: not_found]
- [other] verify the reported Figure 4 pattern that Anchored-fusion detects fusion genes at every low sequencing depth where all four competing tools fail to detect them: [evidence span withheld — anchor status: not_found]
