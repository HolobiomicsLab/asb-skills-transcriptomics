---
name: bayesian-cell-genotype-mixture-assignment
description: Use when scRNA-seq data (e.g. 10x/GemCode 3' gene-barcode matrices from
  bone marrow mononuclear cells of transplant patients) must be decomposed into host
  vs donor cell genotypes without a prior known genotype.
when_to_use_negative:
- Samples where a priori donor/host genotypes are already available from conventional
  genotyping — the method is designed for de novo calls without prior genotype knowledge
- Detection of very low-level chimerism below ~4% of 6,000 cells, since a K=1 selection
  only indicates the mixture fraction is below the method's detection level
- Bulk (non-single-cell) sequencing data, as the method requires per-cell barcode-resolved
  SNV fractions
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2885
- http://edamontology.org/topic_0625
tools:
- name: Freebayes 1.0.2
  role: De novo SNV calling on the Cell Ranger genome BAM, ignoring indels, to obtain
    the variant matrix used by the Bayesian model
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM and gene-cell-barcode matrices (sample demultiplexing,
    barcode processing, single-cell 3' gene counting) that are the SNV calling input
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 cDNA inserts to the hg19 reference genome prior to Cell Ranger
    counting
- name: Seurat
  role: Annotates each cell's type by correlation against reference populations so
    genotypes can be cross-tabulated with cell types
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:89f5c190e5bbc88469df902a7670c0c8d1075fe7fb599faf6cc98c2a99f5a4b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2885
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bayesian-cell-genotype-mixture-assignment

## Summary

A de novo SNV-based Bayesian method that assigns each single cell a host or donor genotype from scRNA-seq transcriptomes, enabling single-cell-resolution chimerism determination in post-transplant bone marrow samples without prior donor/host genotype knowledge.

## When to use

Use when scRNA-seq data (e.g. 10x/GemCode 3' gene-barcode matrices from bone marrow mononuclear cells of transplant patients) must be decomposed into host vs donor cell genotypes without a prior known genotype.

## When NOT to use

- Samples where a priori donor/host genotypes are already available from conventional genotyping — the method is designed for de novo calls without prior genotype knowledge
- Detection of very low-level chimerism below ~4% of 6,000 cells, since a K=1 selection only indicates the mixture fraction is below the method's detection level
- Bulk (non-single-cell) sequencing data, as the method requires per-cell barcode-resolved SNV fractions

## Inputs

- Genome BAM from Cell Ranger single-cell 3' pipeline (hg19-aligned Read1 cDNA inserts)
- Freebayes 1.0.2 SNV calls filtered to at least two cell barcodes, SNV Qual >=30, base Qual >=1
- Log-transformed gene-cell-barcode UMI count matrix with per-cell type annotations (Seurat)

## Outputs

- Per-cell host/donor genotype posterior assignments (K=1 or K=2 mixture)
- Genotype mixture frequencies (e.g. AML027 post-transplant: 13.8% minor / 86.2% major genotype)
- Multiplet flags for cells with SNV fraction between 0.2 and 0.8 from either genotype
- Per-genotype cell-type composition tables (e.g. aml_transplant_chimerism_results.csv)

## How to apply

First call SNVs de novo by running Freebayes 1.0.2 on the Cell Ranger genome BAM produced after aligning Read1 cDNA inserts to hg19 with STAR, ignoring indels, and retain only SNVs supported by at least two cell barcodes with minimal SNV Qual >=30 and minimal SNV base Qual >=1. Then fit K=1 and K=2 genotype mixtures with the SNV-based Bayesian model, selecting the K=2 model only when 90% of the cells have posterior probability >75%; otherwise K=1 indicates the mixture fraction is below the detection level (~4% of 6,000 cells in in silico mixing). Flag cells with an SNV fraction between 0.2 and 0.8 from either genotype as multiplets. Finally compute genotype frequencies per sample and cross-tabulate genotype against cell type (Seurat correlation-based annotations) to quantify host/donor composition shifts, e.g. AML027 post-transplant showing 13.8% minor genotype with the major genotype 97% similar to host.

## Related tools

- **Freebayes 1.0.2** (De novo SNV calling on the Cell Ranger genome BAM, ignoring indels, to obtain the variant matrix used by the Bayesian model)
- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM and gene-cell-barcode matrices (sample demultiplexing, barcode processing, single-cell 3' gene counting) that are the SNV calling input) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 cDNA inserts to the hg19 reference genome prior to Cell Ranger counting)
- **Seurat** (Annotates each cell's type by correlation against reference populations so genotypes can be cross-tabulated with cell types) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
freebayes -p 1 --ploidy 1 --haplotype-length 0 --min-alternate-count 2 --min-quality 30 --min-base-quality 1 -v snvs.vcf cellranger_genome.bam
```

## Evaluation signals

- AML027 post-transplant should select the K=2 model with a minor genotype frequency near 13.8% (86.2% major) and the major genotype ~97% similar to the host genotype, consistent with the clinical 12% donor chimerism assay
- AML035 should yield a K=1 / all-donor call (~78% similarity) rather than a K=2 mixture
- K=2 model acceptance requires 90% of cells to have posterior probability >75%; otherwise fall back to K=1
- All retained SNVs must satisfy: support from at least two cell barcodes, SNV Qual >=30, and minimal SNV base Qual >=1 (indels ignored)
- Cells with SNV fraction between 0.2 and 0.8 from either genotype are flagged as multiplets, not assigned to a genotype

## Limitations

- K=1 selection only indicates the mixture fraction is below the method's detection level (~4% of 6,000 cells in in silico mixing), limiting sensitivity for low-level chimerism
- Proof-of-principle covered only two transplant patients (AML027, AML035); larger cohorts not yet performed
- Clinical chimerism testing with this method is aspirational ('in the near future'), not yet clinically validated
- Cell type assignment by correlation was low-confidence for some cells (e.g. cytotoxic T vs NK) when correlation differences were small

## Evidence

- [discussion] Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included.: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] K=2 model selection criterion: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] SNVs were called by running Freebayes 1.0.2 on the Cell Ranger genome BAM.: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [abstract] Chimerism determination from transcriptome sequence variation: "we used sequence variation in the transcriptome data to determine host and donor chimerism at single-cell resolution from bone marrow mononuclear cells isolated from transplant patients."
- [discussion] First high-throughput chimerism method claim: "To our knowledge, this is the first report of a high-throughput method to determine chimerism of immune cell populations."
