---
name: snrna-seq-rdr-baf-input-construction
description: Use when you need to convert raw snRNA-seq/scRNA-seq alignments (BAM/CRAM)
  into XClone-ready RDR and AD/DP AnnData objects for allele-specific CNA detection
  on a tumour dataset such as the 2R astrocytoma snRNA-seq sample.
when_to_use_negative:
- The data are bulk DNA-seq or single-cell DNA-seq profiles — XClone models raw read/UMI
  counts from scRNA-seq, and scDNA CNA tools like ACT or CHISEL are the appropriate
  family.
- Only a normalised expression matrix is available and the allele-specific AD/DP matrices
  cannot be generated from source BAM/CRAM files, since XClone's BAF module cannot
  run from RDR alone in allele-specific applications.
- The goal is CNA calling in the absence of allelic information only for methods like
  InferCNV/CopyKAT that never consume AD/DP inputs.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: xcltk
  role: Preprocessing toolkit that generates the RDR, AD, and DP matrices from SAM/BAM/CRAM
    files via its independent RDR and BAF modules
  repo: https://github.com/hxj5/xcltk
- name: XClone
  role: Consumes the constructed AnnData inputs; xclone.pp.xclonedata, xclone.pp.load_anno,
    and xclone.pp.extra_anno assemble and annotate the RDR/BAF input objects
  repo: https://github.com/single-cell-genetics/XClone
- name: cellsnp-lite
  role: Original genotyping software that was re-designed into the tailored xcltk
    preprocessing package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:1954e5e46095cdf8fd66a40e18f204ca7da4126c303af0e41753ef8bee2dddd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hxj5/xcltk
  tools:
  - CaSpER
  - CopyKAT
  - InferCNV
  - Numbat
  - XClone
  - cellsnp-lite
  - xcltk
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# snrna-seq-rdr-baf-input-construction

## Summary

Constructs the three XClone input matrices from single-nucleus RNA-seq data — a cell-by-gene total read-depth matrix (RDR) and allele-specific AD/DP matrices (BAF) — wrapped into AnnData objects with hg38 gene/barcode annotation, so the XClone RDR, BAF, and combine modules can detect allele-specific CNAs.

## When to use

Use when you need to convert raw snRNA-seq/scRNA-seq alignments (BAM/CRAM) into XClone-ready RDR and AD/DP AnnData objects for allele-specific CNA detection on a tumour dataset such as the 2R astrocytoma snRNA-seq sample.

## When NOT to use

- The data are bulk DNA-seq or single-cell DNA-seq profiles — XClone models raw read/UMI counts from scRNA-seq, and scDNA CNA tools like ACT or CHISEL are the appropriate family.
- Only a normalised expression matrix is available and the allele-specific AD/DP matrices cannot be generated from source BAM/CRAM files, since XClone's BAF module cannot run from RDR alone in allele-specific applications.
- The goal is CNA calling in the absence of allelic information only for methods like InferCNV/CopyKAT that never consume AD/DP inputs.

## Inputs

- SAM/BAM/CRAM alignment files from 10x Genomics snRNA-seq (e.g. the 4416-cell 2R astrocytoma snRNA-seq dataset)
- Reference genome annotation (hg38 genes and hg38 blocks, loadable via xclone.pp.load_anno genome_mode)
- Cell annotation table (e.g. clone labels for the 53-cell 2R clone 1) for xclone.pp.extra_anno

## Outputs

- Cell-by-gene total read-depth AnnData (RDR) suitable for xclone.model.run_RDR
- Allele-specific AD and DP count AnnData objects (BAF) suitable for xclone.model.run_BAF
- Annotation-enriched XClone input objects carrying hg38 gene/barcode metadata and extra cell annotations

## How to apply

Run xcltk's RDR and BAF preprocessing modules independently on the SAM/BAM/CRAM files to generate the three required matrices: total read depth (RDR) plus allele-specific AD and DP counts at heterozygous variants (BAF signals). Load genome/feature annotations with xclone.pp.load_anno in hg38 mode (e.g. "hg38_genes", "hg38_blocks"), then assemble the cell-by-gene RDR matrix and the allele-specific AD/DP matrices into AnnData objects via xclone.pp.xclonedata, keeping hg38 gene and barcode annotation consistent across all three inputs. Add sample-level metadata such as cell-type clone labels with xclone.pp.extra_anno before modelling. The rationale is that XClone models raw UMI/read counts, not normalised expression, and its BAF module needs the allele-specific count pair (AD, DP) rather than a precomputed BAF fraction. Verify that cell barcodes and gene identifiers agree across the RDR and BAF objects before passing them to xclone.model.run_RDR and run_BAF.

## Related tools

- **xcltk** (Preprocessing toolkit that generates the RDR, AD, and DP matrices from SAM/BAM/CRAM files via its independent RDR and BAF modules) — https://github.com/hxj5/xcltk
- **XClone** (Consumes the constructed AnnData inputs; xclone.pp.xclonedata, xclone.pp.load_anno, and xclone.pp.extra_anno assemble and annotate the RDR/BAF input objects) — https://github.com/single-cell-genetics/XClone
- **cellsnp-lite** (Original genotyping software that was re-designed into the tailored xcltk preprocessing package)

## Examples

```
hg38_genes = xclone.pp.load_anno(genome_mode = "hg38_genes"); RDR_adata = xclone.pp.xclonedata(RDR_matrix, "cell_by_gene", hg38_genes)
```

## Evaluation signals

- All three output matrices (RDR, AD, DP) share identical cell barcodes and the same hg38 gene ordering; AD entries never exceed the corresponding DP entries.
- RDR is a raw cell-by-gene total read-depth count matrix (not log-normalised expression), consistent with XClone modelling 'the raw read/UMI counts from scRNA-seq data'.
- Genome annotation is loaded in hg38 mode (hg38_genes, hg38_blocks) so gene coordinates and chromosomal blocks used for smoothing match the reference genome of the dataset.
- Extra cell annotations (e.g. clone membership such as the 53-cell 2R clone 1) are present in the AnnData after xclone.pp.extra_anno and survive into downstream RDR/BAF/combine runs.
- The assembled inputs run end-to-end through xclone.model.run_RDR and xclone.model.run_BAF without shape or barcode-mismatch errors, producing per-cell, per-gene CNA states comparable to the published astrocytoma results.

## Limitations

- Allele-specific (BAF-module) inputs require heterozygous variant coverage; at single-variant coverage the signal is noisy, which is why methods using individual SNPs independently 'often suffers from high noise due to extremely low coverage'.
- Input construction is platform-dependent: xcltk supports droplet-based (10x Genomics) and well-based (SMART-seq) platforms, so preprocessing settings must match the assay of the source data.
- InferCNV and CopyKAT do not consume allelic information and 'cannot detect copy-neutral loss of heterozygosity (LOH)', so these matrices enable capabilities not all downstream competitors share.

## Evidence

- [readme] It takes three matrices as input: the allele-specific *AD* and *DP* matrices (BAF signals) and the *total read depth* matrix (RDR signals).: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [methods] hg38 genes annotation loading via xclone.pp.load_anno: "hg38_genes = xclone.pp.load_anno(genome_mode = "hg38_genes")"
- [abstract] XClone strengthens read-depth and allelic imbalance signals for CNA detection: "strengthens the signals of read depth and allelic imbalance by effective smoothing on cell neighborhood and gene coordinate graphs"
- [readme] xcltk provides independent RDR and BAF preprocessing modules: "The [xcltk][xcltk repo] package implements a preprocessing pipeline to 
generate the three matrices from SAM/BAM/CRAM files."
