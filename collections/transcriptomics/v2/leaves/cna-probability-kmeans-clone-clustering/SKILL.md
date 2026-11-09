---
name: cna-probability-kmeans-clone-clustering
description: Use when scRNA-seq (10x Genomics or Smart-seq2) of a tumour sample has
  been processed by XClone's RDR+BAF+Combine modules and per-cell clone labels are
  needed from the resulting CNA probability tensor.
when_to_use_negative:
- Input data has not been processed by XClone's RDR and BAF modules — clustering raw
  expression or raw UMI counts is not the intended use
- You need copy-neutral LOH resolution from methods that only use read depth — InferCNV
  and CopyKAT do not consider allelic information and cannot detect copy-neutral LOH
- Only single-variant BAF signals are available without XClone's smoothing — independent
  use of individual SNPs suffers from high noise due to extremely low per-variant
  coverage
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: XClone
  role: Produces the RDR/BAF/Combine CNA-state probability tensor and provides the
    XClustering analysis function used for K-means clone assignment
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates the allele-specific AD/DP matrices (BAF)
    and total read depth matrix (RDR) from SAM/BAM/CRAM files
  repo: https://github.com/hxj5/xcltk
- name: CopyKAT
  role: Provides the independent Clone_ID annotation used as the reference for ARI
    concordance evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:833f01569576c804fbd335da6e603555c1502e7139b5362f7962c1029b49cf9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CopyKAT
  - XClone
  - xcltk
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cna-probability-kmeans-clone-clustering

## Summary

Reconstruct tumour subclonal structure from scRNA-seq by flattening XClone's per-cell x per-gene CNA-state probability tensor and assigning per-cell clone labels with K-means via XClone's XClustering analysis function. This was validated on the TNBC1 triple-negative breast cancer sample, where the reconstruction was concordant with CopyKAT-derived clone annotations (ARI 0.965).

## When to use

Use when scRNA-seq (10x Genomics or Smart-seq2) of a tumour sample has been processed by XClone's RDR+BAF+Combine modules and per-cell clone labels are needed from the resulting CNA probability tensor.

## When NOT to use

- Input data has not been processed by XClone's RDR and BAF modules — clustering raw expression or raw UMI counts is not the intended use
- You need copy-neutral LOH resolution from methods that only use read depth — InferCNV and CopyKAT do not consider allelic information and cannot detect copy-neutral LOH
- Only single-variant BAF signals are available without XClone's smoothing — independent use of individual SNPs suffers from high noise due to extremely low per-variant coverage

## Inputs

- xcltk RDR module matrix (RDR_adata, AnnData) from TNBC1 10x Genomics droplet-based scRNA-seq (GEO GSE148673) or xclone.data.tnbc1_rdr()
- xcltk BAF module matrices (BAF_adata, AnnData; allele-specific AD/DP) or xclone.data.tnbc1_baf()
- XClone combined CNA-state probability tensor from xclone.model.run_combine
- Reference per-cell clone annotation (CopyKAT-derived Clone_ID) for concordance evaluation

## Outputs

- Per-cell clone labels (XClone_ID) from K-means clustering of the flattened CNA probability matrix
- Adjusted Rand index (ARI) quantifying concordance with CopyKAT Clone_ID (target 0.965 for TNBC1)
- Per-cell table file tnbc1_xclone_clone_labels.csv (cell barcode, XClone_ID, CopyKAT Clone_ID) with ARI value

## How to apply

Run XClone's three modules on the TNBC1-style inputs: RDR via xclone.model.run_RDR with cell_anno_key='cell_type', ref_celltype='N', top_n_marker=25, trans_t=1e-6 and start_prob=[0.3, 0.4, 0.3]; BAF via xclone.model.run_BAF with concentration=35.5 (bounds 20/100), theo_neutral_BAF=0.5, phasing_region_key='chr', phasing_len=100, WMA_window_size=6, CNV_N_components=5, trans_t=1e-6 and BAF_denoise=True; then fuse with xclone.model.run_combine using merge_loh=True and copygain_correct=False to obtain the combined CNA-state probability tensor. Flatten this per-cell x per-gene probability tensor into a feature matrix and apply K-means through XClone's XClustering analysis function, storing per-cell labels as XClone_ID. Judge success by computing the adjusted Rand index (ARI) of XClone_ID against the CopyKAT-derived Clone_ID annotation; the TNBC1 target agreement is ARI 0.965. Write the per-cell table (cell barcode, XClone_ID, CopyKAT Clone_ID) with the ARI value to tnbc1_xclone_clone_labels.csv.

## Related tools

- **XClone** (Produces the RDR/BAF/Combine CNA-state probability tensor and provides the XClustering analysis function used for K-means clone assignment) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates the allele-specific AD/DP matrices (BAF) and total read depth matrix (RDR) from SAM/BAM/CRAM files) — https://github.com/hxj5/xcltk
- **CopyKAT** (Provides the independent Clone_ID annotation used as the reference for ARI concordance evaluation)

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig); BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file=xconfig); combined = xclone.model.run_combine(RDR_Xdata, BAF_merge_Xdata, config_file=xconfig); labels = xclone.pl.XClustering(combined, flatten=True)
```

## Evaluation signals

- Adjusted Rand index between XClone_ID and CopyKAT Clone_ID reaches ~0.965 on TNBC1
- Output CSV tnbc1_xclone_clone_labels.csv contains one row per cell barcode with non-null XClone_ID and CopyKAT Clone_ID columns plus the ARI value
- The flattened probability matrix fed to XClustering has one row per cell and one column per gene in the combined CNA tensor
- Each cell receives exactly one K-means clone label; label count is finite and small (clone-level, not per-cell-level)
- All three XClone modules completed with the specified config (trans_t=1e-6, BAF_denoise=True, merge_loh=True) before clustering

## Limitations

- Concordance benchmark (ARI 0.965) is specific to the TNBC1 sample; other datasets (BCH869, ATC2, GBM_10XsnRNA) may show different agreement with reference annotations
- XClone is updated and improved frequently; results may vary across versions — the stable environment is xclone 0.3.6 on Python 3.9 (xclone0.3.6Python3.9_env.yml)
- CopyKAT-derived Clone_ID is used as the reference, but CopyKAT itself cannot detect copy-neutral LOH, so agreement does not capture XClone's allele-specific advantages

## Evidence

- [results] K-means clustering on XClone's flattened CNA probability matrix: [evidence span withheld — anchor status: not_found]
- [methods] Run the RDR and BAF modules then combine: [evidence span withheld — anchor status: not_found]
- [intro] Two-module design of XClone: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [methods] TNBC1 dataset description: "`TNBC1 data`_: a triple-negative breast cancer (TNBC) sample that was assayed by droplet-based scRNA-seq (10x Genomics), from `Gao et al. (2021)`_."
- [readme] XClone install instructions: "XClone package can be conveniently (1~2mins) installed via PyPI::

    pip install xclone"
