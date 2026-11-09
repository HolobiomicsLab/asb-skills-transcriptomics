---
name: secondary-metabolite-class-profiling
description: Use when a comparative-genomics pipeline needs BGC-level absence/presence
  features across many bacterial genomes — e.g. annotating .gbk files with Prokka
  then mining secondary metabolites with antiSMASH and BiG-SCAPE before lifestyle
  association.
when_to_use_negative:
- The input is already a binary feature matrix (e.g. a gene-cluster absence/presence
  table) ready for lifestyle prediction or machine learning — the BGC mining step
  would be redundant.
- Genomes have not passed quality filtering (CheckM >90% completeness, <5% contamination)
  or redundancy reduction at 0.99 Mash-estimated ANI — antiSMASH/BiG-SCAPE would be
  run on low-quality or near-duplicate assemblies.
- The goal is lifestyle prediction itself (random-forest classifier with >0.8 confidence
  threshold) rather than secondary-metabolite profiling — use the lifestyle-prediction
  module instead.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Prokka 1.14.6
  role: Gene prediction producing the .gbk files required as antiSMASH input
- name: antiSMASH 5.1.2
  role: Predicts BGCs per .gbk file with clusterblast flags --cb-general --cb-knownclusters
    --cb-subclusters
- name: BiG-SCAPE 1.1.5
  role: Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance
    cutoff
- name: Snakemake
  role: Workflow engine for the bacLIFE clustering module rules that generate the
    GCF absence/presence matrix
- name: Python
  role: Custom scripts within the Snakemake module that write gcf_absence_presence_matrix.tsv
    and bgc_class_tally.csv
- name: bacLIFE
  role: Overall workflow whose clustering module integrates the antiSMASH + BiG-SCAPE
    BGC analysis
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:12a410c852aa8223fd06a436727bbf9d1e390662cbf00a572a39e82da36ffcaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - BiG-SCAPE
  - BiG-SCAPE 1.1.5
  - Prokka
  - Prokka 1.14.6
  - Python
  - Snakemake
  - antiSMASH
  - antiSMASH 5.1.2
  - bacLIFE
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# secondary-metabolite-class-profiling

## Summary

Reconstructs the bacLIFE secondary-metabolite analysis in which BGCs predicted per genome by antiSMASH are clustered into Gene Cluster Families (GCFs) with BiG-SCAPE at a 0.70 distance cutoff, yielding a binary GCF absence/presence matrix and per-genome BGC-class tallies.

## When to use

Use when a comparative-genomics pipeline needs BGC-level absence/presence features across many bacterial genomes — e.g. annotating .gbk files with Prokka then mining secondary metabolites with antiSMASH and BiG-SCAPE before lifestyle association.

## When NOT to use

- The input is already a binary feature matrix (e.g. a gene-cluster absence/presence table) ready for lifestyle prediction or machine learning — the BGC mining step would be redundant.
- Genomes have not passed quality filtering (CheckM >90% completeness, <5% contamination) or redundancy reduction at 0.99 Mash-estimated ANI — antiSMASH/BiG-SCAPE would be run on low-quality or near-duplicate assemblies.
- The goal is lifestyle prediction itself (random-forest classifier with >0.8 confidence threshold) rather than secondary-metabolite profiling — use the lifestyle-prediction module instead.

## Inputs

- Non-redundant bacterial genome assemblies (e.g. RefSeq assemblies from Supplementary Datasets 1, quality-filtered with CheckM to >90% completeness and <5% contamination)
- Prokka 1.14.6 annotation output (.gbk files per genome)
- antiSMASH 5.1.2 BGC predictions per .gbk file

## Outputs

- gcf_absence_presence_matrix.tsv — rows = genomes, columns = GCFs, binary values at the 0.70 BiG-SCAPE distance cutoff
- Per-genome BGC families clustered by BiG-SCAPE 1.1.5 (856 GCFs for Burkholderia/Paraburkholderia; 1,279 for Pseudomonas)
- bgc_class_tally.csv — per-genome counts of predicted BGC classes (e.g. NRPS, PKS)

## How to apply

Annotate each non-redundant genome assembly with Prokka 1.14.6 to generate the .gbk files required as antiSMASH input, then run antiSMASH 5.1.2 with the clusterblast flags --cb-general --cb-knownclusters --cb-subclusters to predict BGCs (expected ~12,474 BGCs across ~14 per genome for Burkholderia/Paraburkholderia and ~19,087 BGCs across ~9 per genome for Pseudomonas). Feed all predicted BGCs to BiG-SCAPE 1.1.5 to cluster them into Gene Cluster Families (expected 856 GCFs for Burkholderia/Paraburkholderia and 1,279 for Pseudomonas). Within the Snakemake-based bacLIFE clustering module, custom Python scripts then generate the absence/presence matrix of individual GCFs at the 0.70 BiG-SCAPE distance cutoff, saved as gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs, binary values). Tally predicted BGC classes per genome across both genera into bgc_class_tally.csv; in this study NRPS was the most abundant BGC class in Pseudomonas (Supplementary Fig. 17a).

## Related tools

- **Prokka 1.14.6** (Gene prediction producing the .gbk files required as antiSMASH input)
- **antiSMASH 5.1.2** (Predicts BGCs per .gbk file with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters)
- **BiG-SCAPE 1.1.5** (Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance cutoff)
- **Snakemake** (Workflow engine for the bacLIFE clustering module rules that generate the GCF absence/presence matrix)
- **Python** (Custom scripts within the Snakemake module that write gcf_absence_presence_matrix.tsv and bgc_class_tally.csv)
- **bacLIFE** (Overall workflow whose clustering module integrates the antiSMASH + BiG-SCAPE BGC analysis) — https://github.com/Carrion-lab/bacLIFE

## Evaluation signals

- gcf_absence_presence_matrix.tsv exists and contains only binary (0/1) values with genomes as rows and GCFs as columns
- Total predicted BGCs match reported values: 12,474 for Burkholderia/Paraburkholderia and 19,087 for Pseudomonas
- Total GCFs match reported values: 856 for Burkholderia/Paraburkholderia and 1,279 for Pseudomonas
- bgc_class_tally.csv confirms NRPS as the most abundant BGC class in Pseudomonas, as reported in Supplementary Fig. 17a
- GCF clustering was performed at the 0.70 BiG-SCAPE distance cutoff, consistent with the method description

## Limitations

- GCF counts and class abundances are specific to the 0.70 BiG-SCAPE distance cutoff; other cutoffs will change the absence/presence matrix granularity.
- BGC totals depend on the genome set having undergone redundancy reduction at 0.99 Mash-estimated ANI and CheckM quality filtering (>90% completeness, <5% contamination); unfiltered inputs will not reproduce the reported 12,474 / 19,087 BGC counts.
- The 0.70-cutoff matrix is generated by custom scripts inside the Snakemake-based bacLIFE module, so reproducing it requires the bacLIFE repository rather than standalone antiSMASH/BiG-SCAPE runs.

## Evidence

- [methods] an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [methods] Gene prediction was performed using Prokka 1.14.6: "Gene prediction was performed using Prokka 1.14.6"
- [readme] bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes"
- [abstract] a streamlined computational workflow for genome annotation, large-scale comparative genomics, and prediction of lifestyle-associated genes (LAGs): "a streamlined computational workflow for genome annotation, large-scale comparative genomics, and prediction of lifestyle-associated genes (LAGs)"
