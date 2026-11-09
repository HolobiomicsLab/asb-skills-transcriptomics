---
name: breast-cancer-de-gene-interpretation
description: Use when single-cell RNA-seq data has cells assigned to mtDNA-defined
  clones (e.g., TNBC1 T0 vs T1 subclones) and you need to test whether the subclones
  show detectable transcriptional differences via differential expression.
when_to_use_negative:
- Input is bulk RNA-seq or data without clone/subclone assignments — the contrast
  requires pre-defined mtDNA clone groups (T0 vs T1)
- The goal is calling clonally informative mtDNA variants themselves (use the cellSNP–MQuad–vireoSNP
  pipeline instead of nuclear-gene DE)
- Cells are not tumor cells or the mitochondrial subclones have too few cells per
  group to support a DESeq2-based two-sided Wald test
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MQuad
  role: 'Upstream: detects clonally informative mtDNA variants whose clones (MT0/MT1)
    define the DE comparison groups'
  repo: github.com/single-cell-genetics/MQuad
- name: Seurat
  role: Runs FindMarkers differential expression between the two mtDNA subclones
- name: DESeq2
  role: Statistical model used by FindMarkers (two-sided Wald test) for the DE calls
- name: vireoSNP
  role: 'Downstream of MQuad: assigns cells to clones based on the mtDNA variant profile,
    providing the T0/T1 group labels'
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:7ec6811bee8dcc846e48df66a400bf16d4264f3fae647787a38aab7dc7354a74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MQuad
  - Seurat
  - vireoSNP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# breast-cancer-de-gene-interpretation

## Summary

Detects transcriptional differences between mtDNA-defined tumor subclones in the TNBC1 10x scRNA-seq dataset by running Seurat FindMarkers (DESeq2 model) between the T0 (Tumor & MT0) and T1 (Tumor & MT1) mitochondrial subclones, then interpreting the significant gene set (57 up-regulated genes including SEC61G, RPL39, SETD5) with a volcano plot.

## When to use

Use when single-cell RNA-seq data has cells assigned to mtDNA-defined clones (e.g., TNBC1 T0 vs T1 subclones) and you need to test whether the subclones show detectable transcriptional differences via differential expression.

## When NOT to use

- Input is bulk RNA-seq or data without clone/subclone assignments — the contrast requires pre-defined mtDNA clone groups (T0 vs T1)
- The goal is calling clonally informative mtDNA variants themselves (use the cellSNP–MQuad–vireoSNP pipeline instead of nuclear-gene DE)
- Cells are not tumor cells or the mitochondrial subclones have too few cells per group to support a DESeq2-based two-sided Wald test

## Inputs

- TNBC1 10x scRNA-seq expression matrix (Seurat object) with tumor-cell annotations
- Clone assignments for TNBC1 cells into T0 (Tumor & MT0) and T1 (Tumor & MT1) from mtDNA variant analysis
- Comparison labels specifying subclone 1 (T1) as the group expected to harbor up-regulated genes

## Outputs

- tnbc1_de_genes.csv: full DE table with gene, log2 fold change, p-value, adjusted p-value
- tnbc1_de_volcano.png: volcano plot of negative log10 P-values vs average log2 fold change with significant genes in red
- Set of ~57 significant up-regulated genes at adjusted p-value < 0.01, including SEC61G, RPL39 and SETD5

## How to apply

Load the TNBC1 10x scRNA-seq expression data and subset tumor cells into the two mtDNA clone groups — T0 (Tumor & MT0) and T1 (Tumor & MT1) — using clone assignments obtained from the MQuad/vireoSNP pipeline. Run Seurat's FindMarkers between T0 and T1 with the DESeq2 model and a two-sided Wald test, writing the full DE results (gene, log2 fold change, p-value, adjusted p-value) to tnbc1_de_genes.csv. Select significant genes at adjusted p-value < 0.01 and check that roughly 57 genes are up-regulated in subclone 1, including SEC61G, RPL39 and SETD5. Finally, plot negative log10 P-values against average log2 fold changes with significant genes highlighted in red, saving the figure as tnbc1_de_volcano.png. The rationale is that MQuad-detected mtDNA variants can mark subclones whose propagated mutations may be accompanied by transcriptional differences, so DE analysis validates whether the clonal substructure is biologically meaningful.

## Related tools

- **MQuad** (Upstream: detects clonally informative mtDNA variants whose clones (MT0/MT1) define the DE comparison groups) — github.com/single-cell-genetics/MQuad
- **Seurat** (Runs FindMarkers differential expression between the two mtDNA subclones)
- **DESeq2** (Statistical model used by FindMarkers (two-sided Wald test) for the DE calls)
- **vireoSNP** (Downstream of MQuad: assigns cells to clones based on the mtDNA variant profile, providing the T0/T1 group labels) — https://github.com/single-cell-genetics/vireo

## Examples

```
tnbc_markers <- FindMarkers(tnbc_seurat, ident.1 = "Tumor & MT1", ident.2 = "Tumor & MT0", test.use = "DESeq2"); write.csv(tnbc_markers, "tnbc1_de_genes.csv")
```

## Evaluation signals

- DE table at adjusted p-value < 0.01 contains approximately 57 up-regulated genes in subclone 1
- SEC61G, RPL39 and SETD5 are present among the significant up-regulated genes
- tnbc1_de_genes.csv has the expected columns: gene, log2 fold change, p-value, adjusted p-value
- tnbc1_de_volcano.png exists and displays negative log10 P-value on one axis and average log2 fold change on the other
- Volcano plot highlights significant genes in red

## Limitations

- DE differences between mtDNA clones may be modest — the reported result is 57 genes at adjusted p < 0.01, so weaker transcriptional separation between subclones may not be detectable
- MtDNA mutations complement rather than replace nuclear SNV/CNV lineage markers; subclonal structure from coverage-inferred CNVs may not be as dynamic as subclones shown by propagated point mutations
- Uninformative or noisy mtDNA SNVs confound clonal inference if upstream variant selection is not stringent, which would corrupt the T0/T1 group labels fed to DE

## Evidence

- [results] Between the two mtDNA clones in TNBC1, 57 genes were up-regulated in subclone 1 at adjusted p-value < 0.01, detected with Seurat FindMarkers (DESeq2 model): [evidence span withheld — anchor status: not_found]
- [discussion] MQuad selects high quality informative variants with a binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [readme] use vireoSNP to assign cells to clones based on mtDNA variant profile: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [intro] mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution: "our analysis reveals that mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution."
