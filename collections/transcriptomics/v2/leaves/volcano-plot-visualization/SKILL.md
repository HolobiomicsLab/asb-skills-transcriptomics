---
name: volcano-plot-visualization
description: Use when you have a two-group differential expression table (gene, log2FC,
  p-value, adjusted p-value) — e.g. Seurat FindMarkers output between mtDNA-defined
  subclones — and need to visualize which genes are both significant and large-effect.
when_to_use_negative:
- 'There is no two-group comparison: the input is a single-cell clustering or clonal-reconstruction
  result (e.g. MQuad/vireoSNP clone assignments) rather than a DE results table.'
- The goal is clone discovery from mtDNA variants — use the cellSNP-MQuad-vireoSNP
  pipeline, not volcano plotting, which only visualizes already-computed DE statistics.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Runs the differential expression analysis (FindMarkers, DESeq2 model) whose
    results are visualized in the volcano plot
- name: DESeq2
  role: Statistical model used by FindMarkers (two-sided Wald test) producing log2FC
    and adjusted p-values plotted in the volcano
- name: MQuad
  role: Identifies the informative mtDNA variants that define the T0/T1 subclones
    being contrasted
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:2e0378fb7237feaccbf156b430b9684694c0c766244c894e5c3e800fc8220e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - DESeq2
  - MQuad
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# volcano-plot-visualization

## Summary

Generates a volcano plot of differential expression results, plotting negative log10 p-values against average log2 fold changes, so that significance and effect size are visible simultaneously. This skill was used to visualize 57 genes up-regulated in the TNBC1 mtDNA subclone T1 (Fig. 5e of the MQuad paper).

## When to use

Use when you have a two-group differential expression table (gene, log2FC, p-value, adjusted p-value) — e.g. Seurat FindMarkers output between mtDNA-defined subclones — and need to visualize which genes are both significant and large-effect.

## When NOT to use

- There is no two-group comparison: the input is a single-cell clustering or clonal-reconstruction result (e.g. MQuad/vireoSNP clone assignments) rather than a DE results table.
- The goal is clone discovery from mtDNA variants — use the cellSNP-MQuad-vireoSNP pipeline, not volcano plotting, which only visualizes already-computed DE statistics.

## Inputs

- Differential expression table from Seurat FindMarkers (DESeq2 model) with columns: gene, log2 fold change, p-value, adjusted p-value (tnbc1_de_genes.csv)
- Clone/subclone group assignments (T0: Tumor & MT0 vs T1: Tumor & MT1) from MQuad/vireoSNP-based mtDNA clonal reconstruction
- Adjusted p-value cutoff (0.01) used to define the significant gene set

## Outputs

- tnbc1_de_volcano.png — volcano plot of negative log10 p-values vs average log2 fold changes, significant genes highlighted in red
- Significant DE gene list (adjusted p < 0.01), expected to contain ~57 up-regulated genes including SEC61G, RPL39 and SETD5

## How to apply

First run differential expression between the two groups — here the TNBC1 tumor cells split by mtDNA clone into T0 (Tumor & MT0) and T1 (Tumor & MT1) — using Seurat FindMarkers with the DESeq2 model (two-sided Wald test), writing the full results (gene, log2 fold change, p-value, adjusted p-value) to tnbc1_de_genes.csv. Select significant genes at adjusted p-value < 0.01 and confirm the expected set (~57 up-regulated genes in subclone 1, including SEC61G, RPL39 and SETD5). Then plot negative log10 of the p-value (y-axis) against average log2 fold change (x-axis), highlighting significant genes in red, and save as tnbc1_de_volcano.png. Rationale: the volcano plot simultaneously encodes statistical significance (adjusted p<0.01) and biological effect size (log2FC), matching the presentation of clone-specific transcriptional differences in Fig. 5e.

## Related tools

- **Seurat** (Runs the differential expression analysis (FindMarkers, DESeq2 model) whose results are visualized in the volcano plot)
- **DESeq2** (Statistical model used by FindMarkers (two-sided Wald test) producing log2FC and adjusted p-values plotted in the volcano)
- **MQuad** (Identifies the informative mtDNA variants that define the T0/T1 subclones being contrasted) — https://github.com/single-cell-genetics/MQuad

## Examples

```
seurat_obj <- subset(tnbc1, subset = clone %in% c('T0','T1')); de <- FindMarkers(seurat_obj, ident.1='T1', ident.2='T0', test.use='DESeq2'); write.csv(de,'tnbc1_de_genes.csv')
```

## Evaluation signals

- The DE table written to tnbc1_de_genes.csv contains gene, log2 fold change, p-value and adjusted p-value columns for all tested genes.
- The significant set at adjusted p-value < 0.01 contains approximately 57 up-regulated genes in subclone 1, including SEC61G, RPL39 and SETD5.
- tnbc1_de_volcano.png exists and its axes are negative log10 p-value (y) and average log2 fold change (x).
- Significant genes are visually distinguished (highlighted in red) from non-significant points in the plot.
- The plot is consistent with the published figure (Fig. 5e) in which 57 genes appear up-regulated in subclone 1.

## Limitations

- The interpretation depends on the upstream clone assignment quality: noisy or uninformative mtDNA SNVs can confound clonal inference, so DE groups must come from reliable informative variants (MQuad ΔBIC model selection).
- Only transcriptional differences at adjusted p < 0.01 are highlighted; genes with modest fold change or sub-threshold significance are not represented as significant in the volcano.
- The reported ~57 up-regulated gene count is dataset-specific (TNBC1 10x scRNA-seq); other datasets will not reproduce this exact number or gene set.

## Evidence

- [results] Between the two mtDNA clones in TNBC1, 57 genes were up-regulated in subclone 1 at adjusted p-value < 0.01, detected with Seurat FindMarkers (DESeq2 model), as shown in the volcano plot (Fig. 5e).: [evidence span withheld — anchor status: not_found]
- [methods] Generate a volcano plot of negative log10 P-values against average log2 fold changes with significant genes highlighted in red: [evidence span withheld — anchor status: not_found]
- [methods] Run differential expression between T0 and T1 with the FindMarkers function from Seurat using the DESeq2 model (two-sided Wald test): [evidence span withheld — anchor status: not_found]
- [readme] MQuad README on pipeline position: use cellSNP or cellsnp-lite to pileup mtDNA variants, MQuad to select informative variants, vireoSNP to assign cells to clones: "use vireoSNP <https://github.com/single-cell-genetics/vireo>_ to assign cells to clones based on mtDNA variant profile"
