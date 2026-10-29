---
name: filtered-cell-proportion-quantification
description: Use when benchmarking or auditing MUSIC-style preprocessing on single-cell
  CRISPR screening data and you need per-dataset QC-filtered and sgRNA-filtered cell
  proportions comparable to published Fig. 4a statistics.
when_to_use_negative:
- The goal is downstream perturbation effect prioritization or topic modeling rather
  than auditing how much data the filtering steps discard
- Input is not yet in MUSIC's standard format (expression matrix plus named perturbation
  vector) and would first require Input_preprocess_10X() conversion
- Working with non-single-cell or bulk CRISPR screening data where per-cell QC thresholds
  (genes, UMIs, mitochondrial percentage) do not apply
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MUSIC
  role: Provides the Input_preprocess(), Cell_qc(), and Cell_filtering() functions
    used to apply QC thresholds (500 genes, 1000 UMIs, 10% mitochondrial) and sgRNA
    filtering (cell_num_threshold=20, filtered_rate=90%) whose cell removal rates
    are quantified
  repo: https://github.com/bm2-lab/MUSIC
- name: R
  role: Runtime environment (R>=3.4.1) in which the MUSIC package is installed via
    devtools and the per-dataset proportion table is computed and written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:2bb59631cff831b4f59cb939526edbc96f806c4ea421e575504884202f6ee05d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# filtered-cell-proportion-quantification

## Summary

Quantify the proportion of single-cell CRISPR screening cells removed by MUSIC's quality control and sgRNA-based cell filtering steps, per dataset and averaged across datasets, to assess how much data each filter discards.

## When to use

Use when benchmarking or auditing MUSIC-style preprocessing on single-cell CRISPR screening data and you need per-dataset QC-filtered and sgRNA-filtered cell proportions comparable to published Fig. 4a statistics.

## When NOT to use

- The goal is downstream perturbation effect prioritization or topic modeling rather than auditing how much data the filtering steps discard
- Input is not yet in MUSIC's standard format (expression matrix plus named perturbation vector) and would first require Input_preprocess_10X() conversion
- Working with non-single-cell or bulk CRISPR screening data where per-cell QC thresholds (genes, UMIs, mitochondrial percentage) do not apply

## Inputs

- Gene-by-cell expression profile matrix (e.g., crop_stimulated expression_profile.txt, genes x cells)
- Named character vector of per-cell sgRNA perturbation annotations (perturb_information; control label must be 'CTRL')
- 14 publicly available single-cell CRISPR screening datasets processed one at a time

## Outputs

- Per-dataset table of QC-filtered and sgRNA-filtered cell proportions (filtered_cell_proportions.csv)
- Cross-dataset average QC-filtered proportion (~6%) and sgRNA-filtered proportion (~41%)
- Cell_filtering diagnostic figure (Invalid_rate.png)

## How to apply

Load the expression profile matrix and perturbation annotation vector (e.g., the crop_stimulated example) and combine them with MUSIC's Input_preprocess(), which also filters mitochondrial (^MRP) and ribosomal protein (^RP) genes. Run Cell_qc with species='Hs' to retain only cells with more than 500 detected genes, more than 1000 UMIs, and less than 10% mitochondrial gene percentage. Then run Cell_filtering on the QC-passed cells with cell_num_threshold=20 and the default filtered_rate of 90% (perturbations whose influenced cells filtered amount to 90% or more are dropped), using cpu_num=30 for parallelization; this step removes perturbed cells whose differential expression profiles resemble controls and perturbations with all-zero expression. Record per-dataset cell counts before QC, after QC, and after sgRNA filtering, compute the two filtered proportions per dataset, average them across datasets, and write the table (filtered_cell_proportions.csv) plus the Cell_filtering diagnostic figure (Invalid_rate.png).

## Related tools

- **MUSIC** (Provides the Input_preprocess(), Cell_qc(), and Cell_filtering() functions used to apply QC thresholds (500 genes, 1000 UMIs, 10% mitochondrial) and sgRNA filtering (cell_num_threshold=20, filtered_rate=90%) whose cell removal rates are quantified) — https://github.com/bm2-lab/MUSIC
- **R** (Runtime environment (R>=3.4.1) in which the MUSIC package is installed via devtools and the per-dataset proportion table is computed and written)

## Examples

```
crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F); crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Cross-dataset average QC-filtered proportion approximates the reported ~6% from Fig. 4a
- Cross-dataset average sgRNA-filtered proportion approximates the reported ~41%
- Every per-dataset filtered proportion lies in [0,1]
- Each dataset row of filtered_cell_proportions.csv contains both the QC-filtered and sgRNA-filtered fraction fields
- Invalid_rate.png is produced by Cell_filtering (plot=T) as the sgRNA filtering diagnostic figure

## Limitations

- The article acknowledges that current filtering steps for data noise may need future refinement, so the measured filtered proportions reflect an imperfect filter
- sgRNA knockout efficiency is imperfect: only 70-80% of sgRNAs generate true loss-of-function of the targeted gene, affecting the validity of sgRNA-based cell filtering
- Off-target analysis in MUSIC is only provided for knockout, not CRISPRi, limiting the skill's applicability to CRISPRi screens

## Evidence

- [discussion] QC thresholds: "MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial"
- [discussion] cell retention rule: "Only cells with the first two factors above the thresholds and the third factor below the threshold are retained"
- [discussion] perturbation filter rule: "if the influenced cells filtered are amount to a high proportion (default 90%) among all, such perturbation is filtered"
- [methods] Cell_qc invocation: "crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)"
- [readme] Cell_filtering invocation: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
- [readme] input preprocessing: "crop_seq_list<-Input_preprocess(expression_profile,perturb_information)"
