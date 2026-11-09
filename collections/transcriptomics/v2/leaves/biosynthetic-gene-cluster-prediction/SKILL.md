---
name: biosynthetic-gene-cluster-prediction
description: Use when you have annotated genome assemblies (.gbk) across a genus-scale
  dataset and need BGC predictions, GCF clustering at a defined distance cutoff, and
  a binary GCF-by-genome matrix for lifestyle association testing.
when_to_use_negative:
- Input is raw reads or unannotated assemblies without a prior gene-annotation step
  producing .gbk files, since antiSMASH requires .gbk input
- The analysis goal is lifestyle prediction from gene clusters via machine learning
  (bacLIFE's lifestyle-prediction module), not secondary-metabolite BGC/GCF analysis
- Only a small number of genomes is available and a genus-scale comparative BGC matrix
  is unnecessary
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Snakemake-based workflow containing the clustering module that orchestrates
    BGC prediction, GCF clustering, and GCF matrix generation
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Prokka 1.14.6
  role: Performs gene prediction/annotation producing the .gbk files required as antiSMASH
    input
- name: antiSMASH 5.1.2
  role: Predicts BGCs from each .gbk file with clusterblast flags --cb-general --cb-knownclusters
    --cb-subclusters
- name: BiG-SCAPE 1.1.5
  role: Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance
    cutoff
- name: Snakemake
  role: Workflow engine of the bacLIFE first module that processes, annotates and
    generates the data
- name: CheckM
  role: Quality filter retaining only high-quality genomes (>90% completeness, <5%
    contamination) before BGC analysis
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:f8856be46f9d512f7bbdd1971e1ebc75173f5be8a1cf696a16a4bea6db47456e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - BiG-SCAPE
  - BiG-SCAPE 1.1.5
  - CheckM
  - Prokka
  - Prokka 1.14.6
  - Python
  - Snakemake
  - antiSMASH
  - antiSMASH 5.1.2
  - bacLIFE
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# biosynthetic-gene-cluster-prediction

## Summary

Predict and cluster biosynthetic gene clusters (BGCs) from annotated bacterial genomes into Gene Cluster Families (GCFs) and build a GCF absence/presence matrix, as implemented in the bacLIFE clustering module with antiSMASH 5.1.2 and BiG-SCAPE 1.1.5.

## When to use

Use when you have annotated genome assemblies (.gbk) across a genus-scale dataset and need BGC predictions, GCF clustering at a defined distance cutoff, and a binary GCF-by-genome matrix for lifestyle association testing.

## When NOT to use

- Input is raw reads or unannotated assemblies without a prior gene-annotation step producing .gbk files, since antiSMASH requires .gbk input
- The analysis goal is lifestyle prediction from gene clusters via machine learning (bacLIFE's lifestyle-prediction module), not secondary-metabolite BGC/GCF analysis
- Only a small number of genomes is available and a genus-scale comparative BGC matrix is unnecessary

## Inputs

- Non-redundant genome assemblies (NCBI RefSeq assemblies, clustered at 0.99 Mash-estimated ANI)
- Prokka 1.14.6 annotation files (.gbk) per genome
- Lifestyle metadata annotations collected from the literature (for downstream association testing)

## Outputs

- gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs, binary values)
- bgc_class_tally.csv (per-genome counts of predicted BGC classes)
- Predicted BGCs per genome from antiSMASH 5.1.2
- GCF assignments from BiG-SCAPE 1.1.5 at 0.70 distance cutoff

## How to apply

First annotate the non-redundant genome assemblies (e.g., after 0.99 Mash-ANI redundancy reduction and CheckM >90% completeness / <5% contamination filtering) with Prokka 1.14.6 to produce .gbk files. Run antiSMASH 5.1.2 on every .gbk file with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters to predict BGCs (~12,474 BGCs / ~14 per genome for Burkholderia/Paraburkholderia; ~19,087 BGCs / ~9 per genome for Pseudomonas). Cluster all predicted BGCs into Gene Cluster Families with BiG-SCAPE 1.1.5 at the 0.70 distance cutoff (856 GCFs for Burkholderia/Paraburkholderia; 1279 GCFs for Pseudomonas). Using custom Python scripts within the Snakemake-based bacLIFE module, generate the absence/presence matrix of individual GCFs saved as gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs, binary values), and tally BGC classes per genome as bgc_class_tally.csv.

## Related tools

- **bacLIFE** (Snakemake-based workflow containing the clustering module that orchestrates BGC prediction, GCF clustering, and GCF matrix generation) — https://github.com/Carrion-lab/bacLIFE
- **Prokka 1.14.6** (Performs gene prediction/annotation producing the .gbk files required as antiSMASH input)
- **antiSMASH 5.1.2** (Predicts BGCs from each .gbk file with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters)
- **BiG-SCAPE 1.1.5** (Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance cutoff)
- **Snakemake** (Workflow engine of the bacLIFE first module that processes, annotates and generates the data)
- **CheckM** (Quality filter retaining only high-quality genomes (>90% completeness, <5% contamination) before BGC analysis)

## Examples

```
bacLIFE clustering module: antiSMASH 5.1.2 --cb-general --cb-knownclusters --cb-subclusters <genome>.gbk; BiG-SCAPE 1.1.5 --distance_cutoff 0.70 -> gcf_absence_presence_matrix.tsv
```

## Evaluation signals

- gcf_absence_presence_matrix.tsv exists and is binary with rows = genomes and columns = GCFs
- Predicted BGC totals match reported values: 12,474 BGCs (~14 per genome) for Burkholderia/Paraburkholderia and 19,087 BGCs (~9 per genome) for Pseudomonas
- GCF totals at the 0.70 BiG-SCAPE distance cutoff match reported values: 856 GCFs (Burkholderia/Paraburkholderia) and 1279 GCFs (Pseudomonas)
- bgc_class_tally.csv confirms NRPS is the most abundant BGC class in Pseudomonas as reported in Supplementary Fig. 17a
- Downstream Fisher's exact test on the matrix recovers the syringofactin and syringomycin NRPS families and a novel NRPS-PKS hybrid cluster at P < 0.01 (Benjamini-Hochberg), with >15% group presence

## Limitations

- Results are calibrated on genus-scale datasets of Burkholderia/Paraburkholderia and Pseudomonas (16,846 genomes); other taxa may yield different BGC/GCF counts
- GCF associations are restricted to clusters meeting a >15% group presence threshold, so rare clusters cannot be flagged
- Genomes must pass quality filtering (>90% completeness, <5% contamination) and 0.99 ANI redundancy reduction before the BGC step to avoid redundant or low-quality predictions

## Evidence

- [methods] Prokka produces antiSMASH input: "Gene prediction was performed using Prokka 1.14.688"
- [methods] GCF absence/presence matrix at 0.70 distance: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [methods] Genome quality filter: "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [readme] bacLIFE modules: "a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes"
- [results] BGC association finding: [evidence span withheld — anchor status: not_found]
