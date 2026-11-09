---
name: deseq2-wald-test-count-modeling
description: Use when raw UMI count data for single cells are split into two pre-defined
  clone groups (e.g., by mtDNA variants from MQuad/vireoSNP) and you need a ranked
  DE gene list with FDR control.
when_to_use_negative:
- Data are already normalized/transformed expression matrices rather than raw UMI
  counts — the DESeq2 model assumes raw counts.
- No pre-defined clone labels exist yet; clone groups must first be reconstructed
  from mtDNA variants (cellsnp-lite → MQuad → vireoSNP).
- The goal is genotype calling or clonal reconstruction itself, not transcriptional
  comparison of defined groups.
tools:
- name: Seurat
  role: Provides FindMarkers, which wraps the DESeq2 count model to compute two-sided
    Wald-test differential expression between the T0 and T1 clone groups.
- name: DESeq2
  role: Negative-binomial count model underlying FindMarkers' statistical test; supplies
    log2FC, p-value, and adjusted p-value per gene.
- name: vireoSNP
  role: Upstream step assigning individual cells to mtDNA-based clones, providing
    the T0/T1 group labels used in the DE contrast.
  repo: https://github.com/single-cell-genetics/vireo
- name: MQuad
  role: Upstream step selecting clonally informative mtDNA variants (binomial mixture
    model, ΔBIC via Kneedle) that define the clone structure for the DE contrast.
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:26eb5da1aa242f3fee59567e4814da83eaba9ff58d30b451663f987dcb294a43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - DESeq2
  - MQuad
  - Seurat
  - vireoSNP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# DESeq2 Wald-test count modeling for mtDNA-clone differential expression

## Summary

Runs two-sided differential expression between mitochondrial-subclone-defined cell groups (e.g., TNBC1 T0 Tumor&MT0 vs T1 Tumor&MT1) using Seurat FindMarkers backed by the DESeq2 count model, producing a gene-level table (log2 FC, p, adjusted p) at adjusted p < 0.01 and a volcano plot.

## When to use

Use when raw UMI count data for single cells are split into two pre-defined clone groups (e.g., by mtDNA variants from MQuad/vireoSNP) and you need a ranked DE gene list with FDR control.

## When NOT to use

- Data are already normalized/transformed expression matrices rather than raw UMI counts — the DESeq2 model assumes raw counts.
- No pre-defined clone labels exist yet; clone groups must first be reconstructed from mtDNA variants (cellsnp-lite → MQuad → vireoSNP).
- The goal is genotype calling or clonal reconstruction itself, not transcriptional comparison of defined groups.

## Inputs

- Seurat object (or raw UMI count matrix) for the TNBC1 10x scRNA-seq sample
- Per-cell mtDNA clone assignments from MQuad/vireoSNP (T0 = Tumor & MT0, T1 = Tumor & MT1)
- Cell metadata distinguishing tumor from non-tumor cells for subsetting

## Outputs

- tnbc1_de_genes.csv with columns gene, log2 fold change, p-value, adjusted p-value
- Significant gene set at adjusted p < 0.01 (~57 up-regulated in subclone 1, incl. SEC61G, RPL39, SETD5)
- tnbc1_de_volcano.png volcano plot (x = average log2 FC, y = negative log10 P-value, significant genes in red)

## How to apply

Load the scRNA-seq expression data (10x TNBC1) and subset tumor cells into the two mtDNA-defined groups T0 (Tumor & MT0) and T1 (Tumor & MT1) using clone assignments reconstructed from MQuad-selected mtDNA variants. Call Seurat's FindMarkers with the DESeq2 model (two-sided Wald test) on the raw count counts to obtain gene, log2 fold change, p-value, and adjusted p-value for every gene. Write the full DE table to tnbc1_de_genes.csv, then select significant genes at adjusted p-value < 0.01 and check that roughly 57 genes are up-regulated in subclone 1, including SEC61G, RPL39, and SETD5. Finally render a volcano plot of negative log10 p-values against average log2 fold changes, highlighting significant genes in red, saved as tnbc1_de_volcano.png. The rationale for a count-based model (DESeq2) is that single-cell UMI counts follow overdispersed negative-binomial-like noise that the Wald test handles better than simple rank or t-test comparisons.

## Related tools

- **Seurat** (Provides FindMarkers, which wraps the DESeq2 count model to compute two-sided Wald-test differential expression between the T0 and T1 clone groups.)
- **DESeq2** (Negative-binomial count model underlying FindMarkers' statistical test; supplies log2FC, p-value, and adjusted p-value per gene.)
- **vireoSNP** (Upstream step assigning individual cells to mtDNA-based clones, providing the T0/T1 group labels used in the DE contrast.) — https://github.com/single-cell-genetics/vireo
- **MQuad** (Upstream step selecting clonally informative mtDNA variants (binomial mixture model, ΔBIC via Kneedle) that define the clone structure for the DE contrast.) — https://github.com/single-cell-genetics/MQuad

## Examples

```
de <- FindMarkers(tumor_obj, ident.1 = 'T1', ident.2 = 'T0', test.use = 'DESeq2'); write.csv(de, 'tnbc1_de_genes.csv')
```

## Evaluation signals

- The DE table at adjusted p < 0.01 contains ~57 up-regulated genes in subclone 1 (T1), matching the published count (Fig. 5e).
- SEC61G, RPL39, and SETD5 are present among the significant up-regulated genes.
- tnbc1_de_genes.csv has the required columns: gene, log2 fold change, p-value, adjusted p-value.
- tnbc1_de_volcano.png plots negative log10 P-values against average log2 fold changes with significant genes highlighted in red.
- The contrast uses exactly the mtDNA-defined tumor subsets T0 (Tumor & MT0) and T1 (Tumor & MT1), not all cells.

## Limitations

- The reported transcriptional signal between mtDNA subclones is modest (57 genes at adj p < 0.01), so results are sensitive to the adjusted-p threshold and group assignment quality.
- Clone labels depend on the upstream cellsnp-lite/MQuad/vireoSNP pipeline; misassigned cells will dilute the DE signal.
- Tested in the paper only on the TNBC1 10x scRNA-seq dataset; SMART-seq2 or scATAC data with different count semantics were not the basis of this DE comparison.

## Evidence

- [other] Run differential expression between T0 and T1 with the FindMarkers function from Seurat using the DESeq2 model (two-sided Wald test).: [evidence span withheld — anchor status: not_found]
- [other] 57 genes up-regulated at adj p < 0.01, verified via key marker genes.: [evidence span withheld — anchor status: not_found]
- [readme] Clone groups derive from the MQuad/vireoSNP mtDNA clonality pipeline.: "3. use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
