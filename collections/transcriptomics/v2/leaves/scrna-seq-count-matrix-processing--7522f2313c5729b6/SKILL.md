---
name: scrna-seq-count-matrix-processing
description: Use when scRNA-seq data from 10x Chromium 3' (v2) or 5' (v1) Gene Expression
  workflows must be converted into per-cell UMI count matrices and per-cell mutation
  genotype calls (from _counts_UB.tsv or cb_sniffer/VarTrix output) to connect genotype
  to phenotype at single-cell resolution.
when_to_use_negative:
- 'The task requires detecting de novo variants from scRNA-seq reads alone: VarTrix
  explicitly does not perform variant calling, and noise from reverse transcription
  yields a high false-positive rate, so variants should be pre-called from eWGS of
  the same sample'
- The samples are full-length cDNA plate-based data (Fluidigm C1/SMART-seq) generated
  for CNA inference rather than end-biased Chromium data processed via cb_sniffer/VarTrix
  barcodes
- Input is already a processed per-cell expression-plus-genotype table; proceed directly
  to regression or signature analysis instead of re-deriving count matrices
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: python3
  role: Scripting language used in methods for count-matrix processing and genotype-to-expression
    analyses, run inside the sridnona/python3:180925.v1 Docker image
  repo: https://github.com/genome/scrna_mutations
- name: pysam
  role: Python library used in methods to parse BAM alignments when counting UB_tag
    reads per barcode
  repo: https://github.com/genome/scrna_mutations
- name: cb_sniffer
  role: Mutation barcode caller that parses CB barcodes from 10x single-cell BAMs
    to call mutant and reference barcodes per cell
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative tool that extracts single-cell variant information from 10x data
    by Smith-Waterman alignment of reads at pre-called variant loci
  repo: https://github.com/10XGenomics/vartrix
- name: Docker
  role: Execution environment (sridnona/python3:180925.v1 image) in which the python3/pysam
    processing was performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:d027d7d07e8f71ddc019d01ff973542e41e96436a0dc9a8cdd1aafd3c2cf0524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genome/scrna_mutations
  tools:
  - Docker
  - VarTrix
  - cb_sniffer
  - pysam
  - python3
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scrna-seq-count-matrix-processing

## Summary

Process 10x Genomics Chromium single-cell RNA-seq count matrices (e.g. the _counts_UB.tsv UB_tag read-count tables) to prepare per-cell expression and mutation-genotype data for downstream subclone-to-phenotype analyses in AML. This includes loading UMI counts, genotyping cells for expressed somatic SNVs, and overlaying mutant-cell labels onto cluster/t-SNE space.

## When to use

Use when scRNA-seq data from 10x Chromium 3' (v2) or 5' (v1) Gene Expression workflows must be converted into per-cell UMI count matrices and per-cell mutation genotype calls (from _counts_UB.tsv or cb_sniffer/VarTrix output) to connect genotype to phenotype at single-cell resolution.

## When NOT to use

- The task requires detecting de novo variants from scRNA-seq reads alone: VarTrix explicitly does not perform variant calling, and noise from reverse transcription yields a high false-positive rate, so variants should be pre-called from eWGS of the same sample
- The samples are full-length cDNA plate-based data (Fluidigm C1/SMART-seq) generated for CNA inference rather than end-biased Chromium data processed via cb_sniffer/VarTrix barcodes
- Input is already a processed per-cell expression-plus-genotype table; proceed directly to regression or signature analysis instead of re-deriving count matrices

## Inputs

- Cellranger-produced BAM files with CB cell-barcode tags from 10x Chromium 5' (v1) or 3' (v2) Gene Expression runs
- Variant file with header columns chrm, start, stop, ref, var, gene_name, trv_type (somatic SNVs discovered by eWGS of paired tumor/normal)
- List of good cell barcodes (e.g. outs/filtered_gene_bc_matrices_mex/${refGenome}/barcodes.tsv)
- Reference genome FASTA with index and pre-called variant VCF (for VarTrix)

## Outputs

- UB_tag read-count matrix per Chromium molecular barcode (_counts_UB.tsv)
- Per-cell genotype calls distinguishing mutant, reference, and ungenotyped barcodes for each expressed somatic mutation
- Mutant-cell labels overlaid on cluster/t-SNE coordinates of the full sample for downstream regression

## How to apply

Start from the BAMs produced by the 10x cellranger pipeline and count reads per Chromium molecular barcode (UB_tag) to obtain the _counts_UB.tsv count table, processing inside the sridnona/python3:180925.v1 Docker image using python3 with pysam. Call mutant and reference barcodes per cell using cb_sniffer (python3 cb_sniffer.py with a variant file of chrm/start/stop/ref/var/gene_name/trv_type columns, applying --filter for reads per barcode and MAPQ/BASEQ thresholds) or with VarTrix, which assigns cells to pre-called SNV/indel loci from a VCF using Smith-Waterman alignment. Analyze the genotyped cells in the context of the entire sample, so expression information from additional non-genotyped cells is leveraged, and overlay mutant-cell labels onto t-SNE/cluster coordinates of the full sample. The result is a per-cell expression matrix annotated with binary or graded mutation status that can feed downstream density regression or subclone expression-signature analyses.

## Related tools

- **python3** (Scripting language used in methods for count-matrix processing and genotype-to-expression analyses, run inside the sridnona/python3:180925.v1 Docker image) — https://github.com/genome/scrna_mutations
- **pysam** (Python library used in methods to parse BAM alignments when counting UB_tag reads per barcode) — https://github.com/genome/scrna_mutations
- **cb_sniffer** (Mutation barcode caller that parses CB barcodes from 10x single-cell BAMs to call mutant and reference barcodes per cell) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative tool that extracts single-cell variant information from 10x data by Smith-Waterman alignment of reads at pre-called variant loci) — https://github.com/10XGenomics/vartrix
- **Docker** (Execution environment (sridnona/python3:180925.v1 image) in which the python3/pysam processing was performed)

## Examples

```
python3 cb_sniffer.py bam_file variant_file barcodes.tsv upn --filter 0 --mapq 0 --baseq 1
```

## Evaluation signals

- The count table contains read counts per Chromium molecular barcode (UB_tag) matching the _counts_UB.tsv format described in the methods
- cb_sniffer parameters are set explicitly: --filter reads per barcode (default 0), --mapq (default 0), --baseq (default 1), and the variant file carries chrm/start/stop/ref/var/gene_name/trv_type columns
- Hundreds to thousands of cells per AML sample are called as containing tumor-specific mutations, consistent with the paper's reported yields across five cryopreserved AML samples
- VarTrix output is a Market Matrix-format genome matrix whose columns match the Cell Ranger barcodes.tsv input, with multi-allelic sites left empty but retained as rows
- Mutant-cell labels overlay sensibly on the t-SNE/cluster space when analyzed in the context of the entire sample including non-genotyped cells

## Limitations

- Dropout (transcript and allelic) occurs on most scRNA-seq platforms and reduces the sensitivity of mutation detection by a factor of two; a cell can never be definitively called wild-type
- Partial transcript coverage on end-biased Chromium platforms limits variant-detection sensitivity, and coverage drops non-linearly across transcript length so some variants are more easily detectable than others
- VarTrix does not perform variant calling and ignores all multi-allelic sites; noise inherent in reverse transcription leads to a high false-positive rate if variants are called from RNA-seq itself rather than from matched WGS
- The utility of the approach depends on the sample's mutational composition and will likely perform better for tumor types with higher mutation burdens than AML

## Evidence

- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: [evidence span withheld — anchor status: not_found]
- [readme] cb_sniffer parses CB barcodes from 10x data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   Bams aligned using 10x cellranger pipeline"
- [readme] VarTrix assigns cells to variants via Smith-Waterman alignment: [evidence span withheld — anchor status: external_doc]
- [discussion] Analyze genotyped cells in context of the entire sample: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
- [abstract] Hundreds to thousands of mutant cells identified per AML sample: "Applying this approach to five cryopreserved AML samples, we identify hundreds to thousands of cells containing tumor-specific mutations in each case"
