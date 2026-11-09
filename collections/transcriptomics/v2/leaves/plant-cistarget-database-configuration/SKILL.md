---
name: plant-cistarget-database-configuration
description: Use when building plant cell-type-specific GRNs with scPlant/pySCENIC
  from an annotated Seurat object and you need to obtain or build the required cisTarget
  databases and aux bundle before running GRN inference.
when_to_use_negative:
- The organism already has precomputed cisTarget databases for human, mouse or fly
  — download directly from resources.aertslab.org instead of building plant databases
- The goal is a different GRN method (e.g. MINI-EX or PMET motif-pair enrichment)
  rather than the scPlant/pySCENIC cisTarget-based workflow
- Input is not an annotated single-cell Seurat object (e.g. bulk expression table
  or already-computed regulon activity matrix)
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Provides RunGRN.sh, the auxiliary scPlantData.zip bundle with species cisTarget
    databases, and R post-processing/visualization (ras_exp_hmp, topRegulons, SpecificityRank)
  repo: https://github.com/compbioNJU/scPlant
- name: pySCENIC
  role: Executes the SCENIC GRN pipeline (motif enrichment against the configured
    cisTarget databases) invoked by RunGRN.sh
  repo: https://github.com/aertslab/pySCENIC
- name: create_cisTarget_databases
  role: Protocol/scripts to build custom plant cisTarget motif databases when no precomputed
    bundle exists
  repo: https://github.com/aertslab/create_cisTarget_databases
- name: SCopeLoomR
  role: R package dependency for reading the pySCENIC loom output during scPlant post-processing
  repo: https://github.com/aertslab/SCopeLoomR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:2bed5f603b1c25922daf142a100770fb75ca58080e005b7f15d7cf2df61d2d48
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
  - create_cisTarget_databases
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# plant-cistarget-database-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure species-specific cisTarget motif/ranking databases and the pySCENIC execution environment needed by scPlant's cell-type-specific gene regulatory network (GRN) module for plant scRNA-seq data, e.g. Arabidopsis thaliana TAIR10.

## When to use

Use when building plant cell-type-specific GRNs with scPlant/pySCENIC from an annotated Seurat object and you need to obtain or build the required cisTarget databases and aux bundle before running GRN inference.

## When NOT to use

- The organism already has precomputed cisTarget databases for human, mouse or fly — download directly from resources.aertslab.org instead of building plant databases
- The goal is a different GRN method (e.g. MINI-EX or PMET motif-pair enrichment) rather than the scPlant/pySCENIC cisTarget-based workflow
- Input is not an annotated single-cell Seurat object (e.g. bulk expression table or already-computed regulon activity matrix)

## Inputs

- Seurat object saved as SeuratObj.rds (annotated plant scRNA-seq clusters)
- scPlantData.zip auxiliary bundle containing species-specific cisTarget databases and RunGRN.sh
- Genome FASTA + gff3 annotation (e.g. TAIR10 for Arabidopsis thaliana) if building custom cisTarget databases
- PWM motif collection (JASPAR or PlantTFDB) for custom database creation

## Outputs

- pyscenicOutput.loom (SCENIC loom with regulons)
- AUCell.txt, rasMat.rds (regulon activity matrix), rssMat.rds (regulon specificity scores)
- tf_target.rds, regulons.gmt, regulons.txt, binary_mtx.txt
- Custom cisTarget motif/ranking feather databases (when built from scratch)

## How to apply

First download the scPlant auxiliary bundle (scPlantData.zip) from Figshare and unzip it to obtain the precomputed Arabidopsis thaliana (TAIR10) cisTarget databases plus the RunGRN.sh wrapper script; for other plant organisms, download the genome and gff3 files from EnsemblPlants or Phytozome and build custom cisTarget databases following the aertslab/create_cisTarget_databases protocol (motif collections can come from JASPAR or PlantTFDB), remembering the 2D memory footprint (3-4 bytes x regions/genes x motifs/tracks, plus ~3x RAM during construction). Next, set up a conda environment with python 3.8 and pySCENIC (pip install pyscenic==0.12.1), plus R packages optparse, Seurat, SCopeLoomR, data.table, pbapply, philentropy and dplyr. Then launch the full GRN run via RunGRN.sh, passing the Seurat .rds object, the species short name (e.g. 'At' for Arabidopsis thaliana), thread count and the cisTarget database directory; scPlant post-processes the pySCENIC results into loom/RDS/GMT outputs consumed by ras_exp_hmp, topRegulons and SpecificityRank.

## Related tools

- **scPlant** (Provides RunGRN.sh, the auxiliary scPlantData.zip bundle with species cisTarget databases, and R post-processing/visualization (ras_exp_hmp, topRegulons, SpecificityRank)) — https://github.com/compbioNJU/scPlant
- **pySCENIC** (Executes the SCENIC GRN pipeline (motif enrichment against the configured cisTarget databases) invoked by RunGRN.sh) — https://github.com/aertslab/pySCENIC
- **create_cisTarget_databases** (Protocol/scripts to build custom plant cisTarget motif databases when no precomputed bundle exists) — https://github.com/aertslab/create_cisTarget_databases
- **SCopeLoomR** (R package dependency for reading the pySCENIC loom output during scPlant post-processing) — https://github.com/aertslab/SCopeLoomR

## Examples

```
bash RunGRN.sh -f SeuratObj.rds -s At -t 2 -d cisTarget_databases -o output
```

## Evaluation signals

- All declared output files exist after the run: pyscenicOutput.loom, AUCell.txt, rasMat.rds, rssMat.rds, tf_target.rds, regulons.gmt, regulons.txt and binary_mtx.txt (intermediates adj.tsv, reg.tsv, exprMat.loom, cellmeta.rds, auc_thresholds.txt can be ignored)
- rasMat.rds, rssMat.rds and tf_target.rds load cleanly in R with library(scPlant) and render via ras_exp_hmp / topRegulons / SpecificityRank
- Top-regulon RSS ranking for the photosynthetic cell cluster of the Zhang et al. 2019 Arabidopsis root atlas places PIF5 first, reproducing Figure 6A-6C
- If building custom databases, the cisTarget rankings database memory check passes: 2 bytes x genes x motifs for <=32768 genes, else 4 bytes x genes x motifs, with ~3x RAM available
- Conda environment resolves pyscenic==0.12.1 under python 3.8 and the R dependency set (optparse, Seurat, SCopeLoomR, data.table, pbapply, philentropy, dplyr)

## Limitations

- Precomputed species bundles are limited — non-model plants require downloading genome/gff3 from EnsemblPlants or Phytozome and building custom cisTarget databases, which is memory-intensive (~3x RAM of the database)
- cisTarget database construction creates a 2D matrix of motifs vs regions/genes; large plant genomes can exceed available RAM
- Pipeline is validated against specific versions (python 3.8, pyscenic==0.12.1); other versions are untested
- Final GRN quality confirmation is expert/visual (Figure 6A-6C comparison), not fully automated

## Evidence

- [readme] cisTarget database memory requirements from README: [evidence span withheld — anchor status: external_doc]
