---
name: te-te-expression-correlation-analysis
description: Use when single-cell BAM/SAM files from a differentiation or development
  time course are available and you need to verify TE quantification quality or detect
  coordinated TE programs by correlating cell-by-TE counts.
when_to_use_negative:
- Input is already a gene or TE feature count table (e.g., a processed matrix) rather
  than raw BAM/SAM alignments — scTE cannot be applied to quantify TEs
- The biological question concerns chromatin accessibility of TEs rather than TE expression
  — use scTEATAC instead of the scRNA-seq pipeline
- Bulk RNA-seq data without single-cell barcodes/UMIs — the correlation-across-cells
  logic does not apply
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTE
  role: Quantifies TE expression per cell from BAM input, producing the .h5ad cell-by-TE
    matrix used for cross-cell TE–TE correlation
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the hg38 gene/TE genome index from GENCODE GTF and UCSC rmsk annotations
    used as the -x index for scTE
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Produces the input BAM files with CB:Z and UB:Z barcode/UMI tags that scTE
    consumes via -CB CB -UMI UB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:5e063f1f9726a9ec0a6259068c6f275d336f5f88133be2eb8d69af988eb26000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - scTE
  - scTE_build
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# te-te-expression-correlation-analysis

## Summary

Quantify transposable element expression per cell with scTE and validate TE co-expression structure by computing cross-cell correlations between TE pairs (e.g., LTR7 and HERVH-int), checking them against known pluripotency dynamics during hPSC differentiation.

## When to use

Use when single-cell BAM/SAM files from a differentiation or development time course are available and you need to verify TE quantification quality or detect coordinated TE programs by correlating cell-by-TE counts.

## When NOT to use

- Input is already a gene or TE feature count table (e.g., a processed matrix) rather than raw BAM/SAM alignments — scTE cannot be applied to quantify TEs
- The biological question concerns chromatin accessibility of TEs rather than TE expression — use scTEATAC instead of the scRNA-seq pipeline
- Bulk RNA-seq data without single-cell barcodes/UMIs — the correlation-across-cells logic does not apply

## Inputs

- Unfiltered Cell Ranger BAM files with cell barcodes and UMIs in CB:Z and UB:Z tags (e.g., E-MTAB-6268 human cardiac differentiation data)
- hg38 scTE genome index built via scTE_build from GENCODE GTF and UCSC hg38 rmsk.txt.gz annotations

## Outputs

- Cell-by-TE count matrix (.h5ad) from scTE
- Pearson correlation coefficient between LTR7 and HERVH-int across cells
- Verification summary of LTR7/HERVH-int co-downregulation with POU5F1 during differentiation

## How to apply

Build an hg38 index with scTE_build from the GENCODE gene GTF and the UCSC hg38 rmsk.txt.gz repeat annotation, then run scTE on the unfiltered BAM files using the default read-allocation mode (reads assigned first to gene exons, then to TEs) with -CB CB -UMI UB and --hdf5 True (~10 GB memory per thread). Extract the cell-by-TE count matrix from the resulting .h5ad output and compute the Pearson correlation between LTR7 and HERVH-int across cells; the article reports these primate-specific TEs as co-expressed in human ESCs. As a second check, verify that LTR7/HERVH-int expression is downregulated in parallel with POU5F1 over the differentiation time course. This validates that TE-level quantification recapitulates known biology before using the matrix for downstream branch-specific comparisons.

## Related tools

- **scTE** (Quantifies TE expression per cell from BAM input, producing the .h5ad cell-by-TE matrix used for cross-cell TE–TE correlation) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds the hg38 gene/TE genome index from GENCODE GTF and UCSC rmsk annotations used as the -x index for scTE) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Produces the input BAM files with CB:Z and UB:Z barcode/UMI tags that scTE consumes via -CB CB -UMI UB)

## Examples

```
scTE -i input.bam -o out -x hg38.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- LTR7 vs HERVH-int Pearson correlation across cells is approximately 0.8
- LTR7 and HERVH-int expression are co-downregulated alongside POU5F1 during hPSC differentiation
- The scTE --hdf5 True output yields a valid cell-by-TE matrix extractable from the .h5ad file
- Reproduced TE phenomena (LTR7/HERVH expression in human ESCs) match the paper's reported patterns, indicating correct index construction and read allocation

## Limitations

- scTE assigns reads first to gene exons, then to TEs by default, so TEs inside exon/UTR regions contribute only to the gene and not the TE score unless -m inclusive is set
- TE information content is lower than that of genes; TE data serve mainly as an addendum to gene information
- scTE recommends unfiltered alignment files as input; filtered BAMs may distort barcode/UMI counting

## Evidence

- [discussion] scTE reproduces previously identified TE expression phenomena (MERVL in mouse ESCs, LTR7/HERVH in human ESCs): "By taking advantage of this tool, we could observe previously identified phenomena such as MERVL and LTR7/HERVH expression in mouse and human ESCs, respectively."
- [readme] Default read allocation: genes first, then TEs: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default."
- [readme] Cell Ranger BAM barcode/UMI tag convention for scTE: "the cell barcodes and UMI need to be integrated into the read 'CB:Z' or 'UB:Z' tage"
- [abstract] TEs dynamically regulated during differentiation: "Specific TE types are expressed in subpopulations of embryonic stem cells and are dynamically regulated during pluripotency reprogramming, differentiation, and embryogenesis"
