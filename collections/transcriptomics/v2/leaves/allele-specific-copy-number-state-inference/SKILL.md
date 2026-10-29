---
name: allele-specific-copy-number-state-inference
description: Use when per-cell copy gain/loss/LOH states must be inferred from raw
  scRNA-seq/snRNA-seq counts (10x or Smart-seq2), especially for tumour subclone dissection
  or when benchmarking XClone against InferCNV/CopyKAT/CaSpER/Numbat with AUROC.
when_to_use_negative:
- Input is already binarized per-gene CNA calls or a processed CNA feature table rather
  than raw counts — the skill operates on AD/DP and total read depth matrices from
  raw data
- Data are single-cell DNA-seq rather than scRNA-seq — methods like ACT5 or CHISEL
  are designed for single-cell DNA-seq CNA analysis
- Only expression data are available with no allelic/BAM support and copy-neutral
  LOH detection is not required — RDR-only callers such as InferCNV or CopyKAT suffice
tools:
- name: XClone
  role: 'Main statistical method: RDR module, BAF module with three-step haplotyping,
    and Combine module producing per-cell allele-specific CNA states'
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit generating the AD, DP, and total read depth matrices
    from SAM/BAM/CRAM files for XClone input
  repo: https://github.com/hxj5/xcltk
- name: Rongtingting/xclone-data
  role: Processed datasets (BCH869_scRNA, TNBC1_scRNA, ATC2_scRNA, GBM_10XsnRNA) and
    Jupyter notebook demos for reproducible runs
  repo: https://github.com/Rongtingting/xclone-data
- name: InferCNV
  role: Competing expression-only CNA baseline for copy gain/loss AUROC comparison
    (cannot detect copy-neutral LOH)
- name: CopyKAT
  role: Competing polynomial dynamic programming baseline for copy gain/loss AUROC
    comparison (cannot detect copy-neutral LOH)
- name: CaSpER
  role: Competing allelic-shift baseline for copy gain AUROC comparison in benchmarks
- name: Numbat
  role: Competing population haplotype-based phasing baseline for copy gain/loss AUROC
    comparison
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:b6f4f3022e499382f1ec950e560bc358257e38995efc732cc7a0e3e547ea50d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CaSpER
  - CopyKAT
  - InferCNV
  - Numbat
  - Rongtingting/xclone-data
  - XClone
  - scanpy
  - xcltk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# allele-specific-copy-number-state-inference

## Summary

Infer allele- and haplotype-specific copy number states (copy gain, copy loss, LOH) per individual cell from sparse scRNA-seq/snRNA-seq data using XClone's combined RDR (read depth ratio) and BAF (B-allele frequency) modules with HMM and KNN graph smoothing, and benchmark against InferCNV, CopyKAT, CaSpER, and Numbat.

## When to use

Use when per-cell copy gain/loss/LOH states must be inferred from raw scRNA-seq/snRNA-seq counts (10x or Smart-seq2), especially for tumour subclone dissection or when benchmarking XClone against InferCNV/CopyKAT/CaSpER/Numbat with AUROC.

## When NOT to use

- Input is already binarized per-gene CNA calls or a processed CNA feature table rather than raw counts — the skill operates on AD/DP and total read depth matrices from raw data
- Data are single-cell DNA-seq rather than scRNA-seq — methods like ACT5 or CHISEL are designed for single-cell DNA-seq CNA analysis
- Only expression data are available with no allelic/BAM support and copy-neutral LOH detection is not required — RDR-only callers such as InferCNV or CopyKAT suffice

## Inputs

- Cell-by-gene total read depth matrix from scRNA-seq (RDR signal)
- Allele-specific AD and DP matrices from heterozygous SNPs (BAF signals)
- AnnData object with cell annotations (e.g. cell_type with reference celltype 'N')
- Genome annotation loaded via xclone.pp.load_anno (genome_mode 'hg38_genes', 'hg38_blocks', 'hg19_genes')
- Ground-truth per-clone CNA annotations from WGS/single-cell DNA-seq (for benchmarking)

## Outputs

- RDR_Xdata from xclone.model.run_RDR
- BAF_merge_Xdata from xclone.model.run_BAF
- Per-cell, per-gene CNA state calls (copy gain, copy loss, LOH) from xclone.model.run_combine
- Clone/subclone assignments (XClustering)
- Benchmark AUROC table (e.g. bch869_benchmark_aurocs.csv, auc_roc_benchmark_astrocytoma_2R.csv)

## How to apply

First run xcltk preprocessing (RDR and BAF modules independently) to generate the three input matrices: the allele-specific AD and DP matrices and the total read depth matrix. Build AnnData inputs via xclone.pp.xclonedata with genome_mode annotation (e.g. hg38_genes, hg38_blocks) and add cell annotations with xclone.pp.extra_anno (cell_anno_key='cell_type', ref_celltype='N' for normal cells). Run the RDR module with XCloneConfig(module='RDR', set_smartseq=True for Smart-seq2 data), top_n_marker=25, trans_t=1e-6, start_prob=np.array([0.3, 0.4, 0.3]), then the BAF module with concentration=35.5 (bounds 20-100), theo_neutral_BAF=0.5, phasing_region_key='chr', phasing_len=100, WMA_window_size=6, trans_t=1e-6, CNV_N_components=5, BAF_denoise=True; merge with run_combine using merge_loh=True, merge_loss=False, copygain_correct=False. The rationale is that XClone strengthens read depth and allelic imbalance signals by smoothing on cell neighborhood (KNN graph) and gene coordinate (HMM along genome) graphs, enabling haplotype-aware CNA detection including copy-neutral LOH that RDR-only methods miss. Validate by scoring per-gene, per-cell CNA calls against WGS/scDNA-derived clone annotations (e.g. Filbin et al. 2018 BCH869, scOne-seq 2R astrocytoma clone 1) with gene-scale AUROCs for copy gain, copy loss, and LOH.

## Related tools

- **XClone** (Main statistical method: RDR module, BAF module with three-step haplotyping, and Combine module producing per-cell allele-specific CNA states) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit generating the AD, DP, and total read depth matrices from SAM/BAM/CRAM files for XClone input) — https://github.com/hxj5/xcltk
- **Rongtingting/xclone-data** (Processed datasets (BCH869_scRNA, TNBC1_scRNA, ATC2_scRNA, GBM_10XsnRNA) and Jupyter notebook demos for reproducible runs) — https://github.com/Rongtingting/xclone-data
- **InferCNV** (Competing expression-only CNA baseline for copy gain/loss AUROC comparison (cannot detect copy-neutral LOH))
- **CopyKAT** (Competing polynomial dynamic programming baseline for copy gain/loss AUROC comparison (cannot detect copy-neutral LOH))
- **CaSpER** (Competing allelic-shift baseline for copy gain AUROC comparison in benchmarks)
- **Numbat** (Competing population haplotype-based phasing baseline for copy gain/loss AUROC comparison)

## Examples

```
import xclone; xconfig = xclone.XCloneConfig(dataset_name='BCH869', module='RDR', set_smartseq=True); RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig)
```

## Evaluation signals

- Reproduce benchmark AUROCs on BCH869: XClone copy gain AUROC 0.925 vs InferCNV 0.853 and Numbat 0.845 (per-method AUROC within tolerance of published numbers)
- Reproduce 2R astrocytoma clone 1 AUCs: copy gain XClone 0.924 (CaSpER 0.827, Numbat 0.814, InferCNV 0.808, CopyKAT 0.798) and copy loss XClone 0.865 vs Numbat 0.619
- Confirm the three xcltk matrices (AD, DP, total read depth) are correctly generated before running RDR/BAF modules
- Check config parameters match: trans_t=1e-6, top_n_marker=25, concentration=35.5, theo_neutral_BAF=0.5, phasing_len=100, CNV_N_components=5, BAF_denoise=True, merge_loh=True
- Binarized per-gene calls scored against WGS/scOne-seq ground-truth clone annotations yield AUROCs in the plausible 0.6-0.95 range per method

## Limitations

- InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH) — comparisons limited to gain/loss AUROCs for those methods
- Numbat may sacrifice the resolution of clonal CNAs from a single-cell level to a pseudo-bulk level, and its performance and robustness remain to be further assessed
- HoneyBADGER and CaSpER's independent use of individual SNPs suffers from high noise due to extremely low coverage of a single variant
- XClone is being updated and improved frequently; Python 3.9 is recommended for stable performance in the latest version (see xclone0.3.6Python3.9_env.yml)

## Evidence

- [abstract] we present a statistical method, XClone, that strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs: "we present a statistical method, XClone, that strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs"
- [intro] this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [intro] both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH): "both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [methods] RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig): "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [methods] xconfig.BAF_denoise = True: "xconfig.BAF_denoise = True"
- [readme] It takes three matrices as input: the allele-specific *AD* and *DP* matrices (BAF signals) and the *total read depth* matrix (RDR signals).: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
