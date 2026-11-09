---
name: gene-cluster-absence-presence-matrix-generation
description: Use when starting from quality-filtered bacterial genome FASTAs and needing
  a gene-cluster-by-genome binary matrix as the foundation for pan-genome analyses
  or lifestyle-associated gene (LAG/pLAG) prediction.
when_to_use_negative:
- Input is already an absence/presence or feature matrix — proceed directly to lifestyle
  prediction or Fisher's exact test pLAG analysis
- Genomes have not passed quality filtering (CheckM >90% completeness, <5% contamination)
  or redundancy reduction at 0.99 Mash ANI — low-quality or redundant assemblies will
  inflate cluster counts
- Only a handful of genomes are available — the clustering module is designed for
  large-scale comparative genomics (thousands of genomes per genus)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Wrapping Snakemake workflow that orchestrates prediction, clustering, and
    matrix generation
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Prokka 1.14.6
  role: Gene prediction producing the protein sequences pooled for clustering
- name: linclust (MMseqs2)
  role: Pre-clustering sequences >90% identical at ≥80% coverage before MCL
- name: diamond
  role: All-vs-all alignment of linclust representatives with e-value cutoff 1e-5
    to build the bitscore network
- name: MCL (Markov clustering algorithm)
  role: Graph clustering of the diamond network at inflation 3.0 to define final gene
    clusters
- name: CheckM
  role: Quality filter retaining only high-quality genomes before clustering
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:123b52944e4d61120d727623317ec5b61e4d3f4abb166d0d94bf59363a52def2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - CheckM
  - MCL (Markov clustering algorithm)
  - Prokka
  - Prokka 1.14.6
  - Snakemake
  - bacLIFE
  - diamond
  - gtools
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Gene cluster absence/presence matrix generation

## Summary

Reconstructs the bacLIFE clustering module that converts Prokka-predicted proteins from quality-filtered genomes into functional gene clusters (linclust + diamond + MCL) and an absence/presence matrix that feeds lifestyle prediction and pLAG detection.

## When to use

Use when starting from quality-filtered bacterial genome FASTAs and needing a gene-cluster-by-genome binary matrix as the foundation for pan-genome analyses or lifestyle-associated gene (LAG/pLAG) prediction.

## When NOT to use

- Input is already an absence/presence or feature matrix — proceed directly to lifestyle prediction or Fisher's exact test pLAG analysis
- Genomes have not passed quality filtering (CheckM >90% completeness, <5% contamination) or redundancy reduction at 0.99 Mash ANI — low-quality or redundant assemblies will inflate cluster counts
- Only a handful of genomes are available — the clustering module is designed for large-scale comparative genomics (thousands of genomes per genus)

## Inputs

- Quality-filtered, non-redundant genome set (e.g. NCBI RefSeq assemblies; 644+200 Burkholderia/Paraburkholderia and 2050 Pseudomonas genomes)
- Prokka-predicted protein sequences pooled into a combined FASTA file
- Per-genome metadata/lifestyle labels for downstream matrix annotation

## Outputs

- Gene cluster absence/presence matrix (TSV; one row per non-redundant genome, one column per cluster)
- linclust cluster representative protein sequences (for downstream BLASTP annotation)
- Cluster totals (reported: 157,696 clusters for Burkholderia/Paraburkholderia; 191,300 for Pseudomonas)

## How to apply

Extract Prokka 1.14.6-predicted protein sequences from all non-redundant, quality-filtered genomes (post 0.99 Mash-estimated ANI redundancy reduction; CheckM >90% completeness, <5% contamination) and pool them into a single FASTA. Pre-cluster with MMseqs2 linclust using --min-seq-id 0.9 --cov-mode 0 -c 0.8 (>90% identity at ≥80% coverage) to remove redundancy before graph clustering. Align linclust representatives all-vs-all with diamond at an e-value cutoff of 1e-5, formatting output as a network file whose first two columns are gene nodes and the third is the diamond bitscore. Run MCL on this bitscore network with inflation 3.0 (tuned against EC-number consistency from the ENZYME database and benchmarked against OrthoFinder defaults), then build the absence/presence matrix with a custom R script, one row per non-redundant genome.

## Related tools

- **bacLIFE** (Wrapping Snakemake workflow that orchestrates prediction, clustering, and matrix generation) — https://github.com/Carrion-lab/bacLIFE
- **Prokka 1.14.6** (Gene prediction producing the protein sequences pooled for clustering)
- **linclust (MMseqs2)** (Pre-clustering sequences >90% identical at ≥80% coverage before MCL)
- **diamond** (All-vs-all alignment of linclust representatives with e-value cutoff 1e-5 to build the bitscore network)
- **MCL (Markov clustering algorithm)** (Graph clustering of the diamond network at inflation 3.0 to define final gene clusters)
- **CheckM** (Quality filter retaining only high-quality genomes before clustering)

## Examples

```
linclust --min-seq-id 0.9 --cov-mode 0 -c 0.8 -i combined_proteins.fasta -o linclust_clusters.tsv; mcl network.tsv --abc -I 3.0 -o gene_clusters.out
```

## Evaluation signals

- Matrix file exists in TSV format and its row count equals the non-redundant genome count after 0.99 ANI filtering (e.g. 2050 for Pseudomonas)
- Total cluster counts approximate the reported 157,696 (Burkholderia/Paraburkholderia) and 191,300 (Pseudomonas) gene clusters
- linclust parameters (--min-seq-id 0.9 --cov-mode 0 -c 0.8) and MCL inflation 3.0 are recorded in the pipeline logs
- Matrix is binary (presence/absence per genome per cluster) and column IDs correspond to cluster representatives usable as BLASTP subjects
- Downstream reproduction checks pass, e.g. pLAG counts of 382/478/786 (Burkholderia) and 48/74/377 (Pseudomonas) per lifestyle from this matrix

## Limitations

- Cluster granularity depends on the MCL inflation value (3.0 tuned via EC-number consistency); alternative inflation values will change cluster counts and downstream pLAG calls
- Quality filtering thresholds (>90% completeness, <5% contamination) and 0.99 ANI redundancy reduction are prerequisites; skipping them biases cluster counts
- Cluster counts are dataset- and genus-specific — they will not match the reported values on different input genome sets

## Evidence

- [methods] cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8): "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] The representatives of the linclust clusters were aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5: "The representatives of the linclust clusters were aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5"
- [methods] we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination): "we used CheckM87 quality analysis to only retain high-quality genomes (>90% completeness and <5% contamination)"
- [methods] Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38: "Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38"
- [readme] a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics: "bacLIFE is a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles"
