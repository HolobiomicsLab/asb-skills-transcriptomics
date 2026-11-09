---
name: mtdna-clone-based-cell-subsetting
description: Use when a scRNA-seq dataset has mtDNA-derived clone labels from MQuad/vireoSNP
  (e.g., TNBC1 clones MT0 vs MT1) and you need to split cells by clone for clone-level
  differential expression or comparison.
when_to_use_negative:
- Cells have no mtDNA clone assignments yet — run the cellSNP-MQuad-vireoSNP pipeline
  first to generate variant calls and clonal labels.
- The dataset lacks heteroplasmic mtDNA variants (e.g., low-coverage scATAC data with
  NUMT contamination or clones indistinguishable by mtDNA mutations).
- The question is donor deconvolution of multiplexed scRNA-seq (vireo's primary use
  case with nuclear SNPs), not within-sample subclone resolution.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MQuad
  role: Selects clonally informative mtDNA variants with a binomial mixture model,
    providing the SNPs that define clone membership
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Assigns individual cells to mtDNA-defined clones, producing the clone labels
    used for cell subsetting
  repo: https://github.com/single-cell-genetics/vireo
- name: cellsnp-lite
  role: Piles up mtDNA reads from BAM files into SNP-by-cell AD/DP matrices that feed
    MQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Seurat
  role: Provides FindMarkers (DESeq2 model) for differential expression between the
    clone-subset cell groups
- name: DESeq2
  role: Statistical model (two-sided Wald test) underlying FindMarkers for the clone-vs-clone
    DE test
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:91b9d67612557167aca1cf8047e2a9254c2e5831b5995e885c8c2756461b2378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - DESeq2
  - MQuad
  - Seurat
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mtdna-clone-based-cell-subsetting

## Summary

Subsets single-cell tumor data into mtDNA-defined clone groups (e.g., TNBC1 T0: Tumor & MT0 vs T1: Tumor & MT1) using clone assignments from the MQuad/cellSNP/vireoSNP pipeline, enabling clone-level downstream analyses such as differential expression with Seurat FindMarkers (DESeq2).

## When to use

Use when a scRNA-seq dataset has mtDNA-derived clone labels from MQuad/vireoSNP (e.g., TNBC1 clones MT0 vs MT1) and you need to split cells by clone for clone-level differential expression or comparison.

## When NOT to use

- Cells have no mtDNA clone assignments yet — run the cellSNP-MQuad-vireoSNP pipeline first to generate variant calls and clonal labels.
- The dataset lacks heteroplasmic mtDNA variants (e.g., low-coverage scATAC data with NUMT contamination or clones indistinguishable by mtDNA mutations).
- The question is donor deconvolution of multiplexed scRNA-seq (vireo's primary use case with nuclear SNPs), not within-sample subclone resolution.

## Inputs

- 10x scRNA-seq BAM/FASTQ-aligned expression data with cell barcodes (e.g., TNBC1 dataset)
- mtDNA clone assignment table per cell from MQuad/vireoSNP (clone labels such as 'Tumor & MT0', 'Tumor & MT1')
- Seurat object with tumor-cell annotations

## Outputs

- tnbc1_de_genes.csv containing gene, log2 fold change, p-value and adjusted p-value for all tested genes
- tnbc1_de_volcano.png volcano plot (negative log10 P-value vs average log2 FC, significant genes in red)
- Clone-subset cell groups (T0: Tumor & MT0 vs T1: Tumor & MT1)

## How to apply

First obtain clone assignments from the cellSNP-MQuad-vireoSNP pipeline: cellsnp-lite (v1.2.1) piles up mtDNA reads from the BAM into a SNP-by-cell AD/DP matrix, MQuad (v0.1.6) selects informative variants with a binomial mixture model using a ΔBIC cutoff from the Kneedle algorithm, and vireoSNP (v0.5.3) assigns cells to clones. Using these assignments, subset the tumor cells into the mtDNA-defined groups — for the TNBC1 10x dataset this is T0 (Tumor & MT0) and T1 (Tumor & MT1). Run differential expression between the subsets with Seurat FindMarkers using the DESeq2 model (two-sided Wald test), writing the full results (gene, log2 fold change, p-value, adjusted p-value) to a CSV such as tnbc1_de_genes.csv. Select significant genes at adjusted p-value < 0.01 and generate a volcano plot of negative log10 P-values against average log2 fold changes with significant genes highlighted in red (tnbc1_de_volcano.png). The rationale is that mtDNA variants provide endogenous lineage markers that resolve subclones finer than CNV-based approaches.

## Related tools

- **MQuad** (Selects clonally informative mtDNA variants with a binomial mixture model, providing the SNPs that define clone membership) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Assigns individual cells to mtDNA-defined clones, producing the clone labels used for cell subsetting) — https://github.com/single-cell-genetics/vireo
- **cellsnp-lite** (Piles up mtDNA reads from BAM files into SNP-by-cell AD/DP matrices that feed MQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **Seurat** (Provides FindMarkers (DESeq2 model) for differential expression between the clone-subset cell groups)
- **DESeq2** (Statistical model (two-sided Wald test) underlying FindMarkers for the clone-vs-clone DE test)

## Examples

```
tnbc1_markers <- FindMarkers(tnbc1_seurat, ident.1 = "Tumor & MT1", ident.2 = "Tumor & MT0", test.use = "DESeq2"); write.csv(tnbc1_markers, "tnbc1_de_genes.csv")
```

## Evaluation signals

- Significant DE gene set at adjusted p < 0.01 contains approximately 57 up-regulated genes in subclone 1 (T1) between TNBC1 clones
- Known marker genes SEC61G, RPL39 and SETD5 are present among the significant up-regulated genes
- tnbc1_de_genes.csv has the expected schema: gene, log2 fold change, p-value, adjusted p-value
- Volcano plot file exists with both axes (negative log10 P-value, average log2 FC) and significant genes highlighted in red
- Cell subsets partition tumor cells without overlap into T0 (Tumor & MT0) and T1 (Tumor & MT1) per the clone assignments

## Limitations

- MQuad explicitly removes frequent false positive variants in error-prone rCRS regions 302–315, 513–525, 3105–3109; unmasked NUMT/sequencing-error regions can confound clone labels.
- SNV-based lineage reconstruction is hampered by few somatic nuclear variants and high dropout, especially in low-efficiency assays such as Smart-seq2; mtDNA variants in conventional 10x data are the more reliable route.
- Subclonal structure from coverage-inferred CNVs may be less dynamic than subclones shown by propagation of true point mutations, so CNV-based grouping may disagree with mtDNA-based clones.

## Evidence

- [discussion] vireoSNP reconstructs clones from MQuad SNPs: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [readme] README: vireoSNP assigns cells to mtDNA clones: "3. use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [results] TNBC1 clone DE finding: 57 genes at adj p<0.01: [evidence span withheld — anchor status: not_found]
- [abstract] MQuad calls clonally informative mtDNA variants: "Here we present an open source computational tool MQuad that accurately calls clonally informative mtDNA variants in a population of single cells"
