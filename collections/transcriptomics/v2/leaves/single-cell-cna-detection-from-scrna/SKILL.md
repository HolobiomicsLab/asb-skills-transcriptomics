---
name: single-cell-cna-detection-from-scrna
description: Use when scRNA-seq data (10x Genomics droplet-based or Smart-seq2) from
  a tumour sample must be profiled for allele-specific CNAs (copy gain, copy loss,
  copy-neutral LOH) and subclones reconstructed from sparse, low-coverage data.
when_to_use_negative:
- The input is single-cell DNA-seq rather than scRNA-seq — tools such as ACT or CHISEL
  are designed for that modality instead.
- 'Only copy-number magnitude is needed and copy-neutral LOH detection is out of scope:
  InferCNV or CopyKAT, which ignore allelic information, may suffice.'
- High-coverage data make per-SNP allelic signals reliable enough without aggregation;
  the smoothing/haplotyping machinery is aimed at extremely low per-variant coverage.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
tools:
- name: XClone
  role: 'Main statistical model: runs RDR, BAF, and Combine modules to infer haplotype-specific
    CNAs and reconstruct clones from scRNA-seq counts'
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the AD, DP, and total read-depth matrices
    from SAM/BAM/CRAM files for XClone input
  repo: https://github.com/hxj5/xcltk
- name: CopyKAT
  role: Baseline comparator providing reference clone annotations (Clone_ID) against
    which XClone clone labels are evaluated; cannot detect copy-neutral LOH
- name: InferCNV
  role: Baseline method for CNA detection benchmarked against XClone at gene scale;
    does not consider allelic information
- name: CaSpER
  role: Baseline method using allelic shift signal; its per-SNP use suffers high noise
    at low coverage
- name: Numbat
  role: Baseline method using population haplotype-based phasing, benchmarked against
    XClone for copy gain detection
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:f7ddc0503db973d9d5ae3b25fc0976579efe5ebde94214dac51c7cd2fd40e68c
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# single-cell-cna-detection-from-scrna

## Summary

Detect allele-specific copy number alterations (CNAs) and reconstruct tumour clonal substructure from sparse single-cell RNA-seq data by combining read-depth ratio (RDR) and B-allele frequency (BAF) modules in XClone, with HMM smoothing along genome coordinates and KNN smoothing across the cell-neighborhood graph.

## When to use

Use when scRNA-seq data (10x Genomics droplet-based or Smart-seq2) from a tumour sample must be profiled for allele-specific CNAs (copy gain, copy loss, copy-neutral LOH) and subclones reconstructed from sparse, low-coverage data.

## When NOT to use

- The input is single-cell DNA-seq rather than scRNA-seq — tools such as ACT or CHISEL are designed for that modality instead.
- Only copy-number magnitude is needed and copy-neutral LOH detection is out of scope: InferCNV or CopyKAT, which ignore allelic information, may suffice.
- High-coverage data make per-SNP allelic signals reliable enough without aggregation; the smoothing/haplotyping machinery is aimed at extremely low per-variant coverage.

## Inputs

- BAM/CRAM alignments from 10x Genomics droplet-based or Smart-seq2 scRNA-seq of a tumour sample
- xcltk outputs: allele-specific AD and DP matrices (BAF) and total read-depth matrix (RDR) as AnnData objects
- Reference annotations loaded via xclone.pp.load_anno(genome_mode='hg38_genes'/'hg38_blocks'/'hg19_genes')
- Cell type annotations under a cell_anno_key (e.g. 'cell_type') identifying a non-malignant reference cell type (e.g. 'N')

## Outputs

- Per-cell × per-gene CNA-state probability tensor from run_combine
- Per-gene CNA calls for copy gain, copy loss, and LOH states
- Per-cell clone labels (e.g. XClone_ID) via K-means on the flattened CNA probability matrix
- Per-cell table of clone assignments for downstream comparison (e.g. tnbc1_xclone_clone_labels.csv)

## How to apply

Run the xcltk preprocessing pipeline on the aligned SAM/BAM/CRAM files to produce three matrices: allele-specific AD and DP counts (BAF signals) and a total read-depth matrix (RDR signals). Run the XClone RDR module with xclone.XCloneConfig(module='RDR', cell_anno_key='cell_type', ref_celltype='N', top_n_marker=25, trans_t=1e-6, start_prob=[0.3, 0.4, 0.3]) — for Smart-seq2 data also set set_smartseq=True — then run_BAF with concentration=35.5 (bounds 20–100), theo_neutral_BAF=0.5, phasing_region_key='chr', phasing_len=100, WMA_window_size=6, CNV_N_components=5, and BAF_denoise=True. The BAF module performs three-step haplotyping (SNP→gene by population-based phasing, gene→100-gene bin by EM, bin→chromosome arm by dynamic programming). Fuse RDR and BAF Xdata with xclone.model.run_combine using merge_loh=True and copygain_correct=False to obtain the per-cell × per-gene CNA-state probability tensor, then assign per-cell clone labels by flattening the probability tensor and K-means clustering via XClone's XClustering function (e.g. storing labels as XClone_ID). The rationale is that effective smoothing on cell-neighborhood and gene coordinate graphs strengthens weak RDR and allelic-imbalance signals that individual-SNP methods miss.

## Related tools

- **XClone** (Main statistical model: runs RDR, BAF, and Combine modules to infer haplotype-specific CNAs and reconstruct clones from scRNA-seq counts) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the AD, DP, and total read-depth matrices from SAM/BAM/CRAM files for XClone input) — https://github.com/hxj5/xcltk
- **CopyKAT** (Baseline comparator providing reference clone annotations (Clone_ID) against which XClone clone labels are evaluated; cannot detect copy-neutral LOH)
- **InferCNV** (Baseline method for CNA detection benchmarked against XClone at gene scale; does not consider allelic information)
- **CaSpER** (Baseline method using allelic shift signal; its per-SNP use suffers high noise at low coverage)
- **Numbat** (Baseline method using population haplotype-based phasing, benchmarked against XClone for copy gain detection)

## Examples

```
conda create -n xclone python=3.9 && pip install xclone; RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xclone.XCloneConfig(module='RDR', cell_anno_key='cell_type', ref_celltype='N', trans_t=1e-6))
```

## Evaluation signals

- Clonal reconstruction concordance: K-means labels (XClone_ID) vs. an independent Clone_ID annotation should reach near-perfect agreement (target ARI ≈ 0.965 on TNBC1)
- Gene-scale AUROC against WGS-derived ground truth (e.g. Filbin et al. 2018 clone annotations for BCH869): copy gain AUROC ≥ 0.925, exceeding InferCNV (0.853) and Numbat (0.845)
- Output schema check: run_combine returns a combined per-cell × per-gene CNA-state probability tensor derived from both RDR_Xdata and BAF_merge_Xdata
- Loss of heterozygosity is detected (e.g. widespread LOH in TNBC1, allele-specific loss in glioma BCH896) — a state InferCNV/CopyKAT cannot produce
- Per-cell output table contains cell barcode, XClone clone label, and reference annotation for each cell before any ARI or AUROC computation

## Limitations

- XClone requires a reference non-malignant cell type within the sample (ref_celltype, e.g. 'N') for RDR normalization; samples without such cells are not covered by the demonstrated workflows.
- Allelic analysis needs heterozygous SNPs: the BAF module depends on phasing individual SNPs to genes, 100-gene bins, and chromosome arms, and HoneyBADGER/CaSpER-style per-SNP approaches suffer high noise at extremely low single-variant coverage — signals must be aggregated.
- Sparse or low-coverage scRNA-seq inputs make RDR and BAF individually noisy; results rely on the smoothing strategy (HMM along genome coordinates plus KNN cell-cell graph), so heavily filtered or very small cell numbers may degrade performance.

## Evidence

- [intro] two-module design: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [intro] smoothing strategy: "horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a"
- [intro] three-step haplotyping: "we implemented a three-step haplotyping, from individual SNPs to a gene by population-based phasing, from consecutive genes (100 by default) to a gene bin by an Expectation-Maximization algorithm"
- [readme] xcltk purpose: [evidence span withheld — anchor status: external_doc]
- [methods] RDR module call: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [abstract] XClone smoothing rationale: "strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs"
