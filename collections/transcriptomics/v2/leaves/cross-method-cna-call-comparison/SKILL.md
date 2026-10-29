---
name: cross-method-cna-call-comparison
description: Use when a task asks to compare ROC-AUC performance of CNA callers on
  snRNA-seq/scRNA-seq data, or to reproduce published AUC benchmark values against
  a single-cell DNA copy-number ground truth.
when_to_use_negative:
- 'Methods cannot be scored on copy loss if they do not model allelic information:
  InferCNV and CopyKAT ''do not consider allelic information and hence cannot detect
  copy-neutral loss of heterozygosity (LOH)'', limiting loss-event comparisons to
  allele-aware callers (XClone, Numbat, CaSpER)'
- The analysis goal is de novo CNA discovery in a single dataset rather than quantitative
  method benchmarking with an independent single-cell DNA ground truth
- The input data lacks allele-specific AD/DP information entirely, so a fair allele-aware
  comparison (BAF vs RDR vs allelic methods) is impossible
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: XClone
  role: Primary CNA caller being benchmarked; runs RDR and BAF modules and combines
    them to produce per-cell, per-gene CNA states
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the three input matrices (AD, DP, total
    read depth) from SAM/BAM/CRAM files for XClone
  repo: https://github.com/hxj5/xcltk
- name: CaSpER
  role: Competing allelic-shift-based CNA caller run on the same snRNA-seq dataset
    with recommended settings for AUC comparison
- name: Numbat
  role: Competing population haplotype-based phasing CNA caller scored against XClone
    for both copy gain and copy loss AUC
- name: InferCNV
  role: Competing read-depth-only CNA caller benchmarked on copy gain detection
- name: CopyKAT
  role: Competing polynomial dynamic programming CNA caller benchmarked on copy gain
    detection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:ec02ab5ecc8fd16c72023bc525d182de503c27ba45b36e602ca2eaeb9139cca1
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-method-cna-call-comparison

## Summary

Benchmark single-cell CNA-detection methods (XClone, CaSpER, Numbat, InferCNV, CopyKAT) by assigning per-cell copy gain/loss labels on the same scRNA-seq dataset, scoring them against single-cell DNA ground truth (scOne-seq), and comparing ROC-AUC values for copy gain and copy loss detection.

## When to use

Use when a task asks to compare ROC-AUC performance of CNA callers on snRNA-seq/scRNA-seq data, or to reproduce published AUC benchmark values against a single-cell DNA copy-number ground truth.

## When NOT to use

- Methods cannot be scored on copy loss if they do not model allelic information: InferCNV and CopyKAT 'do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)', limiting loss-event comparisons to allele-aware callers (XClone, Numbat, CaSpER)
- The analysis goal is de novo CNA discovery in a single dataset rather than quantitative method benchmarking with an independent single-cell DNA ground truth
- The input data lacks allele-specific AD/DP information entirely, so a fair allele-aware comparison (BAF vs RDR vs allelic methods) is impossible

## Inputs

- Cell-by-gene total read-depth matrix from snRNA-seq (e.g. 4416-cell 2R astrocytoma dataset) for XClone RDR module
- Allele-specific AD and DP matrices (BAF signals) from xcltk preprocessing
- hg38 gene and barcode annotation (xclone.pp.load_anno genome_mode='hg38_genes')
- scOne-seq single-cell DNA copy-number profile of the target clone as ground-truth labels (e.g. 53-cell 2R clone 1)
- Per-cell copy gain/loss calls from competing methods: InferCNV, CopyKAT, CaSpER, Numbat

## Outputs

- AUC comparison table (e.g. auc_roc_benchmark_astrocytoma_2R.csv) with per-method AUC values for copy gain and copy loss
- Per-method ROC curve point data
- Per-cell, per-gene CNA states from the XClone combine module used for label assignment

## How to apply

Build each method's required input from the same raw cell-by-gene data: for XClone, construct the RDR AnnData (total read-depth matrix) and BAF AnnData (allele-specific AD/DP matrices) with xclone.pp.xclonedata, add hg38 gene/barcode annotations via xclone.pp.extra_anno, then run xclone.model.run_RDR, xclone.model.run_BAF (e.g. with XCloneConfig, BAF_denoise=True and the HMM transition parameter trans_t) and merge with xclone.model.run_combine. Run InferCNV, CopyKAT, CaSpER, and Numbat on the identical dataset with their recommended settings. Assign per-cell copy gain/loss labels from each method and score against the scOne-seq single-cell copy-number profile (e.g. the 53-cell 2R clone 1). Compute ROC curves and AUCs per event class (copy gain for all five methods; copy loss for XClone vs Numbat, since InferCNV and CopyKAT ignore allelic information), then assemble an AUC comparison table such as auc_roc_benchmark_astrocytoma_2R.csv and verify AUCs fall within tolerance of published values.

## Related tools

- **XClone** (Primary CNA caller being benchmarked; runs RDR and BAF modules and combines them to produce per-cell, per-gene CNA states) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the three input matrices (AD, DP, total read depth) from SAM/BAM/CRAM files for XClone) — https://github.com/hxj5/xcltk
- **CaSpER** (Competing allelic-shift-based CNA caller run on the same snRNA-seq dataset with recommended settings for AUC comparison)
- **Numbat** (Competing population haplotype-based phasing CNA caller scored against XClone for both copy gain and copy loss AUC)
- **InferCNV** (Competing read-depth-only CNA caller benchmarked on copy gain detection)
- **CopyKAT** (Competing polynomial dynamic programming CNA caller benchmarked on copy gain detection)

## Examples

```
BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file = xconfig); combined = xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata)
```

## Evaluation signals

- Reproduced copy-gain AUCs match published values within tolerance: XClone 0.924, CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798
- Reproduced copy-loss AUCs match published values within tolerance: XClone 0.865 vs Numbat 0.619
- All methods were run on the identical dataset (4416-cell 2R astrocytoma snRNA-seq) with competing methods at their recommended settings, ensuring a fair comparison
- Per-cell labels are scored against the scOne-seq single-cell copy-number profile of the same 53-cell 2R clone 1, not against pseudo-bulk ground truth
- Output table auc_roc_benchmark_astrocytoma_2R.csv contains both the AUC comparison and per-method ROC curve point data

## Limitations

- HoneyBADGER and CaSpER's independent use of individual SNPs suffers high noise due to extremely low coverage of a single variant, which can depress their benchmark scores
- Numbat may sacrifice the resolution of clonal CNAs from single-cell to pseudo-bulk level, and its performance and robustness remain to be further assessed
- InferCNV and CopyKAT cannot detect copy-neutral LOH or contribute to copy-loss AUC comparisons because they ignore allelic information

## Evidence

- [results] XClone achieved the highest AUC for detecting copy gain (0.924, vs CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798) and copy loss (0.865, vs Numbat 0.619) in the 2R clone 1.: [evidence span withheld — anchor status: not_found]
- [intro] Methods comparison scope: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [intro] Numbat resolution caveat: "However, this method may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level and its performance and robustness remain to be further assessed"
- [readme] xcltk input matrices: [evidence span withheld — anchor status: external_doc]
- [methods] XClone module combination: "BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata,
                config_file = xconfig)"
