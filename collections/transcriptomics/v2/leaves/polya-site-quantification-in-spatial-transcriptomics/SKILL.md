---
name: polya-site-quantification-in-spatial-transcriptomics
description: Use when raw spatially barcoded ST FASTQ reads (barcodes/UMIs in read
  1) must be turned into a poly(A) site-by-spot matrix and per-spot APA usage ratios
  (RUD) before imputation or layer-level APA analysis.
when_to_use_negative:
- The spatial assay is image-based (MERFISH/seqFISH) rather than spatially barcoded
  ST — the paper notes APA sites can only be identified from spatially barcoded ST
  data using tools like scAPAtrap or Sierra.
- A poly(A) site matrix (PACdataset) already exists — skip to imputation (imputeAPAIndex)
  and spatial clustering instead of re-running site identification.
- The goal is DEAPA/LSAPA/SVAPA gene calling from an already-imputed RUD matrix —
  use findContrastAPA/findAllMarkers/findSVAPA rather than raw site quantification.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: umi_tools
  role: Extract barcodes/UMIs from read 1 and append them to read 2 headers; dedup
    removes PCR duplicates keeping one read per UMI per genomic coordinate.
- name: STAR
  role: Aligns read 2 to the GRCm38 reference genome; only uniquely mapped reads are
    retained.
- name: scAPAtrap
  role: Identifies poly(A) sites from the mapped ST reads, with 3' UTRs extended 1000
    bp to recruit intergenic sites.
- name: stAPAminer
  role: Provides computeAPAIndex to compute RUD APA usage ratios per spot from the
    PACdataset poly(A) site matrix; inputs loaded from package extdata (APA.RDA, count.csv,
    position.txt).
  repo: https://github.com/BMILAB/stAPAminer
- name: movAPA
  role: Supplies the PACdataset object class in which the poly(A) site matrix (APA.RDA)
    is stored.
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:f73121e08b45c6e36d27b2eadfe328108a2e8fa455dfb4dce78146f8e290dd2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - STAR
  - Seurat
  - movAPA
  - org.Mm.eg.db
  - scAPAtrap
  - stAPAminer
  - umi_tools
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# polya-site-quantification-in-spatial-transcriptomics

## Summary

A preprocessing-and-quantification skill that converts raw spatially barcoded ST reads (e.g., MOB ST data, SRA SRR3382373) into a poly(A) site-by-spot count matrix via scAPAtrap, then computes per-spot APA usage ratios (RUD) with stAPAminer's computeAPAIndex. It is the entry point of the stAPAminer pipeline before KNN imputation and spatial APA gene detection.

## When to use

Use when raw spatially barcoded ST FASTQ reads (barcodes/UMIs in read 1) must be turned into a poly(A) site-by-spot matrix and per-spot APA usage ratios (RUD) before imputation or layer-level APA analysis.

## When NOT to use

- The spatial assay is image-based (MERFISH/seqFISH) rather than spatially barcoded ST — the paper notes APA sites can only be identified from spatially barcoded ST data using tools like scAPAtrap or Sierra.
- A poly(A) site matrix (PACdataset) already exists — skip to imputation (imputeAPAIndex) and spatial clustering instead of re-running site identification.
- The goal is DEAPA/LSAPA/SVAPA gene calling from an already-imputed RUD matrix — use findContrastAPA/findAllMarkers/findSVAPA rather than raw site quantification.

## Inputs

- Raw spatially barcoded ST reads (read 1 with barcodes/UMIs, read 2 with sequence; e.g., SRA SRR3382373)
- GRCm38 mouse genome assembly and Ensembl gene annotation (TxDb.Mmusculus.UCSC.mm10.knownGene used in R)
- poly(A) site annotation object as a movAPA PACdataset (APA.RDA from stAPAminer extdata)
- Gene-by-spot expression count matrix (count.csv)
- Spot barcode coordinate table (position.txt)

## Outputs

- poly(A) site-by-spot count table (polyA_sites.tsv), e.g., 47,886 sites with an average of 14,436 sites per spot
- Raw APA usage ratio matrix (RUD_RAW) with rows as poly(A) sites and columns as spots
- Site region-distribution summary and site-level correlation statistics vs SC-MOB and bulk 3'-seq references

## How to apply

Extract barcodes and UMIs from read 1 with umi_tools and append them to read 2 headers; align read 2 to the GRCm38 mouse genome with STAR, retaining only uniquely mapped reads, then remove PCR duplicates with umi_tools dedup keeping one read per UMI per genomic coordinate. Identify poly(A) sites from the mapped reads with scAPAtrap, extending annotated 3' UTRs by 1000 bp to recruit intergenic sites and retaining only spots with ten or more read counts (260 spots retained in the ST-MOB example). Build the site-spot count table (polyA_sites.tsv, packaged as a movAPA PACdataset such as APA.RDA) and load it together with the gene-spot count matrix (count.csv) and spot coordinates (position.txt) into stAPAminer. Quantify per-spot APA usage as the RUD index by calling computeAPAIndex(APA, rownames(position)), yielding RUD_RAW, which feeds downstream KNN imputation (imputeAPAIndex, k=10). Validate site-level quantification by Pearson correlation against independent data: ~0.7 vs SC-MOB single-cell data and ~0.69 vs neural bulk 3'-seq data.

## Related tools

- **umi_tools** (Extract barcodes/UMIs from read 1 and append them to read 2 headers; dedup removes PCR duplicates keeping one read per UMI per genomic coordinate.)
- **STAR** (Aligns read 2 to the GRCm38 reference genome; only uniquely mapped reads are retained.)
- **scAPAtrap** (Identifies poly(A) sites from the mapped ST reads, with 3' UTRs extended 1000 bp to recruit intergenic sites.)
- **stAPAminer** (Provides computeAPAIndex to compute RUD APA usage ratios per spot from the PACdataset poly(A) site matrix; inputs loaded from package extdata (APA.RDA, count.csv, position.txt).) — https://github.com/BMILAB/stAPAminer
- **movAPA** (Supplies the PACdataset object class in which the poly(A) site matrix (APA.RDA) is stored.)

## Examples

```
load(system.file("extdata","APA.RDA",package="stAPAminer")); RUD_RAW <- computeAPAIndex(APA, rownames(position))
```

## Evaluation signals

- Total identified sites for ST-MOB should match the reported 47,886 poly(A) sites with an average of 14,436 sites per spot.
- Spot filter check: only spots with ten or more read counts are retained (262 → 260 spots).
- Site-level expression Pearson correlation vs SC-MOB single-cell data ≈ 0.7 and vs neural bulk 3'-seq data ≈ 0.69.
- RUD_RAW entries are APA usage ratios in [0,1]; matrix dimensions are poly(A) sites (rows) × retained spots (columns).
- Only uniquely mapped reads survive the STAR step, and one read per UMI per genomic coordinate survives umi_tools dedup.

## Limitations

- APA sites can only be identified from spatially barcoded ST data with existing tools such as scAPAtrap or Sierra; the approach does not extend to MERFISH/seqFISH.
- It is crucial to determine whether an entry in the APA ratio matrix is truly missing before imputation — no method is provided for this, and entries still missing after iterative imputation are set to 0.
- No version numbers are reported for the software tools used in the pipeline.

## Evidence

- [discussion] barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2: "barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2"
- [discussion] the 3′ UTRs of annotated genes were extended by 1000 bp to recruit intergenic poly(A) sites: "the 3′ UTRs of annotated genes were extended by 1000 bp to recruit intergenic poly(A) sites"
- [methods] RUD_RAW<-computeAPAIndex(APA,rownames(position)): "RUD_RAW<-computeAPAIndex(APA,rownames(position))"
- [readme] The input of stAPAminer is a poly(A) site matrix with each row being a poly(A) site and each column being a spot.: [evidence span withheld — anchor status: external_doc]
- [readme] APA sites can only be identified from the spatial barcoded ST data for now, using existing scRNA-seq tools like scAPAtrap or Sierra.: [evidence span withheld — anchor status: external_doc]
