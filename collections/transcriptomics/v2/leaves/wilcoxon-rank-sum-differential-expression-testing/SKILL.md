---
name: wilcoxon-rank-sum-differential-expression-testing
description: Use when a scRNA-seq cell population has been binarized by pathway activity
  (e.g., GSDensity compute.cell.label PAL labels) and you must test whether a gene
  of interest is differentially expressed between the pathway-positive and pathway-negative
  cells.
when_to_use_negative:
- The input gene set has not passed the GSDensity coordination test (compute.kld significance);
  the article recommends performing binarization only on gene sets that pass the coordination
  test
- The data are targeted-gene panels with limited numbers of genes — GSDensity-based
  binarization feeding the test is unsuitable there
- Cells are not labeled by pathway activity (no PAL binarization) — the test needs
  a two-group grouping grounded in the pathway activity distribution
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Produces the PAL-based binarized cell labels (compute.cell.label) and run.rwr
    propagation scores that define the G2M-positive/negative groups used as the Wilcoxon
    test grouping
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: NormalizeData-based normalization of the TNBC count matrices and provision
    of per-cell expression values for the Wilcoxon rank-sum test
- name: CelliD
  role: Underlying MCA co-embedding method (via GSDensity's compute.mca) enabling
    the cell-gene graph used to generate pathway activity scores prior to binarization
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:7d27d78739ada36e01be57df420755674fccda8557bbdb191981d344cf18e4e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - scCAD
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wilcoxon-rank-sum-differential-expression-testing

## Summary

Wilcoxon rank-sum testing of a marker gene (e.g., MKI67, TYRO3) between pathway-binarized tumor cell groups defined by GSDensity PAL binarization (G2M checkpoint-positive vs. negative), used to validate that dividing TNBC tumor cells preferentially express proliferation and GAS6-TYRO3 markers.

## When to use

Use when a scRNA-seq cell population has been binarized by pathway activity (e.g., GSDensity compute.cell.label PAL labels) and you must test whether a gene of interest is differentially expressed between the pathway-positive and pathway-negative cells.

## When NOT to use

- The input gene set has not passed the GSDensity coordination test (compute.kld significance); the article recommends performing binarization only on gene sets that pass the coordination test
- The data are targeted-gene panels with limited numbers of genes — GSDensity-based binarization feeding the test is unsuitable there
- Cells are not labeled by pathway activity (no PAL binarization) — the test needs a two-group grouping grounded in the pathway activity distribution

## Inputs

- Seurat-normalized scRNA-seq count matrix (e.g., GSE148673 TNBC-1, GSE161529 TNBC-2/TNBC-5)
- GSDensity PAL-binarized per-cell pathway labels (G2M-positive vs. G2M-negative) from compute.cell.label(run.rwr(...))
- Tumor cell identity calls (copyKat-based tumor cell subsetting)
- Target gene expression vector per cell (MKI67 or TYRO3)

## Outputs

- Wilcoxon rank-sum test statistic and p-value comparing target gene expression between G2M-positive and G2M-negative tumor cells
- tyro3_dividing_tumor_wilcoxon_results.csv with per-sample binarization labels and TYRO3 Wilcoxon p-values

## How to apply

After GSDensity assigns PAL-based labels (compute.cell.label on the run.rwr propagation scores seeded with the Hallmark G2M checkpoint gene set), split tumor cells (identified by copyKat) into G2M-positive and G2M-negative groups and run a Wilcoxon rank-sum test comparing the target gene's normalized expression between groups using Seurat-normalized counts. In TNBC-1 (GSE148673), this tests MKI67 between dividing and non-dividing tumor cells with an expected p < 2.2e-16; in TNBC-5 and the 8-sample GSE161529 cohort, it tests TYRO3, with an expected p = 3.18e-8 for TNBC-5 and p = 0.039 for GSM4909284_TN-MH0114-T2 (the only cohort sample with relatively high TYRO3, detected in 24% of tumor cells). Per-sample binarization labels and p-values should be written to tyro3_dividing_tumor_wilcoxon_results.csv. Acceptance requires reproducing the reported p-values within numerical tolerance and that GAS6/TYRO3 profiles support the GAS6-TYRO3 association with tumor cell division.

## Related tools

- **GSDensity** (Produces the PAL-based binarized cell labels (compute.cell.label) and run.rwr propagation scores that define the G2M-positive/negative groups used as the Wilcoxon test grouping) — https://github.com/qingnanl/gsdensity
- **Seurat** (NormalizeData-based normalization of the TNBC count matrices and provision of per-cell expression values for the Wilcoxon rank-sum test)
- **CelliD** (Underlying MCA co-embedding method (via GSDensity's compute.mca) enabling the cell-gene graph used to generate pathway activity scores prior to binarization) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html

## Examples

```
wilcox.test(MKI67 ~ G2M_label, data = tumor_meta)  # G2M_label from compute.cell.label(run.rwr(compute.nn.edges(compute.mca(seurat_obj), nn.use = 300), gene_set = G2M_checkpoint))
```

## Evaluation signals

- MKI67 Wilcoxon test in TNBC-1 G2M-positive vs. G2M-negative tumor cells reproduces p < 2.2e-16
- TYRO3 Wilcoxon test in TNBC-5 reproduces the reported p = 3.18e-8 within numerical tolerance
- TYRO3 Wilcoxon test for GSM4909284_TN-MH0114-T2 (TYRO3 in 24% of tumor cells) reproduces p = 0.039 within numerical tolerance
- tyro3_dividing_tumor_wilcoxon_results.csv exists and contains per-sample binarization labels plus TYRO3 Wilcoxon p-values for all 8 cohort samples
- PAL distribution used for binarization was thresholded at its antimode (multimode-based antimode detection), and visual inspection of the PAL distribution confirms a bimodal shape

## Limitations

- PAL-based binarization can fail for pathways with more than two modalities; the article recommends visually inspecting PAL distributions of important pathways and correcting automatic results as necessary
- The significant TYRO3 result is only available in the single cohort sample with relatively high TYRO3 expression (GSM4909284_TN-MH0114-T2); other samples may not yield significance
- Batch over- or under-correction may bias pathway activity scores and thus the group definitions used in the test

## Evidence

- [other] Wilcoxon rank-sum tests comparing MKI67 expression between G2M-positive and G2M-negative tumor cells: [evidence span withheld — anchor status: not_found]
- [results] MKI67 higher in dividing cells with p < 2.2e-16: [evidence span withheld — anchor status: not_found]
- [discussion] PAL-based binarization demonstrated on TNBC data: "we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis"
- [discussion] Manual inspection of PAL distributions recommended for binarization validation: "we recommend users to visually inspect the PAL distributions of important pathways to validate the automatic results and perform correction as necessary."
- [discussion] Binarization only for gene sets passing the coordination test: "we recommend performing binarization on only gene sets that pass the coordination test"
- [results] Per-sample TYRO3 result in the 8-sample cohort: [evidence span withheld — anchor status: not_found]
