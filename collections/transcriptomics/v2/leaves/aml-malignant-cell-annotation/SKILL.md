---
name: aml-malignant-cell-annotation
description: Use when a cancer (e.g. AML) 10x scRNA-seq sample needs cell-type annotation
  plus a malignant vs non-malignant call, and bulk SNVs/CNVs must be linked back to
  individual cell barcodes and clusters.
when_to_use_negative:
- You have no pre-called variant set — VarTrix does not perform variant calling, so
  the input VCF must already exist (e.g. from Strelka2 or matched WGS)
- The data are not 10x scRNA-seq with Cell Ranger CB barcode tags and a Cell Ranger
  barcodes.tsv, since vartrix requires these exact inputs
- The goal is pure cell-type annotation of healthy tissue without a malignant/non-malignant
  question — plain ScType marker scoring (sctype_score) suffices without SNV/CopyKAT
  steps
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: ScType
  role: Marker-based cell-type annotation (sctype_score / run_sctype) plus per-cluster
    SNV scoring to flag malignant populations
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: vartrix
  role: Links called SNVs to individual 10x cell barcodes/clusters via Smith-Waterman
    alignment, producing a genome matrix used for per-cluster SNV scores
  repo: https://github.com/10XGenomics/vartrix
- name: Seurat
  role: Normalization, scaling, PCA, clustering and UMAP of the AML scRNA-seq object
    before annotation
- name: samtools
  role: mpileup generation for somatic SNV calling restricted to COSMIC cancer consensus
    genes
- name: Strelka2
  role: Somatic SNV caller producing the VCF consumed by vartrix
- name: CopyKAT
  role: Infers copy-number profiles from the expression matrix to classify cells as
    diploid (non-malignant) or aneuploid (malignant)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:3b88deec8086c6098fed7ef22bef33304457acfcd05238d2d2b1090fb982e2d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - CopyKAT
  - R
  - ScType
  - Seurat
  - Strelka2
  - samtools
  - vartrix
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# aml-malignant-cell-annotation

## Summary

Annotates AML patient scRNA-seq cell types with ScType and separates malignant from non-malignant populations by combining per-cluster single-cell SNV scores (Strelka2/vartrix) with CopyKAT copy-number calls.

## When to use

Use when a cancer (e.g. AML) 10x scRNA-seq sample needs cell-type annotation plus a malignant vs non-malignant call, and bulk SNVs/CNVs must be linked back to individual cell barcodes and clusters.

## When NOT to use

- You have no pre-called variant set — VarTrix does not perform variant calling, so the input VCF must already exist (e.g. from Strelka2 or matched WGS)
- The data are not 10x scRNA-seq with Cell Ranger CB barcode tags and a Cell Ranger barcodes.tsv, since vartrix requires these exact inputs
- The goal is pure cell-type annotation of healthy tissue without a malignant/non-malignant question — plain ScType marker scoring (sctype_score) suffices without SNV/CopyKAT steps

## Inputs

- 10x Genomics AML scRNA-seq dataset (Cell Ranger BAM with CB barcode tags, filtered_gene_bc_matrices barcodes.tsv)
- Reference genome FASTA with index
- Pre-called SNV VCF from samtools mpileup + Strelka2, restricted to COSMIC cancer consensus genes
- Seurat-normalized expression matrix (counts or scaled data)
- ScType marker database file (ScTypeDB_full.xlsx, Immune system tissue)

## Outputs

- Per-cell-type ScType annotations (sctype_classification metadata column)
- Per-cluster/cell-type ScType SNV score matrix from the vartrix genome matrix
- CopyKAT diploid (non-malignant) vs aneuploid (malignant) cell classification
- Chord diagram of SNV-score similarity separating malignant and non-malignant populations

## How to apply

Download the AML patient scRNA-seq data and preprocess with Seurat (NormalizeData LogNormalize with scale.factor 10000, ScaleData, RunPCA, FindNeighbors/FindClusters, RunUMAP), then annotate clusters with sctype_score using the full ScType marker database (ScTypeDB_full.xlsx, 'Immune system' tissue). Call somatic SNVs with samtools mpileup and Strelka2, restricting calls to COSMIC cancer consensus genes, then feed the VCF to vartrix together with the Cell Ranger BAM, genome FASTA, and barcodes.tsv to assign variants to cell barcodes. Compute per-cluster ScType SNV scores from the vartrix genome matrix and run CopyKAT on the expression matrix to infer copy-number profiles, classifying cells as diploid (non-malignant) or aneuploid (malignant). Label populations non-malignant only when both criteria agree: ScType SNV score <20 AND CopyKAT diploid majority. Finally render a chord diagram of SNV-score similarity between cell types to confirm clean separation of non-malignant (CD24+ CD66+ neutrophils, memory CD8+ T cells) from malignant (CD34+ HSC/MPP, CD34+ ISG+ blast) populations.

## Related tools

- **ScType** (Marker-based cell-type annotation (sctype_score / run_sctype) plus per-cluster SNV scoring to flag malignant populations) — https://github.com/IanevskiAleksandr/sc-type
- **vartrix** (Links called SNVs to individual 10x cell barcodes/clusters via Smith-Waterman alignment, producing a genome matrix used for per-cluster SNV scores) — https://github.com/10XGenomics/vartrix
- **Seurat** (Normalization, scaling, PCA, clustering and UMAP of the AML scRNA-seq object before annotation)
- **samtools** (mpileup generation for somatic SNV calling restricted to COSMIC cancer consensus genes)
- **Strelka2** (Somatic SNV caller producing the VCF consumed by vartrix)
- **CopyKAT** (Infers copy-number profiles from the expression matrix to classify cells as diploid (non-malignant) or aneuploid (malignant))

## Examples

```
./vartrix -v cosmic_snvs.vcf -b cellranger_possorted.bam -f reference.fa -c barcodes.tsv -o snv_genome_matrix.mtx
```

## Evaluation signals

- CD34+ HSC/MPP and CD34+ ISG+ blast clusters show higher ScType SNV scores than non-malignant CD24+ CD66+ neutrophils and memory CD8+ T cells
- Non-malignant populations (CD2+CD66+ neutrophils, memory CD8+ T cells) satisfy ScType SNV score <20
- CopyKAT reports a diploid majority for the non-malignant clusters and aneuploid calls for malignant blasts
- The chord diagram shows clean non-malignant vs malignant separation with no cross-group SNV-score similarity edges
- VarTrix output matrix column labels exactly match the barcodes in the Cell Ranger barcodes.tsv input file

## Limitations

- VarTrix does not perform variant calling; it only assigns pre-called variants, and multi-allelic sites are ignored (left as empty rows)
- Variant calling from bulk or single-cell RNA-seq is challenging because reverse-transcription noise leads to a high false-positive rate; matched WGS or careful GATK/Mutect2 RNA-seq best practices are recommended
- ScType's z-score transformation may be suboptimal for single-cell transcriptomic data, and drop-out events are not corrected in the current version
- Negative markers remain relatively few compared to positive markers in the ScType database, which can limit annotation confidence

## Evidence

- [intro] ScType implements a single-cell single-nucleotide variant (SNV) calling option to distinguish between malignant and non-malignant cells: "ScType implements a single-cell single-nucleotide variant (SNV) calling option to distinguish between malignant and non-malignant cells (Fig. 1a)"
- [discussion] Next, ScType connects the SNV to each cluster using vartrix: "Next, ScType connects the SNV to each cluster using vartrix (https://github.com/10XGenomics/vartrix)."
- [discussion] We utilized Seurat v4.042 for data processing and normalization: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] VarTrix extracts single cell variant information from 10x data using previously defined variant calls: [evidence span withheld — anchor status: external_doc]
- [readme] VarTrix requires a pre-called variant set in VCF format, alignments, and a genome FASTA: [evidence span withheld — anchor status: external_doc]
- [methods] sctype_wrapper / run_sctype invocation for automated cell-type assignment: "source("https://raw.githubusercontent.com/kris-nader/sc-type/master/R/sctype_wrapper.R");"
