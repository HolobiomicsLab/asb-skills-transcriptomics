---
name: comparative-genomics-pipeline-execution
description: Use when starting from non-redundant bacterial genome assemblies and
  needing BGC/GCF-level absence/presence features (e.g., for lifestyle-association
  or comparative-genomics analyses) rather than gene-level clustering.
when_to_use_negative:
- The analysis goal is gene-level ortholog clustering (linclust + MCL / OrthoFinder
  comparison), not BGC/GCF-level secondary-metabolite features.
- Genomes are not yet quality-filtered or de-redundanted — apply the CheckM (>90%
  completeness, <5% contamination) and 0.99 ANI filtering steps first.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Prokka
  role: Annotates the non-redundant genome assemblies to produce the .gbk files required
    as antiSMASH input
  repo: https://github.com/Carrion-lab/bacLIFE
- name: antiSMASH
  role: Predicts BGCs on every .gbk file with clusterblast flags --cb-general --cb-knownclusters
    --cb-subclusters
- name: BiG-SCAPE
  role: Clusters all predicted BGCs into Gene Cluster Families (GCFs) used to build
    the absence/presence matrix
- name: Snakemake
  role: Workflow engine underlying the bacLIFE clustering module that processes, annotates,
    and generates data in the correct format
- name: bacLIFE
  role: Streamlined computational workflow containing the clustering module whose
    custom scripts produce the GCF absence/presence matrix
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:9fe986cfc9a4d2f54045e9131e4aac1e5baebd4a25aa68b6a33f4d119012bb85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - BiG-SCAPE
  - Prokka
  - Python
  - Snakemake
  - antiSMASH
  - bacLIFE
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# comparative-genomics-pipeline-execution

## Summary

Reconstructs the bacLIFE secondary-metabolite analysis step, in which Prokka-annotated .gbk genomes are mined with antiSMASH 5.1.2 for biosynthetic gene clusters (BGCs), clustered into Gene Cluster Families with BiG-SCAPE 1.1.5, and converted into a GCF absence/presence matrix via custom Snakemake/Python scripts.

## When to use

Use when starting from non-redundant bacterial genome assemblies and needing BGC/GCF-level absence/presence features (e.g., for lifestyle-association or comparative-genomics analyses) rather than gene-level clustering.

## When NOT to use

- The analysis goal is gene-level ortholog clustering (linclust + MCL / OrthoFinder comparison), not BGC/GCF-level secondary-metabolite features.
- Genomes are not yet quality-filtered or de-redundanted — apply the CheckM (>90% completeness, <5% contamination) and 0.99 ANI filtering steps first.

## Inputs

- Non-redundant genome assemblies ( genomes clustered at 0.99 Mash-estimated ANI, CheckM-filtered >90% completeness / <5% contamination )
- Prokka 1.14.6 annotation .gbk files (antiSMASH input)

## Outputs

- gcf_absence_presence_matrix.tsv — binary matrix, rows = genomes, columns = GCFs, generated at the 0.70 BiG-SCAPE distance cutoff
- bgc_class_tally.csv — per-genome counts of predicted BGC classes
- Predicted BGC sets per genome (~12,474 BGCs Burkholderia/Paraburkholderia; ~19,087 BGCs Pseudomonas) and GCF clusters (856 and 1279 GCFs respectively)

## How to apply

First annotate the non-redundant genome assemblies (Burkholderia/Paraburkholderia and Pseudomonas) with Prokka 1.14.6 to produce the .gbk files required as antiSMASH input. Run antiSMASH 5.1.2 on every .gbk file with the clusterblast flags --cb-general --cb-knownclusters --cb-subclusters, expecting ~12,474 BGCs (~14 per genome) for Burkholderia/Paraburkholderia and ~19,087 BGCs (~9 per genome) for Pseudomonas. Cluster all predicted BGCs into Gene Cluster Families with BiG-SCAPE 1.1.5 (856 GCFs for Burkholderia/Paraburkholderia; 1279 GCFs for Pseudomonas). Within the Snakemake-based bacLIFE clustering module, generate the absence/presence matrix of individual GCFs at the 0.70 BiG-SCAPE distance cutoff and save it as gcf_absence_presence_matrix.tsv (rows = genomes, columns = GCFs, binary values), and tally per-genome BGC-class counts into bgc_class_tally.csv.

## Related tools

- **Prokka** (Annotates the non-redundant genome assemblies to produce the .gbk files required as antiSMASH input) — https://github.com/Carrion-lab/bacLIFE
- **antiSMASH** (Predicts BGCs on every .gbk file with clusterblast flags --cb-general --cb-knownclusters --cb-subclusters)
- **BiG-SCAPE** (Clusters all predicted BGCs into Gene Cluster Families (GCFs) used to build the absence/presence matrix)
- **Snakemake** (Workflow engine underlying the bacLIFE clustering module that processes, annotates, and generates data in the correct format)
- **bacLIFE** (Streamlined computational workflow containing the clustering module whose custom scripts produce the GCF absence/presence matrix) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
antismash --cb-general --cb-knownclusters --cb-subclusters genome.gbk
```

## Evaluation signals

- gcf_absence_presence_matrix.tsv exists and is a binary matrix with rows = genomes and columns = GCFs.
- Predicted BGC totals match the reported values: 12,474 for Burkholderia/Paraburkholderia and 19,087 for Pseudomonas.
- GCF totals match the reported values: 856 (Burkholderia/Paraburkholderia) and 1279 (Pseudomonas) at the 0.70 BiG-SCAPE distance cutoff.
- bgc_class_tally.csv shows NRPS as the most abundant BGC class in Pseudomonas, as reported in Supplementary Fig. 17a.
- Number of BGCs per genome falls around ~14 (Burkholderia/Paraburkholderia) and ~9 (Pseudomonas).

## Limitations

- BGC counts and GCF family numbers are tied to specific tool versions (Prokka 1.14.6, antiSMASH 5.1.2, BiG-SCAPE 1.1.5); other versions may shift these values.
- The absence/presence matrix is defined at the specific 0.70 BiG-SCAPE distance cutoff; other cutoffs produce different GCF partitions.
- The bacLIFE clustering module is Snakemake/Python-based and expects the annotated .gbk inputs in the format produced by the pipeline.

## Evidence

- [methods] an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [methods] Gene prediction was performed using Prokka 1.14.6: "Gene prediction was performed using Prokka 1.14.688"
- [readme] bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes"
- [abstract] a streamlined computational workflow for genome annotation, large-scale comparative genomics, and prediction of lifestyle-associated genes (LAGs): "a streamlined computational workflow for genome annotation, large-scale comparative genomics, and prediction of lifestyle-associated genes (LAGs)"
