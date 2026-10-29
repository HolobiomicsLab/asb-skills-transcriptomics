---
name: negative-binomial-glm-copy-number-fitting
description: Use when raw scRNA-seq UMI/read counts must be transformed into copy-number
  states (loss/neutral/gain) per gene and cell via negative-binomial statistical modeling
  with HMM smoothing.
when_to_use_negative:
- The goal is copy-neutral LOH or allele-specific calls, which the RDR read-depth
  module alone cannot resolve — the BAF module and combine step are required.
- Input is already a CNA state matrix or per-gene-per-cell copy-number call table
  rather than raw read/UMI counts.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0622
tools:
- name: XClone
  role: 'implements the RDR module: negative-binomial GLM fitting with KNN and HMM
    smoothing via xclone.model.run_RDR'
  repo: https://github.com/single-cell-genetics/XClone
- name: statsmodels
  role: used to fit the negative-binomial GLM on normalized expression counts
- name: xcltk
  role: preprocessing toolkit that generates the total read depth (RDR signal) matrix
    from SAM/BAM/CRAM files
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:37e5ccf5c3910a36538bd613766cb1fff383afefe4d647c5fadb859b4f6d6ab2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - statsmodels
  - xcltk
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# negative-binomial-glm-copy-number-fitting

## Summary

Fit a negative-binomial GLM to the gene-by-cell expression read-count matrix of XClone's RDR module, estimating library size, dispersions, and copy loss/neutral/gain ratios to convert read depth into per-gene, per-cell CNA-state probabilities.

## When to use

Use when raw scRNA-seq UMI/read counts must be transformed into copy-number states (loss/neutral/gain) per gene and cell via negative-binomial statistical modeling with HMM smoothing.

## When NOT to use

- The goal is copy-neutral LOH or allele-specific calls, which the RDR read-depth module alone cannot resolve — the BAF module and combine step are required.
- Input is already a CNA state matrix or per-gene-per-cell copy-number call table rather than raw read/UMI counts.

## Inputs

- gene-by-cell expression read-count matrix (AnnData, e.g. from TNBC1 droplet-based scRNA-seq or BCH869 smart-seq2 data)
- hg38 gene annotation loaded via xclone.pp.load_anno(genome_mode='hg38_genes')
- XCloneConfig for the RDR module (ref_celltype, top_n_marker, trans_t, start_prob)

## Outputs

- genes-by-cells-by-states probability tensor detecting gain or loss states per gene and cell
- RDR_Xdata AnnData with per-gene-per-cell CNA state calls and smoothed probability layers

## How to apply

Load the gene-by-cell expression matrix (e.g., TNBC1 demo data via xclone.data.tnbc1_rdr() with hg38 gene annotation from xclone.pp.load_anno(genome_mode='hg38_genes')). Configure the RDR module with xclone.XCloneConfig(module='RDR') setting ref_celltype='N', top_n_marker=25, trans_t=1e-6 and start_prob=np.array([0.3, 0.4, 0.3]), then run xclone.model.run_RDR(RDR_adata, config_file=xconfig). Within the run, normalize counts to the reference cell type, apply gene filtering with library-ratio capping, and fit the negative-binomial GLM (statsmodels) with heuristic estimation of library size, dispersions, and copy loss/neutral/gain ratios. Smooth the resulting CNA-state probabilities along a KNN cell-connectivity graph (run_KNN=True) followed by a hidden Markov model along genome coordinates using the configured transition matrix, producing a genes-by-cells-by-states probability tensor that calls gain or loss per gene and cell.

## Related tools

- **XClone** (implements the RDR module: negative-binomial GLM fitting with KNN and HMM smoothing via xclone.model.run_RDR) — https://github.com/single-cell-genetics/XClone
- **statsmodels** (used to fit the negative-binomial GLM on normalized expression counts)
- **xcltk** (preprocessing toolkit that generates the total read depth (RDR signal) matrix from SAM/BAM/CRAM files) — https://github.com/hxj5/xcltk

## Examples

```
xconfig = xclone.XCloneConfig(module='RDR'); xconfig.trans_t = 1e-6; RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file=xconfig)
```

## Evaluation signals

- RDR_Xdata output exists and contains per-gene-per-cell CNA state calls with smoothed probability layers
- Probability tensor has genes-by-cells-by-states shape covering loss, neutral, and gain states
- Gene filtering with library-ratio capping removed low-quality genes/cells before GLM fitting
- Downstream combine step (run_combine with merge_loh=True) yields four-state CNA calls on the merged RDR+BAF result

## Limitations

- RDR alone detects read-depth-based CNAs only; InferCNV-like approaches that ignore allelic information cannot detect copy-neutral LOH
- Software is updated frequently, so XClone release/version should be kept current (Python 3.9 environment recommended for stable performance)
- Requires reference non-malignant cells (ref_celltype, e.g. 'N' in the TNBC1 demo) for count normalization

## Evidence

- [methods] The RDR module subjects the gene-by-cell expression matrix to negative-binomial statistical modeling paired with HMM iterations: [evidence span withheld — anchor status: not_found]
- [methods] RDR config and run command from the article workflow: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
- [intro] smoothing strategy on cell and genome graphs: "we employed two orthogonal strategies to smooth the CNA-state assignments on both BAF and RDR modules: horizontally with hidden Markov models (HMM) along the genome coordinates and vertically with a"
- [readme] install RDR XClone package via pip: "pip install git+https://github.com/single-cell-genetics/XClone"
- [readme] library-ratio capping filter update: "- update libratio capping and add cell filter limits"
