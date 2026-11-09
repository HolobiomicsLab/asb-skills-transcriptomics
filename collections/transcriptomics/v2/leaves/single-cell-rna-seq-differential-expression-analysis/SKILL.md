---
name: single-cell-rna-seq-differential-expression-analysis
description: Use when two or more single-cell groups (e.g., cells assigned to mtDNA
  variants/clones by MQuad+vireoSNP) need to be compared transcriptomically to report
  up-/down-regulated genes with adjusted p-value cutoffs.
when_to_use_negative:
- The goal is discovering which mtDNA variants discriminate clones, not testing transcriptional
  consequences — use MQuad on cellsnp-lite AD/DP matrices instead of differential
  expression on the expression matrix.
- The comparison groups are defined by a single confounded variable (e.g., clone assignment
  perfectly co-segregates with batch or donor), so DE calls would not be attributable
  to the clone.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: FindMarkers function performs the between-clone differential expression test
- name: DESeq2
  role: Statistical model (two-sided Wald test) used as the test.use backend of FindMarkers
- name: MQuad
  role: 'Upstream: selects clonally informative mtDNA variants that define the T0/T1
    comparison groups'
  repo: github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: 'Upstream: assigns cells to mtDNA-defined clones used as the DE grouping'
  repo: github.com/single-cell-genetics/vireo
- name: cellsnp-lite
  role: 'Upstream: piles up mtDNA reads from BAM files to produce SNP-by-cell AD/DP
    matrices for MQuad'
  repo: github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:46f8ef818b38851b781b212319ec73b6766e6bac88e4055d8c4f4677cda2b29a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MQuad
  - Seurat
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-rna-seq-differential-expression-analysis

## Summary

Detects transcriptional differences between single-cell-defined subclones (here, mtDNA-defined clones T0 vs T1 in the TNBC1 10x scRNA-seq dataset) by running Seurat FindMarkers with the DESeq2 two-sided Wald test and summarizing results as a table of log2 fold changes and adjusted p-values plus a volcano plot.

## When to use

Use when two or more single-cell groups (e.g., cells assigned to mtDNA variants/clones by MQuad+vireoSNP) need to be compared transcriptomically to report up-/down-regulated genes with adjusted p-value cutoffs.

## When NOT to use

- The goal is discovering which mtDNA variants discriminate clones, not testing transcriptional consequences — use MQuad on cellsnp-lite AD/DP matrices instead of differential expression on the expression matrix.
- The comparison groups are defined by a single confounded variable (e.g., clone assignment perfectly co-segregates with batch or donor), so DE calls would not be attributable to the clone.

## Inputs

- Seurat scRNA-seq expression object with 10x TNBC1 data and per-cell metadata (cell class and mtDNA clone assignment: T0 = Tumor & MT0, T1 = Tumor & MT1)
- Clone-assignment table from MQuad/vireoSNP (cell barcode → clone)

## Outputs

- tnbc1_de_genes.csv with columns gene, log2 fold change, p-value, adjusted p-value
- tnbc1_de_volcano.png volcano plot (x: average log2 FC; y: negative log10 p-value; significant genes in red)
- Significant-gene set at adjusted p < 0.01 (reported: 57 genes up-regulated in subclone 1, incl. SEC61G, RPL39, SETD5)

## How to apply

Subset the Seurat object to the cell classes under comparison — for TNBC1, tumor cells split into T0 (Tumor & MT0) and T1 (Tumor & MT1) using clone assignments from the MQuad/vireoSNP pipeline (cellsnp-lite pileup → MQuad binomial mixture model → vireoSNP variational-inference clone assignment). Run FindMarkers with test.use='DESeq2' (two-sided Wald test) comparing T1 against T0, and export gene, log2 fold change, p-value, and adjusted p-value columns. Threshold significance at adjusted p-value < 0.01 and count genes up-regulated in subclone 1; in this dataset this yielded 57 up-regulated genes including SEC61G, RPL39, and SETD5. Render a volcano plot of negative log10 p-values against average log2 fold change, highlighting significant genes in red.

## Related tools

- **Seurat** (FindMarkers function performs the between-clone differential expression test)
- **DESeq2** (Statistical model (two-sided Wald test) used as the test.use backend of FindMarkers)
- **MQuad** (Upstream: selects clonally informative mtDNA variants that define the T0/T1 comparison groups) — github.com/single-cell-genetics/MQuad
- **vireoSNP** (Upstream: assigns cells to mtDNA-defined clones used as the DE grouping) — github.com/single-cell-genetics/vireo
- **cellsnp-lite** (Upstream: piles up mtDNA reads from BAM files to produce SNP-by-cell AD/DP matrices for MQuad) — github.com/single-cell-genetics/cellsnp-lite

## Examples

```
markers <- FindMarkers(tnbc1_seurat, ident.1 = "T1", ident.2 = "T0", test.use = "DESeq2", logfc.threshold = 0); write.csv(markers, "tnbc1_de_genes.csv")
```

## Evaluation signals

- The DE table at adjusted p < 0.01 contains approximately 57 genes up-regulated in subclone 1, matching the published volcano plot (Fig. 5e).
- The gene set includes SEC61G, RPL39, and SETD5 among the significant up-regulated genes.
- tnbc1_de_genes.csv exists with the four required columns (gene, log2FC, p-value, adjusted p-value).
- tnbc1_de_volcano.png exists and plots negative log10 p-value vs average log2 fold change with significant genes highlighted in red.

## Limitations

- DE results depend critically on the upstream clone assignments; noisy or false-positive mtDNA variant calls propagate into the DE grouping.
- cellsnp-lite README notes it is 'a faster version of cellSNP, still at testing stage so might be unstable', so upstream genotyping quality can affect clone definition feeding the DE step.
- The reported result (57 genes at adj p < 0.01) is specific to the TNBC1 dataset and cutoff; other datasets need their own significance thresholds and validation.

## Evidence

- [full_text] Run differential expression between T0 and T1 with the FindMarkers function from Seurat using the DESeq2 model (two-sided Wald test): [evidence span withheld — anchor status: not_found]
- [full_text] 57 genes were up-regulated in subclone 1 at adjusted p-value < 0.01: [evidence span withheld — anchor status: not_found]
- [readme] use vireoSNP to assign cells to clones based on mtDNA variant profile: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [discussion] MQuad takes this output and selects high quality informative variants with a binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
