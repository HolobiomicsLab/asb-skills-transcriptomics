---
name: allelic-count-matrix-generation-from-bam
description: Use when starting from cell-barcoded BAM/CRAM files from a scRNA-seq
  experiment (e.g., 10x Genomics) and you need allele-specific AD/DP matrices to feed
  haplotype-aware CNA callers such as XClone's BAF module.
when_to_use_negative:
- Input is already a processed count/feature matrix (e.g., a cell x gene expression
  matrix or cell x gene bin counts) with no BAM files available — xcltk needs raw
  alignments to genotype heterozygous variants.
- The analysis goal is copy-number calling from single-cell DNA-seq (e.g., using ACT
  or CHISEL) rather than scRNA-seq-based inference.
- Allelic information is not needed — RDR-only tools such as InferCNV can run from
  the expression matrix alone, though they cannot detect copy-neutral LOH.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: xcltk
  role: Preprocessing toolkit that converts SAM/BAM/CRAM alignments into the allele-specific
    AD/DP matrices and the total read depth matrix required by XClone
  repo: https://github.com/hxj5/xcltk
- name: XClone
  role: Downstream consumer of the AD/DP and RDR matrices; its BAF module uses allelic
    imbalance and its RDR module uses read depth to infer allele-specific CNAs
  repo: https://github.com/single-cell-genetics/XClone
- name: cellsnp-lite
  role: Original genotyping software that xcltk was re-designed from into a package
    tailored for XClone preprocessing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:3e930c176cbec52a70e6322e58620aa77967d3f4b446dd68ce554c9cc3f0c23a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hxj5/xcltk
  tools:
  - InferCNV
  - Numbat
  - XClone
  - cellsnp-lite
  - scCNASimulator
  - xcltk
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# allelic-count-matrix-generation-from-bam

## Summary

Generate the allele-specific AD (alternate-allele depth) and DP (total depth) count matrices from single-cell BAM/CRAM files using xcltk, providing the BAF-signal inputs that XClone needs alongside the RDR read-depth matrix.

## When to use

Use when starting from cell-barcoded BAM/CRAM files from a scRNA-seq experiment (e.g., 10x Genomics) and you need allele-specific AD/DP matrices to feed haplotype-aware CNA callers such as XClone's BAF module.

## When NOT to use

- Input is already a processed count/feature matrix (e.g., a cell x gene expression matrix or cell x gene bin counts) with no BAM files available — xcltk needs raw alignments to genotype heterozygous variants.
- The analysis goal is copy-number calling from single-cell DNA-seq (e.g., using ACT or CHISEL) rather than scRNA-seq-based inference.
- Allelic information is not needed — RDR-only tools such as InferCNV can run from the expression matrix alone, though they cannot detect copy-neutral LOH.

## Inputs

- Cell-barcoded BAM/CRAM alignment files from single-cell RNA-seq (droplet-based e.g. 10x Genomics, or well-based e.g. SMART-seq)
- Reference genome annotation (e.g., hg38_genes, hg38_blocks, or hg19_genes via xclone.pp.load_anno)
- Cell barcode annotation (barcodes.tsv)

## Outputs

- Allele-specific AD (alternate allele count) matrix, cells x variants
- Allele-specific DP (total depth) matrix, cells x variants
- Total read depth (RDR) count matrix, cells x genes
- AnnData objects loadable by xclone.pp for the XClone BAF and RDR modules

## How to apply

Install xcltk via pip ('pip install -U xcltk', or from GitHub for the development version) and run its preprocessing pipeline on the aligned SAM/BAM/CRAM files to extract, per cell and per heterozygous variant, the alternate-allele count (AD) and total read depth (DP). Run the xcltk RDR module independently to obtain the total read depth matrix for the same cells. The three matrices (AD, DP, total read depth) are the exact inputs XClone expects: load them into AnnData with xclone.pp.xcltedata-style loading (genome_mode = 'hg38_genes' or 'hg19_genes', with barcodes.tsv and feature annotation files) before running run_BAF and run_RDR. Rationale: XClone models the B-allele frequency of heterozygous variants and read depth ratio per gene, so the allelic matrices must be generated with the same cell barcodes and gene annotations as the expression matrix.

## Related tools

- **xcltk** (Preprocessing toolkit that converts SAM/BAM/CRAM alignments into the allele-specific AD/DP matrices and the total read depth matrix required by XClone) — https://github.com/hxj5/xcltk
- **XClone** (Downstream consumer of the AD/DP and RDR matrices; its BAF module uses allelic imbalance and its RDR module uses read depth to infer allele-specific CNAs) — https://github.com/single-cell-genetics/XClone
- **cellsnp-lite** (Original genotyping software that xcltk was re-designed from into a package tailored for XClone preprocessing)

## Examples

```
conda create -n xclone python=3.9; conda activate xclone; pip install -U xcltk
```

## Evaluation signals

- The pipeline yields exactly three matrices — allele-specific AD, allele-specific DP, and total read depth — as stated in the xcltk README as the three inputs XClone takes.
- Cell barcodes in the AD/DP matrices match the barcodes.tsv used for the expression/RDR matrix so the same cells can be annotated with xclone.pp.extra_anno.
- The matrices load into AnnData via xclone.pp with a valid genome_mode (e.g., 'hg38_genes') without coordinate or feature-name errors.
- Running xclone.model.run_BAF on the loaded BAF_adata completes without error and produces CNA-state calls consistent with known allele-specific events (e.g., loss on chr3 in simulated GX109-T1c data, detected with AUROC 0.9964).
- Both xcltk modules (RDR and BAF) are run independently per the documented Step1 before loading and analysis.

## Limitations

- xcltk supports data from droplet-based (e.g., 10x Genomics) and well-based (e.g., SMART-seq) platforms but requires raw BAM/CRAM alignments with cell barcodes; pre-counted matrices cannot be used.
- Coverage of a single variant in scRNA-seq is extremely low, so per-SNP allelic signals are noisy; XClone mitigates this by haplotyping across SNPs, genes, and gene bins rather than using individual SNPs.
- InferCNV-style RDR-only approaches that skip allelic matrix generation cannot detect copy-neutral LOH, so the AD/DP matrices are essential whenever allelic events are of interest.

## Evidence

- [readme] xcltk takes three matrices as input: [evidence span withheld — anchor status: external_doc]
- [readme] xcltk implements BAM-to-matrix preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] Platform support: [evidence span withheld — anchor status: external_doc]
- [methods] xcltk installation step in the documented workflow: "Step1: :ref:`Install xcltk <xcltk installation>` and run xctlkt :ref:`RDR module <xcltk RDR>` and :ref:`BAF module <xcltk BAF>`, independently."
- [intro] xcltk re-designed from cellsnp-lite: "we re-designed our original genotyping software cellsnp-lite13 to a tailored package xcltk"
