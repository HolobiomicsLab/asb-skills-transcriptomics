---
name: spatial-spot-quality-filtering
description: Use when a spatially barcoded ST dataset (e.g., MOB ST from SRA SRR3382373,
  ~260–262 spots) is being prepared for poly(A) site identification with scAPAtrap
  and downstream stAPAminer APA usage quantification, and low-count spots must be
  excluded.
when_to_use_negative:
- The data are scRNA-seq or non-spatially barcoded ST (e.g., MERFISH/seqFISH), where
  APA sites cannot be identified with scAPAtrap/Sierra and per-spot filtering does
  not apply
- The spot set has already been QC-filtered upstream and spot coordinates/counts tables
  are final inputs to stAPAminer
- The task is filtering poly(A) sites (read support >= 3 reads, >= 3 spots) or genes
  (>= 2 3' UTR poly(A) sites), not spots
edam_operation: http://edamontology.org/operation_3678
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: Consumes the spot-filtered APA-site and count matrices; the filtered spot
    coordinates (position table) define rownames(position) used by computeAPAIndex
    and imputeAPAIndex
  repo: https://github.com/BMILAB/stAPAminer
- name: scAPAtrap
  role: Produces per-spot poly(A) site read counts from mapped ST reads, which supply
    the per-spot read-count metric used for spot filtering
- name: umi_tools
  role: Deduplication (one read per UMI per genomic coordinate) upstream of counting,
    so per-spot read counts are PCR-duplicate-free and the quality threshold is meaningful
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:07e1c813128cd7e75f0fbba312bf62a1fd47ce456c03397bcaad00c655e3871b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - STAR
  - scAPAtrap
  - stAPAminer
  - umi_tools
  edam_operation: http://edamontology.org/operation_3678
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-spot-quality-filtering

## Summary

Quality-filtering step for spatially barcoded ST data in which low-information spots are removed based on per-spot mapped read counts before APA/poly(A) site quantification, ensuring the stAPAminer APA ratio matrix and spatial gene expression matrix cover a reliable spot set.

## When to use

Use when a spatially barcoded ST dataset (e.g., MOB ST from SRA SRR3382373, ~260–262 spots) is being prepared for poly(A) site identification with scAPAtrap and downstream stAPAminer APA usage quantification, and low-count spots must be excluded.

## When NOT to use

- The data are scRNA-seq or non-spatially barcoded ST (e.g., MERFISH/seqFISH), where APA sites cannot be identified with scAPAtrap/Sierra and per-spot filtering does not apply
- The spot set has already been QC-filtered upstream and spot coordinates/counts tables are final inputs to stAPAminer
- The task is filtering poly(A) sites (read support >= 3 reads, >= 3 spots) or genes (>= 2 3' UTR poly(A) sites), not spots

## Inputs

- BAM/file of uniquely mapped, deduplicated ST reads (STAR-aligned read 2 with barcode/UMI-appended headers)
- Spot barcode-to-coordinate table (e.g., position.txt) from the spatially barcoded ST array
- scAPAtrap poly(A) site calls with per-spot read counts (polyA_sites.tsv)

## Outputs

- Filtered spot list (spots with >= 10 read counts; 260 spots for ST-MOB)
- Spot-filtered gene expression count matrix (gene x spot)
- Spot-filtered APA/poly(A) site matrix (poly(A) site x spot) used by computeAPAIndex

## How to apply

After aligning read 2 to the GRCm38 genome with STAR, removing PCR duplicates with umi_tools dedup, and identifying poly(A) sites with scAPAtrap (extending annotated 3' UTRs by 1000 bp), compute per-spot total read counts from the mapped reads. Retain only spots with ten or more read counts — in the ST-MOB analysis this retained 260 of 262 spots. Use the filtered spot set as the column universe for both the poly(A) site expression table (polyA_sites.tsv) and the gene-spot count matrix that stAPAminer uses for KNN-based imputation (imputeAPAIndex with k=10), so APA ratios and spatial expression profiles are computed over identical spots. The rationale is that spots with very few reads carry unreliable APA usage estimates and would corrupt the imputation and spatial pattern detection.

## Related tools

- **stAPAminer** (Consumes the spot-filtered APA-site and count matrices; the filtered spot coordinates (position table) define rownames(position) used by computeAPAIndex and imputeAPAIndex) — https://github.com/BMILAB/stAPAminer
- **scAPAtrap** (Produces per-spot poly(A) site read counts from mapped ST reads, which supply the per-spot read-count metric used for spot filtering)
- **umi_tools** (Deduplication (one read per UMI per genomic coordinate) upstream of counting, so per-spot read counts are PCR-duplicate-free and the quality threshold is meaningful)

## Examples

```
count<-read.csv(system.file("extdata", "count.csv", package = "stAPAminer")); keep <- colnames(count[,-1])[colSums(count[,-1]) >= 10]  # retain spots with >= 10 read counts (260 spots in ST-MOB)
```

## Evaluation signals

- Applying the >= 10 read-count threshold to the ST-MOB data should yield exactly 260 retained spots (from 262)
- The retained spot identifiers must match the rownames of the position/coordinate table and the columns of both the count matrix and the APA matrix passed to stAPAminer
- Per-spot read counts after filtering should all be >= 10 and spot coordinates should remain spatially coherent across the array, not randomly scattered
- Downstream validation should reproduce expected correlations of site-level expression: ~0.7 Pearson correlation against SC-MOB and ~0.69 against neural bulk 3'-seq data

## Limitations

- A fixed read-count threshold (10) is dataset-specific and may need recalibration for other tissues or ST platforms
- No dedicated criterion exists for whether an APA ratio matrix entry is truly missing; after spot filtering, residual missing APA usage entries are arbitrarily set to 0 after imputation iterations
- Discrepancies among ST replicates are not mitigated by this filtering, which can limit reproducibility of spatial APA patterns
- Software version numbers for the filtering/counting steps were not reported in the article

## Evidence

- [discussion] We retained spots with ten or more read counts, resulting in 260 spots: "We retained spots with ten or more read counts, resulting in 260 spots"
- [readme] The input of stAPAminer is a poly(A) site matrix with each row being a poly(A) site and each column being a spot.: [evidence span withheld — anchor status: external_doc]
- [methods] position<-read.table(system.file("extdata", "position.txt", package = "stAPAminer")): "position<-read.table(system.file("extdata", "position.txt", package = "stAPAminer"))"
