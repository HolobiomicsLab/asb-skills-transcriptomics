---
name: bone-marrow-subpopulation-shift-analysis-across-transplant-timepoints
description: Use when analyzing paired pre/post-transplant bone marrow mononuclear
  cell scRNA-seq (e.g., SRP073767 AML027/AML035) and you need per-cell host/donor
  genotypes plus cell-type composition changes across timepoints.
when_to_use_negative:
- The sample is expected to have low-level chimerism below the method's detection
  limit (K=1 selection only indicates mixture fraction below detection, determined
  at 4% of 6,000 cells in in silico mixing)
- Clinical chimerism reporting is required for patient management — this method is
  aspirational ('in the near future') and not yet clinically validated
- The input is already an annotated feature/cell-type matrix without SNV-level genotype
  information, so host/donor inference de novo is impossible
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: STAR
  role: Aligns Read1 (cDNA insert) from BMMC scRNA-seq FASTQs to the hg19 reference
    genome as the first step of the chimerism pipeline
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (correction at posterior
    probability >= 0.975), and single-cell 3' gene counting, producing the genome
    BAM and gene-cell-barcode matrices
  repo: https://github.com/10XGenomics/cellranger
- name: Freebayes 1.0.2
  role: Calls SNVs de novo on the Cell Ranger genome BAM (ignoring indels) to discover
    host/donor discriminating variants without prior genotype knowledge
- name: Seurat
  role: Consumes the log-transformed gene-cell-barcode matrix and assigns each cell
    a type by correlation against reference populations, enabling per-genotype composition
    tables
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:a7a3a6663924896f75ce2abeaee7c65d9064da4bc3c757875b9ea68bf6cb26ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  - Seurat
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bone-marrow-subpopulation-shift-analysis-across-transplant-timepoints

## Summary

A de novo SNV-based single-cell chimerism and cell-type composition workflow that assigns host/donor origin to BMMCs from scRNA-seq transcriptomes (STAR + Cell Ranger + Freebayes 1.0.2 + Seurat) and quantifies subpopulation shifts (erythroid expansion, immature granulocytes, monocytopenia) across pre/post transplant timepoints in AML patients.

## When to use

Use when analyzing paired pre/post-transplant bone marrow mononuclear cell scRNA-seq (e.g., SRP073767 AML027/AML035) and you need per-cell host/donor genotypes plus cell-type composition changes across timepoints.

## When NOT to use

- The sample is expected to have low-level chimerism below the method's detection limit (K=1 selection only indicates mixture fraction below detection, determined at 4% of 6,000 cells in in silico mixing)
- Clinical chimerism reporting is required for patient management — this method is aspirational ('in the near future') and not yet clinically validated
- The input is already an annotated feature/cell-type matrix without SNV-level genotype information, so host/donor inference de novo is impossible

## Inputs

- Cryopreserved BMMC scRNA-seq FASTQ files, pre- and post-transplant (SRA accession SRP073767)
- hg19 human reference genome for STAR alignment
- Cell Ranger genome BAM output (possorted genome BAM) for Freebayes SNV calling
- Cell Ranger gene-cell-barcode count matrices for Seurat analysis

## Outputs

- Per-cell host/donor genotype assignments (K=1 or K=2 mixture, e.g., AML027 post: 13.8% minor / 86.2% major genotype)
- Genotype frequencies per sample with similarity to host genotype (e.g., major genotype 97% host-like for AML027; all-donor at 78% for AML035)
- Per-genotype cell-type composition tables from Seurat correlation annotation
- Pre/post transplant subpopulation shift summary (erythroid expansion, ~20% immature granulocytes, post-transplant monocytopenia)
- aml_transplant_chimerism_results.csv

## How to apply

Download cryopreserved BMMC FASTQs (SRA SRP073767), align Read1 cDNA inserts to hg19 with STAR, then run Cell Ranger for sample demultiplexing, barcode processing (correction only at posterior probability >= 0.975), and single-cell 3' gene counting to obtain the genome BAM and gene-cell-barcode matrices. Call SNVs de novo with Freebayes 1.0.2 on the Cell Ranger genome BAM, ignoring indels, retaining only SNVs supported by at least two cell barcodes, SNV Qual >= 30, and minimal SNV base Qual >= 1. Fit the SNV-based Bayesian genotype model at K=1 and K=2, selecting K=2 only when 90% of cells have posterior probability > 75%, and flag cells with an SNV fraction between 0.2 and 0.8 from either genotype as multiplets. Log-transform the gene-cell-barcode matrix and annotate cell types by correlation against reference populations in Seurat, then compute genotype frequencies and per-genotype cell-type compositions for each timepoint. Compare pre/post compositions (e.g., erythroid expansion, ~20% immature granulocytes, post-transplant monocytopenia) and write per-cell genotype assignments and cell-type tables to aml_transplant_chimerism_results.csv.

## Related tools

- **STAR** (Aligns Read1 (cDNA insert) from BMMC scRNA-seq FASTQs to the hg19 reference genome as the first step of the chimerism pipeline) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (correction at posterior probability >= 0.975), and single-cell 3' gene counting, producing the genome BAM and gene-cell-barcode matrices) — https://github.com/10XGenomics/cellranger
- **Freebayes 1.0.2** (Calls SNVs de novo on the Cell Ranger genome BAM (ignoring indels) to discover host/donor discriminating variants without prior genotype knowledge)
- **Seurat** (Consumes the log-transformed gene-cell-barcode matrix and assigns each cell a type by correlation against reference populations, enabling per-genotype composition tables) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
freebayes -f hg19.fa cellranger_out/outs/possorted_genome_bam.bam
```

## Evaluation signals

- AML027 post-transplant selects the K=2 model with a minor genotype near 13.8% and a major genotype ~97% similar to the host genotype
- AML035 yields a K=1 / all-donor call (78% similarity), consistent with no detectable host mixture
- AML027 genotype frequency agrees with the clinical donor chimerism assay (~12% donor chimerism)
- Cell-type composition tables reproduce the reported shifts: erythroid expansion, ~20% immature granulocytes, and post-transplant monocytopenia
- SNV set respects filters: support from >=2 cell barcodes, SNV Qual >= 30, minimal SNV base Qual >= 1, indels ignored; multiplets flagged at SNV fraction 0.2–0.8

## Limitations

- K=1 selection only indicates the mixture fraction is below the level of detection (4% of 6,000 cells in in silico mixing experiments), limiting sensitivity for low-level chimerism
- The proof-of-principle chimerism study included only two transplant patients; larger cohorts were not yet performed
- Cell-type assignment by correlation was low-confidence for some cells (e.g., cytotoxic T vs NK) due to small correlation differences
- Erythrocyte precursor expansion was not detectable by clinical flow cytometry/morphology due to limited surface markers, making cross-validation of this subpopulation difficult
- Some earlier libraries used 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity across libraries (documented in Supplementary Table 1)

## Evidence

- [abstract] we used sequence variation in the transcriptome data to determine host and donor chimerism at single-cell resolution from bone marrow mononuclear cells isolated from transplant patients.: "we used sequence variation in the transcriptome data to determine host and donor chimerism at single-cell resolution from bone marrow mononuclear cells"
- [discussion] The speed, reproducibility and high cell capture rate of GemCode technology allowed us to profile the transcriptome of ∼17,000 single BMMCs: "GemCode technology allowed us to profile the transcriptome of ∼17,000 single BMMCs"
- [discussion] in one patient, AML027, we detected a population of atypical blast and granulocyte precursors that were likely refractory to chemotherapy before transplantation and thus responsible for disease: "in one patient, AML027, we detected a population of atypical blast and granulocyte precursors that were likely refractory to chemotherapy"
- [discussion] Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included.: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [readme] Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting, V(D)J transcript sequence assembly and annotation: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
