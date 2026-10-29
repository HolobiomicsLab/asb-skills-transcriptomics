---
name: smart-seq2-rdr-baf-anndata-loading
description: Use when input is Smart-seq2 scRNA-seq data for the BCH869 glioma sample
  (489 malignant + 3 non-tumour cells) already processed by xcltk into RDR and BAF
  matrices that must be loaded as anndata for XClone.
when_to_use_negative:
- Input is raw FASTQ/BAM data that has not been preprocessed by the xcltk RDR and
  BAF modules into AD/DP and read-depth matrices
- Data are droplet-based (10x) rather than Smart-seq2 — the set_smartseq=True configuration
  and loading assumptions would not match
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: XClone
  role: Provides xclone.data loaders and xclone.pp.xclonedata to load Smart-seq2 RDR/BAF
    inputs as anndata and attach genome_mode feature annotations
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Upstream preprocessing toolkit that generates the three input matrices (allele-specific
    AD and DP, and total read depth) from BAM/CRAM files before loading
  repo: https://github.com/hxj5/xcltk
- name: xclone-data
  role: Hosts the processed BCH869_scRNA dataset folder and example Jupyter notebooks
    demonstrating the loading step
  repo: https://github.com/Rongtingting/xclone-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:68dbab9bf44bc6f9a3edd2632f18c497f2656300679bc56b64183f0072cedd28
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
  - xclone-data
  - xcltk
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Smart-seq2 RDR/BAF anndata loading for XClone

## Summary

Loads the Smart-seq2 glioma (BCH869) read-depth-ratio and B-allele-frequency matrices as anndata objects, with hg19/hg38 gene and genome-block annotations, so they can be passed into XClone's RDR and BAF modules.

## When to use

Use when input is Smart-seq2 scRNA-seq data for the BCH869 glioma sample (489 malignant + 3 non-tumour cells) already processed by xcltk into RDR and BAF matrices that must be loaded as anndata for XClone.

## When NOT to use

- Input is raw FASTQ/BAM data that has not been preprocessed by the xcltk RDR and BAF modules into AD/DP and read-depth matrices
- Data are droplet-based (10x) rather than Smart-seq2 — the set_smartseq=True configuration and loading assumptions would not match

## Inputs

- xcltk-generated RDR matrix (total read depth) for BCH869 Smart-seq2 scRNA-seq
- xcltk-generated allele-specific AD/DP matrices for BAF signals
- Cell type annotations (cell_anno_key='cell_type'; 'N' reference cells)
- Genome feature annotations via genome_mode ('hg19_genes', 'hg38_genes', 'hg38_blocks')

## Outputs

- RDR_adata anndata object ready for xclone.model.run_RDR
- BAF_adata anndata object ready for xclone.model.run_BAF
- Feature-annotated XClone input with hg19/hg38 gene and genome-block coordinates

## How to apply

After running the xcltk RDR and BAF preprocessing modules independently, load the resulting matrices with the XClone data loaders (e.g. xclone.data.bch869 or xclone.pp.xclonedata) into anndata objects. Specify the genome_mode (e.g. 'hg19_genes', 'hg38_genes' or 'hg38_blocks') so XClone attaches the default feature annotation to the expression/allelic features. Confirm the BCH869 anndata carries the cell_anno_key ('cell_type') distinguishing malignant cells from the 'N' (normal) reference cells before passing RDR_adata and BAF_adata to xclone.model.run_RDR and run_BAF. The Smart-seq2 flag (set_smartseq=True in XCloneConfig) must be set because read-depth characteristics differ from 10x droplet data.

## Related tools

- **XClone** (Provides xclone.data loaders and xclone.pp.xclonedata to load Smart-seq2 RDR/BAF inputs as anndata and attach genome_mode feature annotations) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Upstream preprocessing toolkit that generates the three input matrices (allele-specific AD and DP, and total read depth) from BAM/CRAM files before loading) — https://github.com/hxj5/xcltk
- **xclone-data** (Hosts the processed BCH869_scRNA dataset folder and example Jupyter notebooks demonstrating the loading step) — https://github.com/Rongtingting/xclone-data

## Examples

```
hg38_genes = xclone.pp.load_anno(genome_mode = "hg38_genes")
```

## Evaluation signals

- The loaded RDR anndata contains 492 cells (489 malignant + 3 non-tumour) from the BCH869 sample
- Both RDR and BAF anndata carry gene feature coordinates matching the genome_mode annotation (hg19_genes/hg38_genes) and a cell_type annotation including 'N' reference cells
- The BAF anndata exposes AD and DP layers suitable for phasing_region_key='chr' phasing in the downstream BAF module
- Setting set_smartseq=True in XCloneConfig and running run_RDR completes without dimension/annotation errors, confirming correct loading

## Limitations

- BCH869 has only 3 non-tumour cells as reference, which limits reference-based RDR/BAF normalization stability
- The processed-data repository (xclone-data) is explicitly preliminary, with 'additional information and description will be available soon'
- Annotation loading depends on the bundled genome_mode choices ('hg19_genes', 'hg38_genes', 'hg38_blocks'); other genomes need custom annotations

## Evidence

- [methods] BCH869 data: "`BCH869 data`_: a glioma sample BCH869 with histone H3 lysine27-to-methionine mutations (H3K27M-glioma), where 489 malignant cells and 3 non-tumour cells were probed by smart-seq2"
- [methods] genome_mode annotations: "use default feature annotation after you specify the genome_mode (include: "hg19_genes","
- [methods] xclone data loaders in workflow: "Step1: :ref:`Install xcltk <xcltk installation>` and run xctlkt :ref:`RDR module <xcltk RDR>` and :ref:`BAF module <xcltk BAF>`, independently."
- [readme] XClone input scope: "XClone is an algorithm to infer allele- and haplotype-specific copy numbers in individual cells from low-coverage and sparse single-cell RNA sequencing data"
- [readme] Smart-seq platform support: [evidence span withheld — anchor status: external_doc]
