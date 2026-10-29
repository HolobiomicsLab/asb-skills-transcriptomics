---
name: plant-transcription-factor-motif-pair-enrichment
description: Use when you have Seurat cluster-specific marker genes (e.g. DEG.txt
  for the photosynthetic cluster) from a plant scRNA-seq atlas and want to find which
  paired TF-binding motifs are enriched in their promoters.
when_to_use_negative:
- You need single-motif (not paired-motif) enrichment or TF regulon inference — use
  SCENIC/pySCENIC or MINI-EX instead
- Your organism's genome and gff3 are not available through the supported EnsemblPlants/Phytozome
  sources and no PMET index database exists
- You have no cluster-specific marker gene set (DEG.txt) and only raw scRNA-seq counts
  — basic scPlant processing must be done first
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: scPlant
  role: Provides processPMET, PMEThmp, triHmp and topPairsNet for processing and visualizing
    PMET motif-pair enrichment results; part of its cell-type-specific gene regulatory
    network module
  repo: https://github.com/compbioNJU/scPlant
- name: PMET
  role: Paired Motif Enrichment Tool that assigns motifs to promoters with fimo and
    scores motif-pair co-localisation via a hypergeometric test (RunPMET.sh)
  repo: https://github.com/kate-wa/PMET-software-python
- name: Seurat
  role: Identifies cluster-specific differentially expressed marker genes (pct.1 >
    pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25) used as the PMET gene input
- name: pySCENIC
  role: Provides regulon TFs on which the PMET paired motif enrichment is run in the
    scPlant workflow
  repo: https://github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:fb5b6388a3c4e036f3c75758a8fe8bf22c9cb6b5b4b1add184c277a2780c38f7
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
  - PMET
  - Seurat
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# plant-transcription-factor-motif-pair-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run PMET (Paired Motif Enrichment Test) on cluster-specific differentially expressed genes from a plant single-cell atlas to detect co-localized pairs of transcription factor binding motifs in promoters, as implemented in the scPlant cell-type-specific gene regulatory network workflow.

## When to use

Use when you have Seurat cluster-specific marker genes (e.g. DEG.txt for the photosynthetic cluster) from a plant scRNA-seq atlas and want to find which paired TF-binding motifs are enriched in their promoters.

## When NOT to use

- You need single-motif (not paired-motif) enrichment or TF regulon inference — use SCENIC/pySCENIC or MINI-EX instead
- Your organism's genome and gff3 are not available through the supported EnsemblPlants/Phytozome sources and no PMET index database exists
- You have no cluster-specific marker gene set (DEG.txt) and only raw scRNA-seq counts — basic scPlant processing must be done first

## Inputs

- 2-column tab-delimited DEG.txt (cluster ID, gene) of cluster-specific marker genes from the Arabidopsis root atlas (Zhang et al., 2019), filtered with pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25
- PMET databases and scripts (scPlantData.zip from the scPlant Figshare deposit) placed in scPlantData/PMET/
- Arabidopsis thaliana TAIR10 genome FASTA and gff3 annotation from EnsemblPlants release 55
- Installed dependencies: samtools, bedtools, meme, parallel

## Outputs

- output/output.txt raw PMET result table of enriched motif pairs per cluster
- PMETresult table from scPlant processPMET
- Heatmap (PMEThmp, topn = 5), triangle heatmap (triHmp) per cluster, and topPairsNet network of the top 20 motif pairs (e.g. TCP14/TCP15 and ABI5-ABF for cluster 23)

## How to apply

Prepare the gene input from the Arabidopsis root atlas (Zhang et al., 2019) by extracting cluster-specific DEGs with Seurat marker filters pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25 and avg_logFC > 0.25, written as a 2-column tab-delimited DEG.txt (cluster ID in column 1, gene in column 2). Download the PMET databases and scripts (scPlantData.zip from the scPlant Figshare deposit) plus the Arabidopsis thaliana TAIR10 genome FASTA and gff3 from EnsemblPlants release 55 into scPlantData/PMET/, and verify samtools, bedtools, meme and parallel are installed. Run the Paired Motif Enrichment Test with 'bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At' (step 2 of the script is time-consuming); PMET assigns motifs to promoters with fimo and assesses motif-pair co-localisation with a hypergeometric test. Post-process output/output.txt with the scPlant function processPMET, then visualize with PMEThmp (topn = 5), the triangle heatmap triHmp for cluster 23, and topPairsNet for the top 20 motif pairs of the cluster.

## Related tools

- **scPlant** (Provides processPMET, PMEThmp, triHmp and topPairsNet for processing and visualizing PMET motif-pair enrichment results; part of its cell-type-specific gene regulatory network module) — https://github.com/compbioNJU/scPlant
- **PMET** (Paired Motif Enrichment Tool that assigns motifs to promoters with fimo and scores motif-pair co-localisation via a hypergeometric test (RunPMET.sh)) — https://github.com/kate-wa/PMET-software-python
- **Seurat** (Identifies cluster-specific differentially expressed marker genes (pct.1 > pct.2, p_val < 0.05, pct.1 > 0.25, avg_logFC > 0.25) used as the PMET gene input)
- **pySCENIC** (Provides regulon TFs on which the PMET paired motif enrichment is run in the scPlant workflow) — https://github.com/aertslab/pySCENIC

## Examples

```
bash RunPMET.sh -r ./PMET -o output -m DEG.txt -s At
```

## Evaluation signals

- output/output.txt exists and contains PMET results for cluster 23
- The triangle heatmap (triHmp) for cluster 23 reproduces the TCP14/TCP15 motif-pair enrichment reported in the paper's Figure 6G
- The ABI5-ABF motif pairing appears as an enriched pair for the photosynthetic cluster
- The top-scoring TCP-family motif pairing is specific to cluster 23 and not enriched in other clusters' promoters
- DEG.txt strictly follows the 2-column tab-delimited format (cluster ID, gene) and marker thresholds were applied

## Limitations

- Step 2 of RunPMET.sh (building/using the PMET index over the genome) is time-consuming; indexing requires samtools, bedtools, meme and GNU parallel to be installed
- The PMET databases must match the species (only species At for A. thaliana was exercised here; other organisms need EnsemblPlants/Phytozome genome and gff3 downloads)
- PMET assesses co-localisation of motif pairs, not individual motif enrichment or TF activity, so results depend on motif collection quality (e.g. JASPAR/PlantTFDB-derived motifs)

## Evidence

- [readme] A tool for determining the co-localisation of pairs of transcription factor binding motifs within a given set of promoters using a hypergeometric test.: [evidence span withheld — anchor status: external_doc]
