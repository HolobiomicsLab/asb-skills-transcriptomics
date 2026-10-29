---
name: regulon-activity-score-computation-aucell
description: Use when you have an annotated plant (e.g., Arabidopsis TAIR10) scRNA-seq
  Seurat object and need to infer cell-type-specific regulons/GRNs and identify top
  regulon TFs per cluster.
when_to_use_negative:
- The organism has no precomputed cisTarget database and no genome/gff3 available
  to build one (though custom databases can be built via aertslab/create_cisTarget_databases)
- The goal is bulk deconvolution, trajectory inference, or cross-species integration
  rather than GRN/regulon inference — use scPlant's decomposeBulk, RunMonocle2/3,
  RunSlingshot or crossSpecies_integrate instead
- Input is raw unprocessed counts without prior QC/clustering/annotation — run scPlant's
  readData, QCfun and RunSeurat first
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scPlant
  role: Pipeline framework providing RunGRN.sh workflow and R visualization functions
    (ras_exp_hmp, topRegulons, SpecificityRank) for the GRN/regulon outputs
  repo: github.com/compbioNJU/scPlant
- name: pySCENIC
  role: Python implementation of the SCENIC pipeline that infers TFs, gene regulatory
    networks and computes AUCell regulon activity scores from scRNA-seq data
  repo: github.com/aertslab/pySCENIC
- name: Seurat
  role: Provides the annotated Seurat object (RDS) that serves as GRN input and underlying
    processing methods
- name: SCopeLoomR
  role: R package used to read/handle the pySCENIC .loom output during post-processing
  repo: github.com/aertslab/SCopeLoomR
- name: create_cisTarget_databases
  role: Protocol/scripts for building species-specific cisTarget databases when precomputed
    ones are unavailable
  repo: github.com/aertslab/create_cisTarget_databases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:b8019f287f707b34b803b5e33ae9a4b6b02f6324bf01c88b0a51e66f8ed8e65d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  tools:
  - SCopeLoomR
  - Seurat
  - create_cisTarget_databases
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Regulon activity score computation (AUCell / SCENIC GRN) in plant scRNA-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute cell-type-specific regulon activity scores from an annotated plant single-cell atlas using pySCENIC's AUCell within the scPlant pipeline, ranking regulon TFs (e.g., RSS-based) per cell cluster to recover cell-type-specific gene regulatory networks.

## When to use

Use when you have an annotated plant (e.g., Arabidopsis TAIR10) scRNA-seq Seurat object and need to infer cell-type-specific regulons/GRNs and identify top regulon TFs per cluster.

## When NOT to use

- The organism has no precomputed cisTarget database and no genome/gff3 available to build one (though custom databases can be built via aertslab/create_cisTarget_databases)
- The goal is bulk deconvolution, trajectory inference, or cross-species integration rather than GRN/regulon inference — use scPlant's decomposeBulk, RunMonocle2/3, RunSlingshot or crossSpecies_integrate instead
- Input is raw unprocessed counts without prior QC/clustering/annotation — run scPlant's readData, QCfun and RunSeurat first

## Inputs

- Seurat object saved as .rds (annotated plant scRNA-seq atlas, e.g., Zhang et al. 2019 Arabidopsis root atlas)
- Species-matched cisTarget ranking databases (e.g., Arabidopsis thaliana TAIR10 from scPlantData.zip)
- TF list source (JASPAR / PlantTFDB motifs)

## Outputs

- pyscenicOutput.loom
- AUCell.txt (regulon activity scores)
- rasMat.rds (regulon activity matrix)
- rssMat.rds (regulon specificity score matrix)
- tf_target.rds (TF-target links)
- regulons.gmt and regulons.txt
- binary_mtx.txt (binarized regulon activity matrix)

## How to apply

Download the scPlant auxiliary bundle (scPlantData.zip) from Figshare to obtain species-specific cisTarget databases (e.g., Arabidopsis thaliana TAIR10) and the RunGRN.sh wrapper. Create a conda environment with python 3.8, install pySCENIC (pip install pyscenic==0.12.1), and the R packages optparse, Seurat, SCopeLoomR, data.table, pbapply, philentropy and dplyr. Run the full GRN construction with RunGRN.sh against the Seurat RDS input, passing species, thread count and cisTarget database directory; the wrapper invokes pySCENIC and post-processing. Verify the declared outputs (pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt, binary_mtx.txt), then load rasMat.rds/rssMat.rds/tf_target.rds in R with library(scPlant) and visualize via ras_exp_hmp, topRegulons and SpecificityRank. Rank regulons per cluster by regulon-specificity score (RSS); a run is validated when the expected files exist and the top-ranked regulon TF matches known biology (e.g., PIF5 first for photosynthetic cells, Figure 6A-6C).

## Related tools

- **scPlant** (Pipeline framework providing RunGRN.sh workflow and R visualization functions (ras_exp_hmp, topRegulons, SpecificityRank) for the GRN/regulon outputs) — github.com/compbioNJU/scPlant
- **pySCENIC** (Python implementation of the SCENIC pipeline that infers TFs, gene regulatory networks and computes AUCell regulon activity scores from scRNA-seq data) — github.com/aertslab/pySCENIC
- **Seurat** (Provides the annotated Seurat object (RDS) that serves as GRN input and underlying processing methods)
- **SCopeLoomR** (R package used to read/handle the pySCENIC .loom output during post-processing) — github.com/aertslab/SCopeLoomR
- **create_cisTarget_databases** (Protocol/scripts for building species-specific cisTarget databases when precomputed ones are unavailable) — github.com/aertslab/create_cisTarget_databases

## Examples

```
bash RunGRN.sh -f SeuratObj.rds -s At -t 2 -d cisTarget_databases -o output
```

## Evaluation signals

- All declared output files exist in the output directory: pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt, binary_mtx.txt
- RSS-based topRegulons network / SpecificityRank dot plot for the photosynthetic cell cluster ranks PIF5 first (reproducing Figure 6A-6C)
- ras_exp_hmp heatmap shows concordance between mean regulon activity and TF expression per cluster
- AUCell thresholding (auc_thresholds.txt intermediate) yields a binarized regulon activity matrix (binary_mtx.txt) with plausible per-cell regulon on/off calls
- Intermediate files adj.tsv, reg.tsv, exprMat.loom, cellmeta.rds, auc_thresholds.txt are produced, confirming pySCENIC coexpression/motif-enrichment stages completed

## Limitations

- Requires species-matched cisTarget databases; for non-model plants these must be built from genome/gff3 and motif sources (JASPAR/PlantTFDB) via create_cisTarget_databases
- Setting up the environment is non-trivial: a conda env with python 3.8 and pyscenic==0.12.1 plus seven specific R packages are required
- RSS-based top-regulon confirmation is expert/visual against published figures, not a fully automated check
- cisTarget database construction is memory intensive (roughly 3x the database RAM during building)

## Evidence

- [readme] pySCENIC is a lightning-fast python implementation of the SCENIC_ pipeline (Single-Cell rEgulatory Network Inference and Clustering): [evidence span withheld — anchor status: external_doc]
