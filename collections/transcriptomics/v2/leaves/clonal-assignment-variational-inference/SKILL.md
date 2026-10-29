---
name: clonal-assignment-variational-inference
description: Use when a SNP-by-cell matrix of mitochondrial alternate-allele depths
  (AD) and total depths (DP) is available with MQuad-selected informative mtDNA variants,
  and cells must be partitioned into clonal populations.
when_to_use_negative:
- Input variants are nuclear SNVs in a diploid context — the article notes nuclear
  SNV callers like Monovar and Conbase assume diploidy, which is violated in the mitochondrial
  genome; conversely, mtDNA variants need mtDNA-aware assignment, not generic diploid
  genotyping
- No MQuad-selected informative variants are available (e.g., raw unfiltered mtDNA
  SNPs including noisy, uninformative variants and error-prone rCRS-region false positives),
  which the article says confounds clonal inference
- Data are from mtscATAC-seq where dedicated tools like mgatk or EMBLEM were the primary
  design target and may be more appropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0654
tools:
- name: vireoSNP
  role: Reconstructs clonal populations and assigns cells to mitochondrial clones
    by variational inference on MQuad-selected mtDNA variants
  repo: https://github.com/single-cell-genetics/vireo
- name: cellsnp-lite
  role: Piles up mtDNA reads from BAM files to generate the SNP-by-cell AD/DP count
    matrices that vireoSNP consumes
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Selects high-quality informative mtDNA variants with a binomial mixture model
    before clonal assignment
  repo: https://github.com/single-cell-genetics/MQuad
- name: Cardelino
  role: Provides nuclear SNV clone labels used to cross-tabulate and validate mtDNA
    clone assignments (joxm fibroblast analysis)
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:92d52f8251ebc2422402b4f461396207de0f8ab2d72f3269fe9134c816eb60fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - Cardelino
  - CellRanger
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# clonal-assignment-variational-inference

## Summary

Assigns single cells to mitochondrial clones by running vireoSNP variational inference on clone-discriminative mtDNA variants selected by MQuad from cellsnp-lite SNP-by-cell AD/DP matrices, completing the cellSNP-MQuad-VireoSNP clonality discovery pipeline.

## When to use

Use when a SNP-by-cell matrix of mitochondrial alternate-allele depths (AD) and total depths (DP) is available with MQuad-selected informative mtDNA variants, and cells must be partitioned into clonal populations.

## When NOT to use

- Input variants are nuclear SNVs in a diploid context — the article notes nuclear SNV callers like Monovar and Conbase assume diploidy, which is violated in the mitochondrial genome; conversely, mtDNA variants need mtDNA-aware assignment, not generic diploid genotyping
- No MQuad-selected informative variants are available (e.g., raw unfiltered mtDNA SNPs including noisy, uninformative variants and error-prone rCRS-region false positives), which the article says confounds clonal inference
- Data are from mtscATAC-seq where dedicated tools like mgatk or EMBLEM were the primary design target and may be more appropriate

## Inputs

- SNP-by-cell matrices of alternate-allele depths (AD) and total depths (DP) from cellsnp-lite pileup of mtDNA reads
- List of informative mtDNA SNPs selected by MQuad's ΔBIC (Kneedle knee-point) cutoff
- Optionally, cell barcode list (-b) and independent clone labels (CNV clones, Cardelino nuclear clones, or source labels) for cross-validation

## Outputs

- Per-cell mitochondrial clone assignments with assignment probabilities
- Clone labels for confident cells (e.g., probability >0.8)
- Mean allelic frequency table of each mtDNA SNV per clone
- Confusion matrix between mitochondrial clones and independent clone/source labels

## How to apply

Take the SNP-by-cell AD/DP matrices produced by cellsnp-lite (v1.2.1) from pileup of mtDNA reads, restricted to the informative SNPs MQuad (v0.1.6) selected via the ΔBIC = BIC(M0) − BIC(M1) Kneedle knee-point cutoff (excluding error-prone rCRS regions 302–315, 513–525, 3105–3109). Run vireoSNP (v0.5.3) variational inference on these variants to reconstruct clonal populations and assign each cell to a mitochondrial clone, optionally keeping only confident assignments (e.g., probability >0.8, as in the TNBC1 analysis where 1097 of 1146 cells, 95.8%, passed this threshold). Tabulate mean allelic frequency of each detected mtDNA SNV per clone to verify clone-discriminative variants (e.g., 2393C>T and 8368G>A marking clone MT0 in MKN-45), and cross-tabulate mtDNA clones against independent labels (CNV clones from CellRanger/BAF clustering, nuclear SNV clones from Cardelino, or tumor/source labels) as external validation. The rationale is that mtDNA mutations detected by MQuad complement nuclear SNVs and CNVs to achieve finer clonal resolution.

## Related tools

- **vireoSNP** (Reconstructs clonal populations and assigns cells to mitochondrial clones by variational inference on MQuad-selected mtDNA variants) — https://github.com/single-cell-genetics/vireo
- **cellsnp-lite** (Piles up mtDNA reads from BAM files to generate the SNP-by-cell AD/DP count matrices that vireoSNP consumes) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Selects high-quality informative mtDNA variants with a binomial mixture model before clonal assignment) — https://github.com/single-cell-genetics/MQuad
- **Cardelino** (Provides nuclear SNV clone labels used to cross-tabulate and validate mtDNA clone assignments (joxm fibroblast analysis))

## Examples

```
vireo -c cellsnp_out/cellSNP.samples.vcf.gz -N 5 -o vireo_out  # assign cells to mitochondrial clones from cellsnp-lite VCF with MQuad-selected SNPs
```

## Evaluation signals

- Number of inferred mitochondrial clones and assigned cells matches known biology (e.g., MKN-45: 5 clones over 5220 cells; joxm fibroblast: ~24 SNVs assigning 77 cells into 3 clones)
- Confident-assignment rate at probability >0.8 matches expectation (TNBC1: 1097/1146 cells, 95.8%)
- Allelic-frequency table shows clone-discriminative variants restricted to their clone (MT0 carries 2393C>T and 8368G>A, largely absent in all other clones)
- Confusion matrix shows expected cross-modality overlap (MKN-45: CNV clone 1 highly overlaps mitochondrial clone 0; joxm: nuclear clone 3 identical to mitochondrial clone 2; TNBC1: mtDNA clone 1 overlaps almost exclusively with CNV subclone B, Fisher's exact p ≈ 6e-5)
- Assignment concordance with source labels falls in reported range (ccRCC: 93% concordance with ~146 MQuad variants, vs 64% for mgatk's 312 mtSNVs)

## Limitations

- Assignment quality depends on the upstream MQuad ΔBIC (Kneedle knee-point) variant selection; uninformative noisy mtDNA SNVs and sequencing errors confound clonal inference
- Low-coverage datasets yield few informative variants (e.g., TNBC1 had only two informative variants), limiting clone resolution
- CNV-inferred subclonal structure may be less dynamic than subclones shown by propagation of true point mutations, so cross-modality clone labels need not coincide
- In Smart-seq2-like deep data, the model-selection approach is needed to reduce false positives compared to raw allele-frequency filtering

## Evidence

- [discussion] vireoSNP uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [intro] Integrated pipeline from mtDNA genotyping to clonal reconstruction: "an integrated clonality discovery pipeline, cellSNP-MQuad-VireoSNP, which provides a complete analysis suite from single cell mtDNA genotyping to clonal reconstruction."
- [discussion] cellsnp-lite pileup generates SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [readme] README describes recommended three-step pipeline ending in vireoSNP clone assignment: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [intro] mtDNA mutations complement nuclear SNVs and CNVs for finer resolution: "our analysis reveals that mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution."
