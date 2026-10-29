---
name: roc-auc-computation-for-copy-number-detection
description: Reference for quantifying and comparing CNA-detection accuracy of scRNA-seq
  copy-number callers via ROC-AUC against single-cell DNA-seq ground truth, as done
  for XClone on the 4416-cell 2R astrocytoma minor clone.
when_to_use_negative:
- The input is bulk DNA-seq or bulk RNA-seq copy-number data rather than single-cell
  transcriptomic matrices with allelic AD/DP information
- No single-cell DNA ground truth (e.g. scOne-seq clone profile) is available, so
  ROC-AUC scoring cannot be performed
- The dataset lacks heterozygous SNP coverage, making the BAF module and allele-specific
  benchmarks inapplicable (RDR-only evaluation would be required)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3178
tools:
- name: XClone
  role: Primary CNA caller whose RDR and BAF modules are combined via run_combine
    to produce per-cell copy gain/loss labels for AUC scoring
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit generating the three XClone input matrices (AD, DP,
    total read depth) from BAM/CRAM files
  repo: https://github.com/hxj5/xcltk
- name: CaSpER
  role: Competing CNA caller benchmarked on copy-gain AUC (0.827)
- name: Numbat
  role: Competing CNA caller benchmarked on copy-gain (0.814) and copy-loss (0.619)
    AUC
- name: InferCNV
  role: Competing expression-based CNA caller benchmarked on copy-gain AUC (0.808)
- name: CopyKAT
  role: Competing expression-based CNA caller benchmarked on copy-gain AUC (0.798)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:ad06994210d6e12a862141afa8d6e6b30e8ff788832412715f906e7af151f3ff
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3178
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# roc-auc-computation-for-copy-number-detection

## Summary

Benchmarking protocol that scores per-cell copy gain/loss calls from XClone (RDR+BAF modules) and competing CNA callers (CaSpER, Numbat, InferCNV, CopyKAT) against scOne-seq single-cell DNA copy-number profiles, then computes ROC-AUC per method on the astrocytoma 2R clone 1 snRNA-seq data.

## When to use

Reference for quantifying and comparing CNA-detection accuracy of scRNA-seq copy-number callers via ROC-AUC against single-cell DNA-seq ground truth, as done for XClone on the 4416-cell 2R astrocytoma minor clone.

## When NOT to use

- The input is bulk DNA-seq or bulk RNA-seq copy-number data rather than single-cell transcriptomic matrices with allelic AD/DP information
- No single-cell DNA ground truth (e.g. scOne-seq clone profile) is available, so ROC-AUC scoring cannot be performed
- The dataset lacks heterozygous SNP coverage, making the BAF module and allele-specific benchmarks inapplicable (RDR-only evaluation would be required)

## Inputs

- Cell-by-gene total read-depth matrix from 10x Genomics snRNA-seq (astrocytoma 2R clone, 4416 cells)
- Allele-specific AD and DP matrices for BAF (from xcltk BAF module)
- hg38 gene and block annotations (xclone.pp.load_anno genome_mode 'hg38_genes' / 'hg38_blocks')
- Binary per-cell copy gain/loss labels from XClone, CaSpER, Numbat, InferCNV, CopyKAT
- scOne-seq single-cell DNA copy-number profile of the 53-cell 2R clone 1 as ground truth

## Outputs

- auc_roc_benchmark_astrocytoma_2R.csv with per-method copy-gain and copy-loss AUC comparison table
- Per-method ROC curve point data for copy-gain (XClone vs CaSpER, Numbat, InferCNV, CopyKAT) and copy-loss (XClone vs Numbat) detection
- Per-cell, per-gene CNA states from xclone.model.run_combine

## How to apply

Build XClone input AnnData objects from the 4416-cell 2R astrocytoma snRNA-seq sample with xclone.pp.xclonedata (cell-by-gene total read-depth matrix for RDR; allele-specific AD and DP matrices for BAF; hg38 gene/barcode annotation via hg38_genes and hg38_blocks, extra annotations via xclone.pp.extra_anno). Run the RDR and BAF modules (xclone.model.run_RDR and xclone.model.run_BAF with XCloneConfig, e.g. BAF_denoise=True and a chosen HMM transition parameter trans_t), merge with xclone.model.run_combine, then run InferCNV, CopyKAT, CaSpER and Numbat on the same dataset with recommended settings. Assign per-cell gain/loss labels from each caller and score them against the scOne-seq copy-number profile of the 53-cell 2R clone 1 as single-cell ground truth; compute ROC curves and AUC for copy-gain (all five methods) and copy-loss (XClone vs Numbat) detection, assembling results into auc_roc_benchmark_astrocytoma_2R.csv. The rationale is that XClone combines BAF and RDR smoothing (HMM along genome coordinates plus KNN cell-cell graph) to strengthen read-depth and allelic-imbalance signals, so its single-cell-resolution calls should outperform methods that use only expression (InferCNV, CopyKAT) or independent SNPs (HoneyBADGER, CaSpER).

## Related tools

- **XClone** (Primary CNA caller whose RDR and BAF modules are combined via run_combine to produce per-cell copy gain/loss labels for AUC scoring) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit generating the three XClone input matrices (AD, DP, total read depth) from BAM/CRAM files) — https://github.com/hxj5/xcltk
- **CaSpER** (Competing CNA caller benchmarked on copy-gain AUC (0.827))
- **Numbat** (Competing CNA caller benchmarked on copy-gain (0.814) and copy-loss (0.619) AUC)
- **InferCNV** (Competing expression-based CNA caller benchmarked on copy-gain AUC (0.808))
- **CopyKAT** (Competing expression-based CNA caller benchmarked on copy-gain AUC (0.798))

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig)
```

## Evaluation signals

- Copy-gain AUCs reproduce published values within tolerance: XClone 0.924, CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798
- Copy-loss AUCs reproduce published values: XClone 0.865, Numbat 0.619
- All methods are scored against the same scOne-seq 53-cell 2R clone 1 ground-truth profile on the same 4416-cell snRNA-seq input
- ROC curves are monotone with points in [0,1] and AUC values within [0.5, 1.0]
- XClone run combines RDR and BAF outputs via run_combine (not RDR or BAF alone) before assigning gain/loss labels

## Limitations

- Copy-loss AUC could only be compared between XClone and Numbat; InferCNV and CopyKAT do not use allelic information and cannot detect copy-neutral loss of heterozygosity
- Numbat may sacrifice single-cell resolution of clonal CNAs to a pseudo-bulk level, and its performance and robustness remain to be further assessed
- XClone is being updated and improved frequently; users should consult the tutorials and release history and keep the software up to date

## Evidence

- [intro] XClone accounts for two modules: BAF of heterozygous variants and RDR of individual genes combined into final CNA states: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [readme] README describes the three xcltk input matrices used to build XClone AnnData inputs: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [methods] Methods show the concrete RDR module invocation used in the benchmarking workflow: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [abstract] Abstract states the smoothing strategy that underlies XClone's superior AUC: "we present a statistical method, XClone, that strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs"
- [intro] Intro notes InferCNV and CopyKAT cannot detect copy-neutral LOH, constraining loss-detection comparisons: "both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
