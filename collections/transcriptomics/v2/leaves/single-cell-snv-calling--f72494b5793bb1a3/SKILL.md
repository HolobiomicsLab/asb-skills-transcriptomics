---
name: single-cell-snv-calling
description: Use when a 10x Genomics cell-barcoded tumor scRNA-seq sample (e.g., AML
  patient) must be split into malignant vs non-malignant populations directly from
  expressed somatic variants rather than from expression clustering alone.
when_to_use_negative:
- VarTrix does not perform variant calling, so it cannot be used without a pre-called
  VCF — do not reach for this skill when no somatic variant calls (bulk or RNA-derived)
  exist for the sample
- Non-tumor or healthy-donor scRNA-seq data lacking somatic variants, where marker-based
  annotation alone (sctype_score) is sufficient
- Datasets without cell-barcoded 10x alignments (e.g., plate-based scRNA-seq without
  CB tags), since VarTrix requires a Cell Ranger BAM and barcodes file
edam_topics:
- http://edamontology.org/topic_2648
- http://edamontology.org/topic_0199
tools:
- name: ScType
  role: Annotates clusters via sctype_score and computes per-cluster SNV scores that
    separate malignant from non-malignant populations
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: VarTrix
  role: Assigns pre-called SNVs to individual cell barcodes using Smith-Waterman alignment,
    producing the genome matrix linked to Seurat clusters
  repo: https://github.com/10XGenomics/vartrix
- name: samtools + Strelka2
  role: Call somatic SNVs from RNA alignments, restricted to COSMIC cancer consensus
    genes, to generate VarTrix's input VCF
- name: CopyKAT
  role: Infers copy-number profiles from the expression matrix to independently validate
    cells as diploid (non-malignant) or aneuploid (malignant)
- name: Seurat
  role: Normalizes, scales, and clusters the patient scRNA-seq data so SNV scores
    can be aggregated per cluster
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:164b064a40cd9a576097df186efa75057f4feeb3f191663cb87bdca2f9dc71e8
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
  - VarTrix
  - samtools
  - samtools + Strelka2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2648
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# single-cell-snv-calling

## Summary

Calls somatic SNVs from scRNA-seq data (samtools mpileup + Strelka2, restricted to COSMIC cancer consensus genes), links variants to individual cell barcodes/clusters with VarTrix, and computes per-cluster ScType SNV scores to distinguish malignant from non-malignant populations, with CopyKAT copy-number profiles as an orthogonal check.

## When to use

Use when a 10x Genomics cell-barcoded tumor scRNA-seq sample (e.g., AML patient) must be split into malignant vs non-malignant populations directly from expressed somatic variants rather than from expression clustering alone.

## When NOT to use

- VarTrix does not perform variant calling, so it cannot be used without a pre-called VCF — do not reach for this skill when no somatic variant calls (bulk or RNA-derived) exist for the sample
- Non-tumor or healthy-donor scRNA-seq data lacking somatic variants, where marker-based annotation alone (sctype_score) is sufficient
- Datasets without cell-barcoded 10x alignments (e.g., plate-based scRNA-seq without CB tags), since VarTrix requires a Cell Ranger BAM and barcodes file

## Inputs

- 10x Genomics scRNA-seq data for a tumor patient sample (Cell Ranger BAM with CB tags, filtered_gene_bc_matrices, barcodes.tsv)
- Reference genome FASTA with index
- Pre-called variant VCF restricted to COSMIC cancer consensus genes (produced via samtools mpileup + Strelka2)
- Processed Seurat object with normalized/scaled expression and cluster annotations (ScTypeDB_full.xlsx marker database)

## Outputs

- Sparse Market Matrix genome matrix of variant-by-cell-barcode assignments from VarTrix
- Per-cluster (and per-cell-type) ScType SNV scores separating malignant from non-malignant populations
- CopyKAT copy-number classification of cells as diploid (non-malignant) or aneuploid (malignant)
- Chord diagram of SNV-score similarity across cell types with non-malignant/malignant separation

## How to apply

Download the patient scRNA-seq data (e.g., AML sample from GEO, such as the GSE76008-style AML hierarchy study), preprocess and cluster with Seurat (NormalizeData, ScaleData, FindClusters), and annotate clusters with ScType's sctype_score using the full marker database (ScTypeDB_full.xlsx, 'Immune system' tissue). Call somatic SNVs from the RNA alignments with samtools mpileup and Strelka2, restricting the calls to COSMIC cancer consensus genes to limit reverse-transcription noise-driven false positives. Feed the resulting VCF together with the Cell Ranger BAM, reference FASTA, and cell barcodes file into VarTrix, which uses Smith-Waterman alignment to assign each variant to cell barcodes and outputs a sparse Market Matrix genome matrix. Link variants back to Seurat clusters and compute per-cluster ScType SNV scores; validate malignancy calls independently with CopyKAT, classifying cells as diploid (non-malignant) or aneuploid (malignant). Finally, compare SNV-score similarity across cell types and render a chord diagram to visualize the non-malignant/malignant separation.

## Related tools

- **ScType** (Annotates clusters via sctype_score and computes per-cluster SNV scores that separate malignant from non-malignant populations) — https://github.com/IanevskiAleksandr/sc-type
- **VarTrix** (Assigns pre-called SNVs to individual cell barcodes using Smith-Waterman alignment, producing the genome matrix linked to Seurat clusters) — https://github.com/10XGenomics/vartrix
- **samtools + Strelka2** (Call somatic SNVs from RNA alignments, restricted to COSMIC cancer consensus genes, to generate VarTrix's input VCF)
- **CopyKAT** (Infers copy-number profiles from the expression matrix to independently validate cells as diploid (non-malignant) or aneuploid (malignant))
- **Seurat** (Normalizes, scales, and clusters the patient scRNA-seq data so SNV scores can be aggregated per cluster)

## Examples

```
./vartrix -v cosmic_snvs.vcf -b cellranger_bam/possorted_genome_bam.bam -f reference.fa -c outs/filtered_gene_bc_matrices_mex/GRCh38/barcodes.tsv -o snv_genome_matrix.mtx
```

## Evaluation signals

- In the AML patient sample, ScType SNV scores are higher in CD34+ progenitor (HSC/MPP) cells and CD34+ ISG+ blast cells than in non-malignant CD24+ CD66+ neutrophils and memory CD8+ T cells
- Non-malignant populations (e.g., CD2+CD66+ neutrophils and memory CD8+ T cells) show ScType SNV scores below the threshold of 20
- CopyKAT independently classifies the majority of non-malignant cells as diploid, consistent with low SNV scores
- VarTrix output is a valid sparse Market Matrix with column labels matching the input cell barcodes file and row names of the form $chromosome_$pos when --out-variants is used
- The chord diagram shows clean separation between non-malignant and malignant populations

## Limitations

- Generating variants from RNA-seq is challenging because noise inherent in reverse transcription leads to a high false positive rate, motivating COSMIC gene restriction and recommended GATK/Mutect2 RNA-seq best practices
- VarTrix does not perform variant calling — it only assigns pre-called variants; all multi-allelic sites are ignored and left empty in the output matrix
- z-score transformation used by ScType for combining markers may be suboptimal for single-cell transcriptomic data, and drop-out events are not corrected in the current version

## Evidence

- [intro] ScType implements single-cell SNV calling for malignancy separation: "ScType implements a single-cell single-nucleotide variant (SNV) calling option to distinguish between malignant and non-malignant cells (Fig. 1a)"
- [discussion] VarTrix links SNVs to clusters: "Next, ScType connects the SNV to each cluster using vartrix (https://github.com/10XGenomics/vartrix)."
- [readme] VarTrix requires pre-called variants: [evidence span withheld — anchor status: external_doc]
- [readme] VarTrix alignment method and scope: [evidence span withheld — anchor status: external_doc]
- [readme] RNA-seq variant calling false positives: [evidence span withheld — anchor status: external_doc]
