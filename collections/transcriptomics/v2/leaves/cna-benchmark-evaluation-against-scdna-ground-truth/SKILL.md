---
name: cna-benchmark-evaluation-against-scdna-ground-truth
description: Use when comparing CNA callers on the same snRNA-seq sample where a single-cell
  DNA copy-number profile exists for a clonal subset, and per-cell copy gain/loss
  sensitivity–specificity (ROC-AUC) is the evaluation metric.
when_to_use_negative:
- No single-cell DNA copy-number ground truth is available for the clone of interest
  — ROC-AUC scoring cannot be performed and only qualitative concordance checks apply.
- The analysis goal is detecting copy-neutral LOH with InferCNV or CopyKAT, which
  do not consider allelic information and hence cannot detect copy-neutral loss of
  heterozygosity (LOH).
- The dataset is bulk DNA-seq or already-phased single-cell DNA data, where direct
  copy-number calling from DNA makes scRNA-seq-based CNA inference unnecessary.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Primary CNA caller under benchmark; RDR and BAF modules run with XCloneConfig
    and merged via run_combine to produce per-cell, per-gene CNA states
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the three XClone input matrices (total
    read depth for RDR; AD/DP for BAF) from SAM/BAM/CRAM files
  repo: https://github.com/hxj5/xcltk
- name: InferCNV
  role: Competing expression-based CNA caller scored for copy-gain ROC-AUC; cannot
    use allelic information
- name: CopyKAT
  role: Competing CNA caller using polynomial dynamic programming smoothing; scored
    for copy-gain ROC-AUC only
- name: CaSpER
  role: Competing CNA caller using allelic shift signal from individual SNPs; scored
    for copy-gain ROC-AUC
- name: Numbat
  role: Competing haplotype-based CNA caller aggregating variants via population phasing;
    scored for both copy-gain and copy-loss ROC-AUC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:f8ab93d46f8b023bb1f2bdf87c885a38af51fa04f9d70c5f965a28881359c7cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CaSpER
  - CopyKAT
  - InferCNV
  - Numbat
  - XClone
  - xcltk
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cna-benchmark-evaluation-against-scdna-ground-truth

## Summary

Benchmarks single-cell RNA-seq CNA-detection methods (XClone, CaSpER, Numbat, InferCNV, CopyKAT) by computing ROC curves and ROC-AUC for per-cell copy-gain and copy-loss calls against single-cell DNA-seq (scOne-seq) copy-number profiles used as ground truth.

## When to use

Use when comparing CNA callers on the same snRNA-seq sample where a single-cell DNA copy-number profile exists for a clonal subset, and per-cell copy gain/loss sensitivity–specificity (ROC-AUC) is the evaluation metric.

## When NOT to use

- No single-cell DNA copy-number ground truth is available for the clone of interest — ROC-AUC scoring cannot be performed and only qualitative concordance checks apply.
- The analysis goal is detecting copy-neutral LOH with InferCNV or CopyKAT, which do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH).
- The dataset is bulk DNA-seq or already-phased single-cell DNA data, where direct copy-number calling from DNA makes scRNA-seq-based CNA inference unnecessary.

## Inputs

- Cell-by-gene total read-depth matrix from the 4416-cell 2R astrocytoma snRNA-seq sample (XClone RDR input)
- Allele-specific AD and DP matrices with hg38 gene/barcode annotation (XClone BAF input)
- Per-cell copy gain/loss calls from InferCNV, CopyKAT, CaSpER, and Numbat run on the same dataset
- scOne-seq single-cell DNA copy-number profile of the 53-cell 2R clone 1 (ground truth labels)

## Outputs

- auc_roc_benchmark_astrocytoma_2R.csv containing the AUC comparison table and per-method ROC curve point data
- Per-method ROC-AUC values for copy gain (XClone 0.924, CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798) and copy loss (XClone 0.865, Numbat 0.619)

## How to apply

Run each CNA caller on the identical cell-by-gene input: for XClone build the AnnData objects from the 4416-cell 2R astrocytoma snRNA-seq dataset via xclone.pp.xclonedata (total read-depth matrix for RDR; allele-specific AD and DP matrices for BAF with hg38 gene/barcode annotation, plus xclone.pp.extra_anno cell annotations), then run xclone.model.run_RDR, run_BAF (XCloneConfig with e.g. BAF_denoise=True and HMM transition parameter trans_t), and run_combine to merge modules. Run InferCNV, CopyKAT, CaSpER, and Numbat on the same dataset with their recommended settings. Assign per-cell copy gain/loss labels from each method and score them against the scOne-seq single-cell DNA copy-number profile of the 53-cell 2R clone 1 as ground truth. Compute ROC curves and ROC-AUC values with standard ROC analysis — copy-gain detection across all five methods (XClone 0.924, CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798) and copy-loss detection for XClone vs Numbat only (0.865 vs 0.619), since InferCNV/CopyKAT do not use allelic information. Assemble per-method AUCs and ROC curve points into a single CSV for validation against published values.

## Related tools

- **XClone** (Primary CNA caller under benchmark; RDR and BAF modules run with XCloneConfig and merged via run_combine to produce per-cell, per-gene CNA states) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the three XClone input matrices (total read depth for RDR; AD/DP for BAF) from SAM/BAM/CRAM files) — https://github.com/hxj5/xcltk
- **InferCNV** (Competing expression-based CNA caller scored for copy-gain ROC-AUC; cannot use allelic information)
- **CopyKAT** (Competing CNA caller using polynomial dynamic programming smoothing; scored for copy-gain ROC-AUC only)
- **CaSpER** (Competing CNA caller using allelic shift signal from individual SNPs; scored for copy-gain ROC-AUC)
- **Numbat** (Competing haplotype-based CNA caller aggregating variants via population phasing; scored for both copy-gain and copy-loss ROC-AUC)

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig); BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig)
```

## Evaluation signals

- Computed copy-gain AUCs fall within tolerance of published values: XClone 0.924, CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798.
- Computed copy-loss AUCs match: XClone 0.865 vs Numbat 0.619, with XClone ranking highest for both gain and loss.
- auc_roc_benchmark_astrocytoma_2R.csv contains both the AUC comparison table and per-method ROC curve point data.
- All five methods are scored on the same 4416-cell input and the same 53-cell 2R clone 1 scDNA ground-truth labels.
- Copy-loss AUCs exist only for methods using allelic information (XClone, Numbat, CaSpER) — absence of InferCNV/CopyKAT loss AUCs is expected, not an error.

## Limitations

- Methods without allelic signal (InferCNV, CopyKAT) cannot be scored for copy loss and cannot detect copy-neutral LOH, restricting copy-loss benchmarking to XClone vs Numbat.
- Ground truth is limited to the 53-cell scOne-seq profile of the 2R clone 1, so scoring depends on clone labeling and a small reference cell count.
- Competing methods must be run with their recommended settings; results may vary with configuration choices such as XClone's BAF_denoise and trans_t HMM transition parameter.
- Numbat may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level, which affects fairness of the per-cell comparison.

## Evidence

- [results] XClone achieved the highest AUC for detecting copy gain (0.924, vs CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798) and copy loss (0.865, vs Numbat 0.619) in the 2R clone 1.: [evidence span withheld — anchor status: not_found]
- [methods] Run XClone RDR/BAF/combine modules on the astrocytoma dataset.: [evidence span withheld — anchor status: not_found]
- [intro] InferCNV/CopyKAT cannot detect copy-neutral LOH.: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [readme] xcltk generates the three XClone input matrices.: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [methods] Run competing CNA callers on the same dataset with recommended settings.: [evidence span withheld — anchor status: not_found]
