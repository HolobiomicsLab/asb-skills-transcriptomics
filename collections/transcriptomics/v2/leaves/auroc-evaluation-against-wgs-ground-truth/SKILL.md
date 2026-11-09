---
name: auroc-evaluation-against-wgs-ground-truth
description: Use when comparing scRNA-seq CNA callers on a sample with WGS-derived
  clone labels (e.g., BCH869 glioma from Filbin et al. 2018) and you need gene-scale
  AUROC per CNA state (copy gain, copy loss, LOH).
when_to_use_negative:
- No WGS-derived clone annotations are available for the sample, so AUROCs cannot
  be computed against ground truth
- The method under evaluation is InferCNV or CopyKAT and the target CNA state is copy-neutral
  LOH — both tools ignore allelic information and cannot detect LOH
- Input data is droplet-based 10x rather than Smart-seq2 and the Smart-seq2-specific
  config (set_smartseq=True, concentration=35.5) would be misapplied without adjustment
edam_operation: http://edamontology.org/operation_3891
tools:
- name: XClone
  role: Primary CNA caller whose RDR, BAF, and Combine modules produce gene-scale
    CNA states to be scored
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit generating the allele-specific AD/DP matrices (BAF signals)
    and total read depth matrix (RDR signals) fed to XClone
  repo: https://github.com/hxj5/xcltk
- name: InferCNV
  role: Competing read-depth-based CNA method benchmarked on identical BCH869 data
- name: CopyKAT
  role: Competing read-depth-based CNA method benchmarked on identical BCH869 data
- name: CaSpER
  role: Competing allelic-shift-based CNA method benchmarked on identical BCH869 data
- name: Numbat
  role: Competing haplotype-phasing-based CNA method benchmarked on identical BCH869
    data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:63df33c4e9100b4779e804e51ef9e39a628dfddda08afb40a31a06936e28a950
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
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3891
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# AUROC evaluation against WGS ground truth

## Summary

Benchmark single-cell CNA detection methods (XClone vs InferCNV, CopyKAT, CaSpER, Numbat) by binarizing each method's per-gene CNA calls and computing gene-scale AUROCs for copy gain, copy loss, and LOH against WGS/scRNA-seq clone annotations as ground truth.

## When to use

Use when comparing scRNA-seq CNA callers on a sample with WGS-derived clone labels (e.g., BCH869 glioma from Filbin et al. 2018) and you need gene-scale AUROC per CNA state (copy gain, copy loss, LOH).

## When NOT to use

- No WGS-derived clone annotations are available for the sample, so AUROCs cannot be computed against ground truth
- The method under evaluation is InferCNV or CopyKAT and the target CNA state is copy-neutral LOH — both tools ignore allelic information and cannot detect LOH
- Input data is droplet-based 10x rather than Smart-seq2 and the Smart-seq2-specific config (set_smartseq=True, concentration=35.5) would be misapplied without adjustment

## Inputs

- BCH869 Smart-seq2 RDR and BAF anndata inputs (489 malignant cells, 3 non-tumour cells)
- WGS-derived clone annotation ground truth from Filbin et al. (2018)
- Per-gene CNA call matrices from InferCNV, CopyKAT, CaSpER, Numbat, and XClone

## Outputs

- Gene-scale AUROCs for copy gain, copy loss, and LOH per method
- bch869_benchmark_aurocs.csv

## How to apply

Run XClone's RDR module (XCloneConfig with module='RDR', set_smartseq=True, cell_anno_key='cell_type', ref_celltype='N', top_n_marker=25), then the BAF module (module='BAF', concentration=35.5, theo_neutral_BAF=0.5, phasing_len=100, CNV_N_components=5, BAF_denoise=True), and finally the Combine module (module='Combine', merge_loh=True, merge_loss=False, copygain_correct=False) on the BCH869 Smart-seq2 anndata inputs (489 malignant cells and 3 non-tumour cells). Run InferCNV, CopyKAT, CaSpER, and Numbat on the same expression/allelic data. Binarize each method's per-gene CNA calls at gene scale and compute AUROCs for copy gain, copy loss, and LOH against the WGS-derived clone annotation ground truth from Filbin et al. (2018). Expect XClone to outperform competitors for copy gain detection (AUROC 0.925 vs 0.853 for InferCNV and 0.845 for Numbat on BCH869).

## Related tools

- **XClone** (Primary CNA caller whose RDR, BAF, and Combine modules produce gene-scale CNA states to be scored) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit generating the allele-specific AD/DP matrices (BAF signals) and total read depth matrix (RDR signals) fed to XClone) — https://github.com/hxj5/xcltk
- **InferCNV** (Competing read-depth-based CNA method benchmarked on identical BCH869 data)
- **CopyKAT** (Competing read-depth-based CNA method benchmarked on identical BCH869 data)
- **CaSpER** (Competing allelic-shift-based CNA method benchmarked on identical BCH869 data)
- **Numbat** (Competing haplotype-phasing-based CNA method benchmarked on identical BCH869 data)

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig)
```

## Evaluation signals

- XClone copy-gain AUROC on BCH869 should be 0.925, exceeding InferCNV (0.853) and Numbat (0.845)
- Each method's per-gene CNA calls must be binarized at gene scale before AUROC computation so all methods are compared on a common scale
- AUROCs are computed separately for the three CNA states: copy gain, copy loss, and LOH
- Cell composition of the BCH869 anndata must be 489 malignant cells plus 3 non-tumour cells matching the Filbin et al. (2018) sample
- Results written to bch869_benchmark_aurocs.csv should include gene-scale AUROCs for all five methods

## Limitations

- InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral LOH, making LOH AUROCs uninformative for these two baselines
- HoneyBADGER and CaSpER's independent use of individual SNPs often suffers from high noise due to extremely low coverage of a single variant
- Numbat may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level
- Benchmarking was demonstrated on four cancer datasets and one simulated dataset; performance on other compositions is not established

## Evidence

- [results] On the BCH869 glioma dataset, XClone outperformed comparison methods for copy gain detection, achieving an AUROC of 0.925 versus 0.853 for InferCNV and 0.845 for Numbat.: [evidence span withheld — anchor status: not_found]
- [intro] However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH).: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [readme] XClone is an algorithm to infer allele- and haplotype-specific copy numbers in individual cells from low-coverage and sparse single-cell RNA sequencing data: "XClone is an algorithm to infer allele- and haplotype-specific copy numbers in individual cells from low-coverage and sparse single-cell RNA sequencing data"
- [intro] Its high accuracy will be demonstrated through comprehensive benchmarking with four cancer data sets and one simulated data set: "Its high accuracy will be demonstrated through comprehensive benchmarking with four cancer data sets and one simulated data set"
- [methods] RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig): "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
