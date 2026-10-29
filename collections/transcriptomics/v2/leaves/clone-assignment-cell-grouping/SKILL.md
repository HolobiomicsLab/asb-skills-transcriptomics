---
name: clone-assignment-cell-grouping
description: Use when single-cell RNA-seq data from a tumour sample must be partitioned
  into malignant subclones and normal cells based on copy number alterations, e.g.,
  before clone-stratified differential expression.
when_to_use_negative:
- Input data lacks heterozygous SNP coverage so BAF signals cannot be computed, and
  only read-depth-based CNA calling is feasible
- The task only needs expression clustering without copy-number-aware clone structure
  — a generic clustering tool would avoid the unnecessary CNA inference
- The dataset is a pseudo-bulk or bulk sample rather than single-cell data, where
  XClone's cell-level smoothing and KNN cell-cell graph do not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Runs the RDR and BAF modules and combines them to infer haplotype-specific
    CNA states and assign cells to clones (XClustering)
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the AD, DP, and total read depth matrices
    from SAM/BAM/CRAM files for the XClone BAF and RDR modules
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:a15c4cf18812fa8be843e99c813940cf1bc77827587341d741a1d862b3f4d3ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# clone-assignment-cell-grouping

## Summary

Assign individual cells to clonal subpopulations (e.g., normal, Clone 1, Clone 2 in the TNBC1 sample) by inferring allele- and haplotype-specific copy number states with XClone's combined RDR and BAF modules, storing labels under a Clone_ID cell-annotation key. Accurate clone grouping is the prerequisite for downstream per-clone analyses such as DEG enrichment in LOH regions.

## When to use

Use when single-cell RNA-seq data from a tumour sample must be partitioned into malignant subclones and normal cells based on copy number alterations, e.g., before clone-stratified differential expression.

## When NOT to use

- Input data lacks heterozygous SNP coverage so BAF signals cannot be computed, and only read-depth-based CNA calling is feasible
- The task only needs expression clustering without copy-number-aware clone structure — a generic clustering tool would avoid the unnecessary CNA inference
- The dataset is a pseudo-bulk or bulk sample rather than single-cell data, where XClone's cell-level smoothing and KNN cell-cell graph do not apply

## Inputs

- BAM/CRAM single-cell RNA-seq alignment files (10x Genomics droplet or SMART-seq)
- xcltk RDR module output (total read depth matrix)
- xcltk BAF module output (allele-specific AD and DP matrices)
- Genome feature annotations (e.g., hg38_genes, hg38_blocks, hg19_genes loaded via xclone.pp.load_anno)

## Outputs

- Per-cell haplotype-aware CNA state matrix (gene x cell) from combined XClone results
- Cell-level clone assignments stored under the Clone_ID cell-annotation key (e.g., normal, Clone 1, Clone 2)
- Annotated combined XClone AnnData object supporting downstream DEG/LOH stratification

## How to apply

Preprocess BAM/CRAM files with xcltk to generate the allele-specific AD and DP matrices (BAF signals) and the total read depth matrix (RDR signals), running the RDR and BAF modules independently. Run XClone's RDR module (xclone.model.run_RDR) and BAF module (xclone.model.run_BAF) with BAF_denoise enabled, then combine them to obtain per-cell, per-gene CNA states; the BAF module is essential because InferCNV and CopyKAT cannot detect copy-neutral LOH. Smooth CNA-state assignments horizontally with HMMs along genome coordinates and vertically with a KNN cell-cell graph to strengthen read depth and allelic imbalance signals, then group cells into clones via the Clone function XClustering and store assignments under the Clone_ID cell-annotation key. Validate by comparing inferred clone structure against the paper's results, e.g., widespread LOH in TNBC1 and loss detection in glioma BCH896.

## Related tools

- **XClone** (Runs the RDR and BAF modules and combines them to infer haplotype-specific CNA states and assign cells to clones (XClustering)) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the AD, DP, and total read depth matrices from SAM/BAM/CRAM files for the XClone BAF and RDR modules) — https://github.com/hxj5/xcltk

## Examples

```
conda activate xclone; pip install xclone xcltk; python -c "import xclone as xclone; RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig); BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig)"
```

## Evaluation signals

- Clone assignments reproduce the expected subclone structure reported for the sample (e.g., TNBC1 divided into normal, Clone 1, and Clone 2 cells with widespread LOH)
- The combined CNA call is supported by both modules — BAF loss signals confirm copy-neutral LOH events that RDR alone would miss
- Downstream sanity check: per-clone DEG enrichment in LOH regions matches reported values (Clone 1: 33.9% vs 21.4%, p = 1.1e-21; Clone 2: 27.8% vs 21.8%, p = 1.7e-3) only when Clone_ID labels are correct
- HMM smoothing along genome coordinates yields contiguous CNA segments rather than scattered per-gene noise
- Processed datasets match the reference outputs hosted in Rongtingting/xclone-data folders (e.g., TNBC1_scRNA, BCH869_scRNA)

## Limitations

- Numbat-style population haplotype-based approaches may sacrifice single-cell resolution to pseudo-bulk, and analogous aggregation choices in XClone trades resolution for robustness
- Results depend on preprocessing quality: cell filter limits, libratio capping, and cell quality checks in RDR extra_preprocess affect which cells are retained
- Software is under frequent update ('XClone is being updated and improved frequently at this stage'), so versions must be pinned (e.g., xclone0.3.6 with Python 3.9) for reproducibility

## Evidence

- [abstract] XClone strengthens read depth and allelic imbalance signals via graph smoothing: "strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs to detect haplotype-aware CNAs"
- [intro] Two-module design combining BAF and RDR: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [methods] Installation via xcltk and independent module runs: [evidence span withheld — anchor status: not_found]
- [methods] XClone run command in methods: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [readme] Processing data repository for results: [evidence span withheld — anchor status: external_doc]
