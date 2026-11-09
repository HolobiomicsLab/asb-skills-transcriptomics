---
name: mgatk-vmr-strand-correlation-variant-calling
description: Use when benchmarking mtDNA variant callers on single-cell data, or when
  comparing an mgatk-based clone-assignment workflow against MQuad-based selection
  using concordance to known source labels.
when_to_use_negative:
- When the goal is to identify informative mtDNA variants for clonal substructure
  in conventional scRNA-seq — the article states mgatk was 'designed primarily for
  single-cell assay for transposase-accessible chromatin (scATAC-seq)', so use MQuad
  instead
- When input is raw FASTQ — alignment (STAR) and duplicate removal (Picard MarkDuplicates)
  must be done upstream first
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mgatk
  role: mtDNA variant caller whose suggested cutoffs (log10(VMR) ≥ −2, strand correlation
    ≥ 0.65) define variant selection in this comparison arm
- name: vireoSNP
  role: Variational-inference clonal assignment using the mgatk-called mtSNVs
  repo: https://github.com/single-cell-genetics/vireo
- name: STAR
  role: Aligns raw FASTQ to UCSC hg19 before variant calling
- name: Picard
  role: MarkDuplicates removes PCR duplicates from raw BAM files before variant detection
- name: MQuad
  role: 'Comparator arm of the benchmark: MQuad-selected variants (~146, 93% concordance)
    vs mgatk (312, 64%)'
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:5714afda21a091e225b8ba44445952c535e419452933911d981626711b74a648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - mgatk
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mgatk-vmr-strand-correlation-variant-calling

## Summary

Call mtDNA variants from single-cell data with mgatk using its suggested quality cutoffs — log10(VMR) ≥ −2 and strand correlation coefficient ≥ 0.65 — and feed the resulting mtSNVs to vireoSNP for clonal assignment. In this article it serves as the benchmark comparator against the cellSNP-MQuad-VireoSNP pipeline on the Kim ccRCC Smart-seq2 scRNA-seq dataset (mgatk: 312 mtSNVs, 64% concordance vs MQuad: ~146 variants, 93%).

## When to use

Use when benchmarking mtDNA variant callers on single-cell data, or when comparing an mgatk-based clone-assignment workflow against MQuad-based selection using concordance to known source labels.

## When NOT to use

- When the goal is to identify informative mtDNA variants for clonal substructure in conventional scRNA-seq — the article states mgatk was 'designed primarily for single-cell assay for transposase-accessible chromatin (scATAC-seq)', so use MQuad instead
- When input is raw FASTQ — alignment (STAR) and duplicate removal (Picard MarkDuplicates) must be done upstream first

## Inputs

- Analysis-ready BAM files of single-cell mtDNA reads (e.g., Smart-seq2 scRNA-seq aligned with STAR to UCSC hg19, PCR duplicates removed with Picard MarkDuplicates)
- Cell source/clone labels for validation (e.g., PDX pRCC, PDX mRCC, Pt mRCC)

## Outputs

- mgatk-called mtSNV set with per-variant VMR and strand correlation statistics
- VireoSNP clone assignments per cell (cells × clones)
- Confusion matrix and assignment concordance vs source labels (reference: 312 mtSNVs, 64% concordance on Kim ccRCC)

## How to apply

Run mgatk on the same analysis-ready BAM files used for the MQuad arm (for Smart-seq2 scRNA-seq: STAR alignment to UCSC hg19 with Picard MarkDuplicates duplicate removal). Apply mgatk's suggested variant-calling cutoffs: log10(VMR) ≥ −2 and strand correlation coefficient ≥ 0.65. Pile up and select mtSNVs, then pass the mgatk variant set to vireoSNP (variational inference) to assign cells to mitochondrial clones. Build a confusion matrix of predicted clones versus known source labels (e.g., PDX pRCC, PDX mRCC, Pt mRCC) and compute assignment concordance to quantify how many informative variants the caller recovered and how faithfully they reproduce lineage structure. Note the article reports mgatk yielded 312 mtSNVs but only 64% concordance on the ccRCC dataset versus MQuad's ~146 variants at 93%, so more variants does not mean better clonal signal — this skill is chiefly a sensitivity/specificity comparator.

## Related tools

- **mgatk** (mtDNA variant caller whose suggested cutoffs (log10(VMR) ≥ −2, strand correlation ≥ 0.65) define variant selection in this comparison arm)
- **vireoSNP** (Variational-inference clonal assignment using the mgatk-called mtSNVs) — https://github.com/single-cell-genetics/vireo
- **STAR** (Aligns raw FASTQ to UCSC hg19 before variant calling)
- **Picard** (MarkDuplicates removes PCR duplicates from raw BAM files before variant detection)
- **MQuad** (Comparator arm of the benchmark: MQuad-selected variants (~146, 93% concordance) vs mgatk (312, 64%)) — https://github.com/single-cell-genetics/MQuad

## Evaluation signals

- Confirm mgatk variant calls pass the suggested cutoffs: log10(VMR) ≥ −2 and strand correlation coefficient ≥ 0.65
- Expect on the Kim ccRCC Smart-seq2 dataset roughly 312 mtSNVs — a large surplus over MQuad's ~146 informative variants
- Confusion matrix of vireoSNP clones vs source labels should yield ≈64% concordance for the mgatk arm (vs 93% for MQuad), i.e., fewer variants per clone signal do not improve assignment
- Verify the comparison summary table lists variant counts and concordance for both mgatk and MQuad arms side by side

## Limitations

- mgatk was designed primarily for scATAC-seq; the article finds its performance on Smart-seq2 scRNA-seq is inferior to MQuad (64% vs 93% concordance) despite calling more variants (312 vs ~146)
- Low-efficiency assays such as Smart-seq2 suffer high dropout, making SNV-based lineage reconstruction challenging for callers that rely on raw allele statistics rather than mixture-model ΔBIC selection
- Frequent false positives in error-prone rCRS regions (302–315, 513–525, 3105–3109) confound mtSNV-based inference if not explicitly removed

## Evidence

- [discussion] mgatk suggested cutoffs for variant calling: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [intro] mgatk designed for scATAC-seq: "MtDNA specific methods, EMBLEM22 and mgatk16 were designed primarily for single-cell assay for transposase-accessible chromatin"
- [discussion] duplicates removed before variant detection: "duplicates were marked and removed from the raw BAM files using MarkDuplicates from Picard (v2.18.9)"
- [discussion] MQuad variant selection is model-based, more robust than raw statistics: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [discussion] clonal assignment step with vireoSNP: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
