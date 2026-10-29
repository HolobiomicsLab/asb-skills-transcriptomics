---
name: regulon-specificity-score-analysis
description: Use when you have an annotated plant single-cell atlas (e.g., Seurat
  object) and need to identify cell-type-specific gene regulatory networks and rank
  the top regulon transcription factor per cell cluster.
when_to_use_negative:
- Your goal is general cell-type annotation or deconvolution rather than gene regulatory
  network / regulon inference
- Your organism has no available cisTarget database and you cannot construct one (e.g.,
  no genome/gff3 or motif collection from JASPAR/PlantTFDB)
- Input is bulk RNA-seq data — scPlant's regulon module targets single-cell atlases;
  bulk data is handled by other modules such as decomposeBulk
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides RunGRN.sh workflow inputs and the R visualization functions ras_exp_hmp,
    topRegulons and SpecificityRank for regulon specificity analysis
  repo: https://github.com/compbioNJU/scPlant
- name: pySCENIC
  role: Executes the SCENIC pipeline (TF inference, regulon construction, AUCell/RSS
    scoring) invoked by RunGRN.sh
  repo: https://github.com/aertslab/pySCENIC
- name: Seurat
  role: Supplies the annotated single-cell object (SeuratObj.rds) that is the input
    to GRN construction
- name: SCopeLoomR
  role: R package dependency used in post-processing of the pySCENIC loom output
  repo: https://github.com/aertslab/SCopeLoomR
- name: create_cisTarget_databases
  role: Protocol/scripts for building species-specific cisTarget databases when precomputed
    ones are unavailable
  repo: https://github.com/aertslab/create_cisTarget_databases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:e4e278344a4b66260b145473a72babdaba8bd51a4dd8c8c45db5da5165fac465
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - SCopeLoomR
  - Seurat
  - create_cisTarget_databases
  - pySCENIC
  - scPlant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# regulon-specificity-score-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers and ranks cell-type-specific regulons from plant scRNA-seq data using pySCENIC with species-specific cisTarget databases, then scores regulon specificity per cluster (RAS/RSS) to identify the characteristic regulon TF for a cell type (e.g., PIF5 for photosynthetic cells in the Arabidopsis root atlas).

## When to use

Use when you have an annotated plant single-cell atlas (e.g., Seurat object) and need to identify cell-type-specific gene regulatory networks and rank the top regulon transcription factor per cell cluster.

## When NOT to use

- Your goal is general cell-type annotation or deconvolution rather than gene regulatory network / regulon inference
- Your organism has no available cisTarget database and you cannot construct one (e.g., no genome/gff3 or motif collection from JASPAR/PlantTFDB)
- Input is bulk RNA-seq data — scPlant's regulon module targets single-cell atlases; bulk data is handled by other modules such as decomposeBulk

## Inputs

- Annotated Seurat object saved as SeuratObj.rds (e.g., Arabidopsis root atlas from Zhang et al., 2019)
- Species-specific cisTarget databases (e.g., Arabidopsis thaliana TAIR10 from scPlantData.zip)
- RunGRN.sh wrapper script from the scPlant auxiliary bundle

## Outputs

- pyscenicOutput.loom
- AUCell.txt
- rasMat.rds (regulon activity score matrix)
- rssMat.rds (regulon specificity score matrix)
- tf_target.rds (TF-target links)
- regulons.gmt and regulons.txt
- binary_mtx.txt
- Visualizations: ras_exp_hmp heatmap, topRegulons RSS network, SpecificityRank dot plot per cluster

## How to apply

Download the scPlant auxiliary bundle (scPlantData.zip) from Figshare to obtain species cisTarget databases (e.g., Arabidopsis thaliana TAIR10) and the RunGRN.sh wrapper. Build the environment (conda python 3.8, pip install pyscenic==0.12.1; R packages optparse, Seurat, SCopeLoomR, data.table, pbapply, philentropy, dplyr), then run the full GRN construction with 'bash RunGRN.sh -f SeuratObj.rds -s At -t 2 -d cisTarget_databases -o output', which invokes pySCENIC and post-processing on a 2-thread setting. Confirm the declared outputs exist (pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt, binary_mtx.txt); intermediate files (adj.tsv, reg.tsv, exprMat.loom, cellmeta.rds, auc_thresholds.txt) can be ignored. Load rasMat.rds, rssMat.rds and tf_target.rds into R and visualize with scPlant's ras_exp_hmp (mean regulon activity and TF expression heatmap per cluster), topRegulons (network of top regulons per cluster by RSS) and SpecificityRank (dot plot of top regulons for a chosen cluster). Rank regulons by regulon specificity score (RSS) to determine the top regulon TF for the cell cluster of interest.

## Related tools

- **scPlant** (Provides RunGRN.sh workflow inputs and the R visualization functions ras_exp_hmp, topRegulons and SpecificityRank for regulon specificity analysis) — https://github.com/compbioNJU/scPlant
- **pySCENIC** (Executes the SCENIC pipeline (TF inference, regulon construction, AUCell/RSS scoring) invoked by RunGRN.sh) — https://github.com/aertslab/pySCENIC
- **Seurat** (Supplies the annotated single-cell object (SeuratObj.rds) that is the input to GRN construction)
- **SCopeLoomR** (R package dependency used in post-processing of the pySCENIC loom output) — https://github.com/aertslab/SCopeLoomR
- **create_cisTarget_databases** (Protocol/scripts for building species-specific cisTarget databases when precomputed ones are unavailable) — https://github.com/aertslab/create_cisTarget_databases

## Examples

```
bash RunGRN.sh -f SeuratObj.rds -s At -t 2 -d cisTarget_databases -o output
```

## Evaluation signals

- All declared output files exist in the output directory: pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt, binary_mtx.txt
- The RSS-based topRegulons network ranks the expected TF first for the target cluster (e.g., PIF5 first for photosynthetic cells, reproducing Figure 6A-6C)
- SpecificityRank dot plot for the photosynthetic cell cluster places PIF5 as the top regulon, consistent with its known role in photosynthesis and photoprotection
- ras_exp_hmp heatmap shows concordance between mean regulon activity and TF expression within clusters

## Limitations

- Requires precomputed species cisTarget databases; other plant genomes must be downloaded from EnsemblPlants or Phytozome and databases built via the create_cisTarget_databases protocol, which is memory intensive (RAM roughly 3x the database size)
- Regulon calls depend on motif databases such as JASPAR or PlantTFDB, so TFs lacking motif support will not be recovered
- Final confirmation of the top regulon TF is expert/visual (comparison against published figures), not a purely automated check

## Evidence

- [readme] pySCENIC pipeline description: [evidence span withheld — anchor status: external_doc]
