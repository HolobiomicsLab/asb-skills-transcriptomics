---
name: vartrix-allele-per-cell-matrix-generation
description: Use when you have a set of pre-called variants (VCF), a Cell Ranger BAM
  with CB tags, and a cell barcodes file, and need to assign variants to single cells/clusters
  for malignant-cell identification.
when_to_use_negative:
- You need to discover/call variants from scratch — VarTrix explicitly does not perform
  variant calling and requires a pre-called VCF as input
- Your BAM lacks the CB cell barcode tag or is not from Cell Ranger / Cell Ranger
  DNA output
- Your variants are multi-allelic sites — these are ignored and will yield empty values
  in the output matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: VarTrix
  role: Assigns pre-called variants to individual cell barcodes via Smith-Waterman
    alignment and outputs a cells-by-variants allele matrix
  repo: https://github.com/10XGenomics/vartrix
- name: Strelka2
  role: Calls the somatic SNVs (with samtools mpileup) that form the input VCF, restricted
    to COSMIC cancer consensus genes
- name: ScType
  role: Consumes the per-cell variant matrix to compute per-cluster SNV scores distinguishing
    malignant from non-malignant populations
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:2675940503478bd92c25759a28d5e84c8812be45ff943358400154f23f2f86dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/vartrix
  tools:
  - R
  - ScType
  - Strelka2
  - VarTrix
  - samtools
  - vartrix
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# vartrix-allele-per-cell-matrix-generation

## Summary

Generates a cells-by-variants allele matrix from 10x scRNA-seq data using VarTrix, linking pre-called SNVs (e.g. Strelka2 calls restricted to COSMIC cancer consensus genes) to individual cell barcodes so per-cluster ScType SNV scores can separate malignant from non-malignant populations.

## When to use

Use when you have a set of pre-called variants (VCF), a Cell Ranger BAM with CB tags, and a cell barcodes file, and need to assign variants to single cells/clusters for malignant-cell identification.

## When NOT to use

- You need to discover/call variants from scratch — VarTrix explicitly does not perform variant calling and requires a pre-called VCF as input
- Your BAM lacks the CB cell barcode tag or is not from Cell Ranger / Cell Ranger DNA output
- Your variants are multi-allelic sites — these are ignored and will yield empty values in the output matrix

## Inputs

- Pre-called variant set in VCF format (e.g. Strelka2 calls restricted to COSMIC cancer consensus genes)
- Cell Ranger BAM with CB tag and index file (10x scRNA-seq alignments)
- Indexed reference genome FASTA matching the BAM sequence names
- Cell barcodes file from Cell Ranger (one barcode per line)
- Optional: cell/cluster annotations from a Seurat object (ScType sctype_score output) for per-cluster SNV scoring

## Outputs

- Cells-by-variants sparse Market Exchange (MEX) format matrix with cell barcodes as column labels
- Optional text file of row names with variants named as $chromosome_$pos (via --out-variants)
- Per-cluster ScType SNV scores for malignant vs non-malignant classification (downstream in R)

## How to apply

First call somatic SNVs externally (VarTrix does not perform variant calling) — in the ScType AML workflow, samtools mpileup and Strelka2 were used, restricted to COSMIC cancer consensus genes, producing a VCF. Then run VarTrix with the required arguments: the input VCF (-v), the Cell Ranger BAM (-b, which must carry the CB tag and be indexed), the reference genome FASTA (-f, indexed), and the Cell Ranger cell barcodes file (-c, one barcode per line from outs/filtered_gene_bc_matrices_mex/${refGenome}/barcodes.tsv). VarTrix uses Smith-Waterman alignment to evaluate reads at each variant locus and assigns variants to cell barcodes, writing a Market Exchange (MEX) sparse matrix whose columns are the supplied barcodes and whose rows are variants named chromosome_position (via --out-variants). Read this matrix into R/Seurat and aggregate per cluster to compute per-cluster ScType SNV scores; in the AML validation, non-malignant CD24+ CD66+ neutrophils and memory CD8+ T cells had SNV scores <20 while CD34+ HSC/MPP and ISG+ blasts scored higher.

## Related tools

- **VarTrix** (Assigns pre-called variants to individual cell barcodes via Smith-Waterman alignment and outputs a cells-by-variants allele matrix) — https://github.com/10XGenomics/vartrix
- **Strelka2** (Calls the somatic SNVs (with samtools mpileup) that form the input VCF, restricted to COSMIC cancer consensus genes)
- **ScType** (Consumes the per-cell variant matrix to compute per-cluster SNV scores distinguishing malignant from non-malignant populations) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
./vartrix -v calls.vcf -b possorted_genome_bam.bam -f reference.fa -c barcodes.tsv -o allele_matrix.mtx --out-variants variant_rows.txt
```

## Evaluation signals

- Output matrix is valid MEX sparse format with column labels exactly matching the supplied cell barcodes file
- Row names file (--out-variants) lists variants as chromosome_position in the same order as the input VCF
- Multi-allelic input sites produce empty rows in the matrix (expected VarTrix behavior)
- Downstream ScType SNV scores <20 for known non-malignant cells (CD24+ CD66+ neutrophils, memory CD8+ T cells) and higher scores for CD34+ HSC/MPP and ISG+ blasts in the AML sample
- CopyKAT classifies the same barcodes that scored high as aneuploid/malignant, confirming concordance

## Limitations

- Generating variants from bulk or single cell RNA-seq data is challenging due to reverse-transcription noise and high false positive rates; the VarTrix README recommends GATK/Mutect2 RNAseq best practices or WGS from the same sample
- Multi-allelic sites are ignored entirely
- VarTrix is provided without full support; issues must be raised on GitHub

## Evidence

- [readme] VarTrix does not perform variant calling: [evidence span withheld — anchor status: external_doc]
- [readme] VarTrix uses Smith-Waterman alignment: [evidence span withheld — anchor status: external_doc]
- [discussion] ScType connects the SNV to each cluster using vartrix: "Next, ScType connects the SNV to each cluster using vartrix (https://github.com/10XGenomics/vartrix)."
- [intro] ScType implements a single-cell SNV calling option: "ScType implements a single-cell single-nucleotide variant (SNV) calling option to distinguish between malignant and non-malignant cells (Fig. 1a)"
- [readme] Required arguments include VCF, BAM with CB tag, FASTA, and cell barcodes: [evidence span withheld — anchor status: external_doc]
