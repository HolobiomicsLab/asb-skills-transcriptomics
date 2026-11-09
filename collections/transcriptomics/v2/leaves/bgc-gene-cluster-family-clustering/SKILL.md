---
name: bgc-gene-cluster-family-clustering
description: Use when you need biosynthetic gene cluster (BGC) detection and Gene
  Cluster Family-level absence/presence encoding across many bacterial genomes, as
  a comparative-genomics feature matrix.
when_to_use_negative:
- The goal is gene/ortholog-level clustering (linclust + MCL gene families) rather
  than secondary-metabolite BGC family detection — that is a separate bacLIFE module.
- Inputs are raw reads or unannotated contigs; antiSMASH requires Prokka-annotated
  .gbk files.
- You only need lifestyle prediction (random forest, >0.8 confidence threshold) rather
  than BGC/GCF feature matrices.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Prokka 1.14.6
  role: Gene prediction producing the .gbk files required as antiSMASH input
- name: antiSMASH 5.1.2
  role: Predicts BGCs per genome with clusterblast flags --cb-general --cb-knownclusters
    --cb-subclusters
- name: BiG-SCAPE 1.1.5
  role: Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance
    cutoff
- name: bacLIFE
  role: Snakemake-based module wrapping the annotation, BGC prediction, GCF clustering,
    and matrix generation with custom scripts
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Snakemake
  role: Workflow engine that processes, annotates and generates the data in the correct
    format for the clustering module
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:6fc23ff773962fbf8d65296ff13e700123b5a600590441298ae0431cd9ccadcb
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# bgc-gene-cluster-family-clustering

## Summary

Reconstructs bacLIFE's secondary-metabolite analysis: annotate non-redundant assemblies with Prokka, predict BGCs with antiSMASH 5.1.2, cluster them into Gene Cluster Families with BiG-SCAPE 1.1.5, and emit a binary GCF absence/presence matrix plus per-genome BGC-class counts.

## When to use

Use when you need biosynthetic gene cluster (BGC) detection and Gene Cluster Family-level absence/presence encoding across many bacterial genomes, as a comparative-genomics feature matrix.

## When NOT to use

- The goal is gene/ortholog-level clustering (linclust + MCL gene families) rather than secondary-metabolite BGC family detection — that is a separate bacLIFE module.
- Inputs are raw reads or unannotated contigs; antiSMASH requires Prokka-annotated .gbk files.
- You only need lifestyle prediction (random forest, >0.8 confidence threshold) rather than BGC/GCF feature matrices.

## Inputs

- Non-redundant bacterial genome assemblies (e.g. NCBI RefSeq assemblies, ≥90% completeness and <5% contamination per CheckM)
- .gbk annotation files produced by Prokka 1.14.6 (antiSMASH input)

## Outputs

- gcf_absence_presence_matrix.tsv — binary matrix, rows = genomes, columns = individual GCFs at 0.70 BiG-SCAPE distance
- bgc_class_tally.csv — per-genome counts of predicted BGC classes (e.g. NRPS)
- BiG-SCAPE GCF cluster files (856 GCFs for Burkholderia/Paraburkholderia; 1279 for Pseudomonas)

## How to apply

Annotate the non-redundant genome assemblies (Burkholderia/Paraburkholderia and Pseudomonas sets) with Prokka 1.14.6 to produce the .gbk files required as antiSMASH input. Run antiSMASH 5.1.2 on every .gbk file with the clusterblast flags --cb-general --cb-knownclusters --cb-subclusters, expecting ~12,474 BGCs (~14 per genome) for Burkholderia/Paraburkholderia and ~19,087 (~9 per genome) for Pseudomonas. Cluster all predicted BGCs into Gene Cluster Families with BiG-SCAPE 1.1.5 at the 0.70 BiG-SCAPE distance cutoff, expecting 856 GCFs for Burkholderia/Paraburkholderia and 1279 for Pseudomonas. The rationale is that GCF presence/absence, like gene-cluster distributions, feeds bacLIFE's downstream lifestyle-prediction machine learning. Within the Snakemake-based bacLIFE clustering module, generate the binary GCF matrix as gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs) and tally BGC classes per genome into bgc_class_tally.csv.

## Related tools

- **Prokka 1.14.6** (Gene prediction producing the .gbk files required as antiSMASH input)
- **antiSMASH 5.1.2** (Predicts BGCs per genome with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters)
- **BiG-SCAPE 1.1.5** (Clusters all predicted BGCs into Gene Cluster Families at the 0.70 distance cutoff)
- **bacLIFE** (Snakemake-based module wrapping the annotation, BGC prediction, GCF clustering, and matrix generation with custom scripts) — https://github.com/Carrion-lab/bacLIFE
- **Snakemake** (Workflow engine that processes, annotates and generates the data in the correct format for the clustering module)

## Examples

```
antismash --cb-general --cb-knownclusters --cb-subclusters Burkholderia_genome.gbk
```

## Evaluation signals

- gcf_absence_presence_matrix.tsv exists, is binary, and its rows equal genomes and columns equal individual GCFs.
- Predicted BGC totals match the reported values: ~12,474 BGCs for Burkholderia/Paraburkholderia and ~19,087 for Pseudomonas (~14 and ~9 BGCs per genome, respectively).
- GCF totals at the 0.70 BiG-SCAPE distance cutoff match 856 (Burkholderia/Paraburkholderia) and 1279 (Pseudomonas).
- bgc_class_tally.csv confirms NRPS is the most abundant BGC class in Pseudomonas as reported in Supplementary Fig. 17a.
- Every input genome passed through antiSMASH with the clusterblast flags applied (--cb-general --cb-knownclusters --cb-subclusters).

## Limitations

- The quoted BGC and GCF counts are dataset-specific (16,846-genome Burkholderia/Paraburkholderia and Pseudomonas benchmark); other datasets will yield different totals.
- Results depend on antiSMASH and BiG-SCAPE versions (5.1.2 and 1.1.5) and the fixed 0.70 BiG-SCAPE distance cutoff.
- Only high-quality genomes (>90% completeness, <5% contamination per CheckM) should enter the pipeline; contaminated or fragmented assemblies degrade BGC prediction.
- BGC detection is prediction-based (antiSMASH), so novel atypical clusters may be missed or misclassified.

## Evidence

- [methods] an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [methods] Gene prediction was performed using Prokka 1.14.6: "Gene prediction was performed using Prokka 1.14.6"
- [methods] the first one is a Snakemake-based37 workflow that processes, annotates and generates the data in the correct format: "the first one is a Snakemake-based37 workflow that processes, annotates and generates the data in the correct format"
- [readme] Predicts, clusters and annotates the genes of every input genome: "Predicts, clusters and annotates the genes of every input genome"
