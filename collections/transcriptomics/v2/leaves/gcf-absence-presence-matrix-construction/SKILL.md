---
name: gcf-absence-presence-matrix-construction
description: Use when you need BGC-level absence/presence features across a genome
  set — e.g. after annotating genomes with Prokka and predicting BGCs with antiSMASH,
  to feed downstream lifestyle or comparative analyses.
when_to_use_negative:
- You need gene-level (not BGC/GCF-level) presence/absence features — the bacLIFE
  gene-clustering module (linclust + MCL) is the appropriate path instead.
- Your genomes have not yet been annotated to .gbk format or screened for quality;
  antiSMASH expects .gbk input and bacLIFE retains only high-quality genomes (>90%
  completeness, <5% contamination).
- You already possess a GCF feature table; regenerating it via antiSMASH/BiG-SCAPE
  would duplicate computation.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0091
tools:
- name: Prokka 1.14.6
  role: Predicts genes and produces the .gbk files that serve as antiSMASH input
- name: antiSMASH 5.1.2
  role: Detects BGCs per annotated genome, run with clusterblast flags --cb-general
    --cb-knownclusters --cb-subclusters
- name: BiG-SCAPE 1.1.5
  role: Clusters predicted BGCs into Gene Cluster Families at a 0.70 distance cutoff
- name: Snakemake
  role: Workflow engine of the bacLIFE clustering module that orchestrates the BGC→GCF→matrix
    steps
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Python
  role: Custom scripts within the Snakemake-based bacLIFE module generate the GCF
    absence/presence matrix and BGC class tally
  repo: https://github.com/Carrion-lab/bacLIFE
- name: bacLIFE
  role: The overall workflow whose clustering module integrates annotation, BGC detection
    and GCF matrix generation
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:94a6b8f6acd22a632bb9c1dfcd1805e9470ce05228a95686a411f8d144cd0d60
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
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gcf-absence-presence-matrix-construction

## Summary

Reconstructs the biosynthetic gene cluster (BGC) analysis step of the bacLIFE clustering module: BGCs predicted per genome with antiSMASH are grouped into Gene Cluster Families (GCFs) with BiG-SCAPE, and a binary absence/presence matrix of GCFs per genome is generated at the 0.70 BiG-SCAPE distance cutoff.

## When to use

Use when you need BGC-level absence/presence features across a genome set — e.g. after annotating genomes with Prokka and predicting BGCs with antiSMASH, to feed downstream lifestyle or comparative analyses.

## When NOT to use

- You need gene-level (not BGC/GCF-level) presence/absence features — the bacLIFE gene-clustering module (linclust + MCL) is the appropriate path instead.
- Your genomes have not yet been annotated to .gbk format or screened for quality; antiSMASH expects .gbk input and bacLIFE retains only high-quality genomes (>90% completeness, <5% contamination).
- You already possess a GCF feature table; regenerating it via antiSMASH/BiG-SCAPE would duplicate computation.

## Inputs

- Non-redundant genome assemblies (NCBI RefSeq, clustered at 0.99 Mash-estimated ANI, CheckM-filtered >90% completeness, <5% contamination)
- Prokka 1.14.6 annotation files (.gbk) per genome
- antiSMASH 5.1.2 BGC predictions per genome (run with --cb-general --cb-knownclusters --cb-subclusters)
- BiG-SCAPE 1.1.5 GCF clustering network at 0.70 distance cutoff

## Outputs

- gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs, binary values at 0.70 BiG-SCAPE distance cutoff)
- bgc_class_tally.csv (per-genome counts of predicted BGC classes)

## How to apply

Annotate the non-redundant genome assemblies (e.g. Burkholderia/Paraburkholderia and Pseudomonas RefSeq sets filtered by CheckM to >90% completeness and <5% contamination) with Prokka 1.14.6 to produce the .gbk files required as antiSMASH input. Run antiSMASH 5.1.2 on every .gbk file with the clusterblast flags --cb-general --cb-knownclusters --cb-subclusters, expecting roughly 12,474 BGCs (~14/genome) for Burkholderia/Paraburkholderia and ~19,087 BGCs (~9/genome) for Pseudomonas. Cluster all predicted BGCs into GCFs with BiG-SCAPE 1.1.5 (~856 GCFs for Burkholderia/Paraburkholderia, ~1279 for Pseudomonas), then use custom Python scripts within the Snakemake-based bacLIFE module to write the GCF absence/presence matrix at the 0.70 BiG-SCAPE distance cutoff. The rationale is that GCF presence/absence encodes biosynthetic capacity differences that bacLIFE uses alongside gene-cluster distributions to associate lifestyles with candidate BGCs.

## Related tools

- **Prokka 1.14.6** (Predicts genes and produces the .gbk files that serve as antiSMASH input)
- **antiSMASH 5.1.2** (Detects BGCs per annotated genome, run with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters)
- **BiG-SCAPE 1.1.5** (Clusters predicted BGCs into Gene Cluster Families at a 0.70 distance cutoff)
- **Snakemake** (Workflow engine of the bacLIFE clustering module that orchestrates the BGC→GCF→matrix steps) — https://github.com/Carrion-lab/bacLIFE
- **Python** (Custom scripts within the Snakemake-based bacLIFE module generate the GCF absence/presence matrix and BGC class tally) — https://github.com/Carrion-lab/bacLIFE
- **bacLIFE** (The overall workflow whose clustering module integrates annotation, BGC detection and GCF matrix generation) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
antismash --cb-general --cb-knownclusters --cb-subclusters annotated_genome.gbk
```

## Evaluation signals

- gcf_absence_presence_matrix.tsv exists and is strictly binary with rows = genomes and columns = GCFs.
- Predicted BGC totals match reported values: ~12,474 BGCs (~14/genome) for Burkholderia/Paraburkholderia and ~19,087 BGCs (~9/genome) for Pseudomonas.
- GCF totals match reported values: ~856 GCFs for Burkholderia/Paraburkholderia and ~1279 GCFs for Pseudomonas.
- bgc_class_tally.csv confirms NRPS is the most abundant BGC class in Pseudomonas, as reported in Supplementary Fig. 17a.
- The matrix is reproducible at the 0.70 BiG-SCAPE distance cutoff specified in the methods.

## Limitations

- Expected BGC and GCF counts are dataset-specific to the Burkholderia/Paraburkholderia and Pseudomonas collections; other genera will differ in scale.
- The 0.70 BiG-SCAPE distance cutoff is a fixed parameter; changing it alters GCF granularity and the resulting matrix.
- Genomes failing the CheckM quality filter (>90% completeness, <5% contamination) or the 0.99 ANI redundancy reduction must be excluded before BGC analysis.

## Evidence

- [methods] an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [methods] Gene prediction was performed using Prokka 1.14.6: "Gene prediction was performed using Prokka 1.14.6"
- [methods] only retain high-quality genomes (>90% completeness and <5% contamination): "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [readme] Clustering module predicts, clusters and annotates the genes of every input genome: "Predicts, clusters and annotates the genes of every input genome"
