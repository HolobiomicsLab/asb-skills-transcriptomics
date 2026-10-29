---
name: rna-seq-outlier-sample-exclusion
description: Use when reproducing a published RNA-seq differential expression analysis
  whose original authors identified an outlier sample via QA/QC, or when PCA/box-plot
  diagnostics reveal a mislabeled or poorly separated sample before fitting a DE model.
when_to_use_negative:
- The original publication did not flag any outlier sample and QC plots show clean
  separation by the metadata of interest — excluding samples would break reproducibility
- Input is already a normalized expression matrix from a microarray rather than RNA-seq
  counts (use the array workflow with 'mean' summary and no count filter instead)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalystR
  role: R package used to load the count table, generate QC box-plot/PCA diagnostics,
    normalize with limma voom, and run the DE analysis after outlier exclusion
  repo: github.com/xia-lab/ExpressAnalystR
- name: limma
  role: Normalization (voom log2 CPM) and differential expression engine with species
    as blocking factor, applied to the retained samples
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:8442932c2157914734e77bee86412e61c22617cb3c284f07a5907729c0ee9298
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# rna-seq-outlier-sample-exclusion

## Summary

Identify and exclude outlier RNA-seq samples flagged by QA/QC diagnostics (box plots, PCA) before differential expression analysis, so that downstream DEG counts reflect the same sample set as a reference publication. In the ExpressAnalyst salamander case study, this meant dropping SRR7499350 from the SRR7499348–SRR7499365 range prior to limma DE analysis.

## When to use

Use when reproducing a published RNA-seq differential expression analysis whose original authors identified an outlier sample via QA/QC, or when PCA/box-plot diagnostics reveal a mislabeled or poorly separated sample before fitting a DE model.

## When NOT to use

- The original publication did not flag any outlier sample and QC plots show clean separation by the metadata of interest — excluding samples would break reproducibility
- Input is already a normalized expression matrix from a microarray rather than RNA-seq counts (use the array workflow with 'mean' summary and no count filter instead)

## Inputs

- RNA-seq gene count table covering the full sample range (e.g., SRR7499348–SRR7499365 as tab-delimited matrix)
- Sample metadata with the factor of interest (time) and blocking factor (species)
- Prior QA/QC outlier designation from the original publication (e.g., SRR7499350) or locally generated PCA/box-plot diagnostics

## Outputs

- QC box-plot and PCA PNG files used to justify sample exclusion
- DE results table (logFC, p-value, FDR-adjusted p-value per gene) written to salamander_de_results.csv for the 17 retained samples
- DEG count at FDR < 0.1 (validated against 2780 in the salamander case)

## How to apply

Load the count table for the full sample range (e.g., SRR7499348–SRR7499365) into ExpressAnalystR via Init.Data(FALSE), SetAnalType("onedata") and ReadTabExpressData. Generate QC diagnostics with PlotDataBox and PlotDataPCA to inspect expression distributions and sample separation by the metadata of interest (here, time as primary factor, species as blocking factor). Exclude any sample flagged as an outlier by the original publication's QA/QC (in the salamander study, SRR7499350) before normalization. Normalize the remaining samples with limma voom (log2 CPM), then build the limma design matrix and run PerformDEAnal with the custom contrast (time0 vs time24). Apply FDR adjustment and call DEGs at adj. p-value < 0.1 (no log2FC cut-off), matching the original salamander publication threshold; exclusion of the outlier is part of the exact recipe that reproduces the 2780-DEG result.

## Related tools

- **ExpressAnalystR** (R package used to load the count table, generate QC box-plot/PCA diagnostics, normalize with limma voom, and run the DE analysis after outlier exclusion) — github.com/xia-lab/ExpressAnalystR
- **limma** (Normalization (voom log2 CPM) and differential expression engine with species as blocking factor, applied to the retained samples)

## Examples

```
Init.Data(FALSE); SetAnalType("onedata"); ReadTabExpressData("salamander_counts.txt"); PlotDataBox("salamander_counts.txt","qc_boxplot_",72,"png"); PlotDataPCA("salamander_counts.txt","qc_pca_",72,"png");
```

## Evaluation signals

- The DEG count at FDR-adjusted p < 0.1 (no log2FC cut-off) matches the 2780 DEGs reported in the salamander publication
- The analyzed sample set is SRR7499348–SRR7499365 minus SRR7499350 (17 samples), verifiable from the count-table columns fed to limma
- QC artifacts qc_boxplot_dpi72.png and qc_pca_dpi72.png are produced and reviewed before normalization
- The DE results table contains logFC, p-value and FDR-adjusted p-value columns per gene
- Normalization method is limma voom (log2 CPM) applied to RNA-seq counts, consistent with the paper's stated transformation

## Limitations

- Outlier status (SRR7499350) was determined by QA/QC in the original publication; without that designation, the exclusion decision depends on subjective inspection of PCA/box plots
- ExpressAnalystR is still under development — full functionality is not guaranteed (per README)
- ExpressAnalystR output values are saved as .qs files (readDataset(fileName)); function return values are often not directly useful in local usage
- Reproducibility gap: metadata such as sample sex details were not included in the original manuscript, limiting covariate choices

## Evidence

- [discussion] outlier exclusion in salamander case study: "excluding sample SRR7499350 which was identified as an outlier by QA/QC performed by the original publication"
- [discussion] DEG threshold for salamander analysis: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [readme] QC diagnostics justify checking sample separation: "Depending of your experimental design, try to see if the samples are separated by the metadata of interest, it can also be used to see whether there are potentially mislabed sample."
- [discussion] sample range for salamander RNA-seq data: "Salamander RNA-seq data were obtained from NCBI’s SRA at sample accessions SRR7499348 - SRR7499365"
