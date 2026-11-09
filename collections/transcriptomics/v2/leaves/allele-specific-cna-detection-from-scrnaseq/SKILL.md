---
name: allele-specific-cna-detection-from-scrnaseq
description: Use when the goal is to infer haplotype-aware CNA states or tumor subclones
  from single-cell RNA-seq count data (e.g., 10x Genomics droplet or Smart-seq2 tumor
  samples), especially when copy-neutral LOH or low tumor purity challenges expression-only
  methods.
when_to_use_negative:
- The task only requires expression-level CNA inference and no allelic/LOH information
  is needed — in that case InferCNV or CopyKAT (which do not model allelic information)
  may suffice
- The input is bulk DNA-seq or single-cell DNA-seq data, for which tools like ACT5
  or CHISEL are more appropriate
- Only pseudo-bulk clonal resolution is acceptable and single-cell-level CNA resolution
  is not required (Numbat-style aggregation)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
tools:
- name: XClone
  role: 'Main statistical model: runs RDR and BAF modules, combines them, and clusters
    subclones'
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit that generates AD/DP and read-depth matrices from SAM/BAM/CRAM
    files
  repo: https://github.com/hxj5/xcltk
- name: xclone-data
  role: Repository of processed datasets and Jupyter notebook examples for reproducible
    results
  repo: https://github.com/Rongtingting/xclone-data
- name: InferCNV
  role: Baseline expression-based CNA method used for ARI comparison of subclone labels
- name: CopyKAT
  role: Baseline polynomial dynamic-programming CNA method used for ARI comparison
    of subclone labels
- name: scanpy
  role: Single-cell analysis utilities supporting the AnnData-based workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:a53aa780978e6640c942882c6e8b0d4f6e617c90746f6b7fc8f0297fa08c8b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - CopyKAT
  - InferCNV
  - XClone
  - scanpy
  - xclone-data
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# allele-specific-cna-detection-from-scrnaseq

## Summary

Detect allele- and haplotype-specific copy number alterations in individual cells from low-coverage, sparse scRNA-seq data using XClone, which combines a B-allele frequency (BAF) module and a read-depth ratio (RDR) module, then resolve tumor subclones by clustering the combined CNA states.

## When to use

Use when the goal is to infer haplotype-aware CNA states or tumor subclones from single-cell RNA-seq count data (e.g., 10x Genomics droplet or Smart-seq2 tumor samples), especially when copy-neutral LOH or low tumor purity challenges expression-only methods.

## When NOT to use

- The task only requires expression-level CNA inference and no allelic/LOH information is needed — in that case InferCNV or CopyKAT (which do not model allelic information) may suffice
- The input is bulk DNA-seq or single-cell DNA-seq data, for which tools like ACT5 or CHISEL are more appropriate
- Only pseudo-bulk clonal resolution is acceptable and single-cell-level CNA resolution is not required (Numbat-style aggregation)

## Inputs

- BAM/CRAM files from droplet-based (10x Genomics) or well-based (SMART-seq2) scRNA-seq tumor samples
- xcltk-generated matrices: allele-specific AD and DP counts (BAF) and total read depth matrix (RDR)
- Cell annotations (cell_anno_key) and normal-cell reference labels (ref_celltype)
- Gene and genome-block annotations via genome_mode (e.g., 'hg38_genes', 'hg38_blocks')

## Outputs

- Per-cell allele-specific CNA states (haplotype-aware, from run_combine)
- Subclone labels from XClustering on combined CNA profiles
- Agreement metrics against other methods, e.g., adjusted_rand_score versus InferCNV/CopyKAT labels (reported ARI ~0.998 vs InferCNV, ~0.961 vs CopyKAT on ATC2)

## How to apply

Preprocess BAM/CRAM files with the xcltk pipeline to generate three matrices: allele-specific AD/DP (BAF signals) and total read depth (RDR signals). Build AnnData objects with xclone.pp.xclonedata (data_mode 'RDR' and 'BAF', genome_mode 'hg38_genes') and attach cell annotations via cell_anno_key and ref_celltype. Run the RDR module with xclone.model.run_RDR and XCloneConfig(module='RDR') (e.g., trans_t = 1e-6, start_prob = np.array([0.3, 0.4, 0.3])), then the BAF module with xclone.model.run_BAF enabling BAF_denoise = True and the 5-state allele-bias transition matrix with trans_t = 1e-6. Merge modules with xclone.model.run_combine to obtain haplotype-aware CNA states per cell, and assign subclones with the XClustering function on the combined profiles. CNA-state assignments are smoothed horizontally with hidden Markov models along genome coordinates and vertically with a k-nearest-neighbors cell-cell connectivity graph, which is what makes the method robust to sparsity.

## Related tools

- **XClone** (Main statistical model: runs RDR and BAF modules, combines them, and clusters subclones) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit that generates AD/DP and read-depth matrices from SAM/BAM/CRAM files) — https://github.com/hxj5/xcltk
- **xclone-data** (Repository of processed datasets and Jupyter notebook examples for reproducible results) — https://github.com/Rongtingting/xclone-data
- **InferCNV** (Baseline expression-based CNA method used for ARI comparison of subclone labels)
- **CopyKAT** (Baseline polynomial dynamic-programming CNA method used for ARI comparison of subclone labels)
- **scanpy** (Single-cell analysis utilities supporting the AnnData-based workflow)

## Examples

```
RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig); BAF_merge_Xdata = xclone.model.run_BAF(BAF_adata, config_file = xconfig)
```

## Evaluation signals

- ARI between XClone subclone labels and precomputed InferCNV/CopyKAT clone labels reproduces ~0.998 (vs InferCNV) and ~0.961 (vs CopyKAT) on the ATC2 sample
- Clustering resolves the two tumor subclones even at ~2% tumor purity, with concordant CNA calls (gains on chr5 in subclone 2, chr7, chr19; loss on chr6p)
- Three output matrices from xcltk (AD, DP, total read depth) are present and consistent in cell barcodes before running the XClone modules
- The BAF module runs with BAF_denoise = True and the 5-state allele-bias transition matrix, and run_combine produces merged haplotype-aware states per cell
- Results can be compared against the published Jupyter notebooks in the xclone-data examples folder (e.g., ATC2_XClone_demo.ipynb)

## Limitations

- InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH); XClone is chosen when LOH detection matters
- Independent use of individual SNPs (as in HoneyBADGER/CaSpER) suffers high noise from extremely low single-variant coverage; XClone addresses this via smoothing on cell neighborhood and gene coordinate graphs
- XClone is under active development; the README advises frequently checking the tutorials and release history and keeping the software up to date
- Python version matters: XClone requires Python 3.7 or >=3.9, with 3.9 recommended for stable performance

## Evidence

- [abstract] we present a statistical method, XClone, that strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs: "we present a statistical method, XClone, that strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs"
- [intro] both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH): "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [methods] xconfig.BAF_denoise = True: "xconfig.BAF_denoise = True"
- [methods] RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig): "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [readme] The xcltk package implements a preprocessing pipeline to generate the three matrices from SAM/BAM/CRAM files.: [evidence span withheld — anchor status: external_doc]
- [results] gains on chr5 in subclone 2, chr7, chr19 and loss on chr6p: [evidence span withheld — anchor status: not_found]
