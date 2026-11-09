---
name: protein-sequence-clustering
description: Use when you have Prokka-predicted protein FASTAs from many quality-filtered
  genomes and need function-based gene clusters plus a genome-by-cluster absence/presence
  matrix for comparative genomics or feature-based ML.
when_to_use_negative:
- Genomes have not been quality-filtered or redundancy-reduced — bacLIFE clusters
  only high-quality (>90% completeness, <5% contamination) non-redundant genomes
- You need ortholog-level one-to-one grouping rather than similar-function clusters;
  the authors benchmarked against OrthoFinder defaults and chose MCL inflation 3.0
  for EC-number consistency/granularity instead
- Input is raw reads or unannotated contigs — Prokka gene prediction must be run first
  to obtain protein sequences
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Snakemake-based workflow wrapping the clustering module that executes linclust,
    diamond, and MCL to build the absence/presence matrix
  repo: https://github.com/Carrion-lab/bacLIFE
- name: linclust (MMseqs2)
  role: k-mer- and alignment-based pre-clustering of pooled proteins at >90% identity
    with 80% minimum coverage
- name: diamond
  role: blast all-vs-all alignment of linclust representatives at e-value cutoff 1e-5,
    producing the bitscore-weighted network for MCL
- name: MCL (Markov clustering algorithm)
  role: Graph clustering of the diamond bitscore network at inflation 3.0 to produce
    final gene clusters
- name: Prokka 1.14.6
  role: Upstream gene prediction generating the protein sequences fed into clustering
- name: Snakemake
  role: Workflow engine orchestrating the clustering rules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:9e4c6bc4eb5ed8dd156322aedf9d9a557ff50a9f9799d70621a44f531a345ac4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - MCL (Markov clustering algorithm)
  - Prokka
  - Prokka 1.14.6
  - Snakemake
  - bacLIFE
  - diamond
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# protein-sequence-clustering

## Summary

Two-stage functional clustering of pooled predicted proteins — MMseqs2 linclust pre-clustering followed by MCL graph clustering on a diamond bitscore network — to build gene-family absence/presence matrices for downstream lifestyle prediction in bacLIFE.

## When to use

Use when you have Prokka-predicted protein FASTAs from many quality-filtered genomes and need function-based gene clusters plus a genome-by-cluster absence/presence matrix for comparative genomics or feature-based ML.

## When NOT to use

- Genomes have not been quality-filtered or redundancy-reduced — bacLIFE clusters only high-quality (>90% completeness, <5% contamination) non-redundant genomes
- You need ortholog-level one-to-one grouping rather than similar-function clusters; the authors benchmarked against OrthoFinder defaults and chose MCL inflation 3.0 for EC-number consistency/granularity instead
- Input is raw reads or unannotated contigs — Prokka gene prediction must be run first to obtain protein sequences

## Inputs

- Combined FASTA of Prokka-predicted protein sequences from quality-filtered, non-redundant genomes
- Quality-filtered genome set (CheckM >90% completeness, <5% contamination; 0.99 Mash-ANI redundancy-reduced)

## Outputs

- Gene clusters from MCL (functional gene family database)
- Absence/presence matrix (TSV, one row per non-redundant genome) of gene clusters across input genomes

## How to apply

Pool Prokka-predicted protein sequences from all non-redundant, high-quality genomes (post 0.99 Mash-ANI redundancy reduction and CheckM >90% completeness/<5% contamination filtering) into a single combined FASTA. Pre-cluster with MMseqs2 linclust using --min-seq-id 0.9 --cov-mode 0 -c 0.8 to collapse >90% identical sequences at >=80% coverage, cheaply removing redundancy before graph clustering. Align the linclust representatives all-vs-all with diamond at an e-value cutoff of 1e-5 and format the output as a network file whose first two columns are gene nodes and the third column is the diamond bitscore. Run MCL on this bitscore network with inflation 3.0, the value tuned for EC-number consistency against OrthoFinder defaults, to obtain the final gene clusters. Finally, build the absence/presence matrix with a custom R script writing one row per non-redundant genome (157,696 clusters for Burkholderia/Paraburkholderia; 191,300 for Pseudomonas).

## Related tools

- **bacLIFE** (Snakemake-based workflow wrapping the clustering module that executes linclust, diamond, and MCL to build the absence/presence matrix) — https://github.com/Carrion-lab/bacLIFE
- **linclust (MMseqs2)** (k-mer- and alignment-based pre-clustering of pooled proteins at >90% identity with 80% minimum coverage)
- **diamond** (blast all-vs-all alignment of linclust representatives at e-value cutoff 1e-5, producing the bitscore-weighted network for MCL)
- **MCL (Markov clustering algorithm)** (Graph clustering of the diamond bitscore network at inflation 3.0 to produce final gene clusters)
- **Prokka 1.14.6** (Upstream gene prediction generating the protein sequences fed into clustering)
- **Snakemake** (Workflow engine orchestrating the clustering rules)

## Examples

```
mmseqs linclust combined_proteins.fasta linclust_db tmp --min-seq-id 0.9 --cov-mode 0 -c 0.8 && mcl network.txt --abc -I 3.0 -o gene_clusters.txt
```

## Evaluation signals

- Absence/presence matrix exists in TSV format with one row per non-redundant genome (row count equals 644 + 200 Burkholderia/Paraburkholderia, 2050 Pseudomonas)
- Inferred cluster totals match the reported 157,696 (Burkholderia/Paraburkholderia) and 191,300 (Pseudomonas) gene clusters
- linclust parameters confirm --min-seq-id 0.9 --cov-mode 0 -c 0.8 was applied before graph clustering
- Network file fed to MCL is a 3-column format: two gene-node columns plus a third column of diamond bitscores
- MCL inflation set to 3.0; cluster consistency can be spot-checked against EC numbers of the ENZYME database

## Limitations

- Clustering quality depends on upstream filtering: only genomes passing CheckM (>90% completeness, <5% contamination) and 0.99 Mash-ANI redundancy reduction should enter the pool
- Inflation 3.0 was tuned against EC-number consistency and benchmarked against OrthoFinder defaults; other granularities may be preferable for other use cases
- Scaling: on the study datasets clustering yielded ~158k–191k clusters across thousands of genomes, so the all-vs-all diamond step can be expensive without the linclust pre-clustering

## Evidence

- [methods] linclust clustering parameters: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] diamond alignment cutoff: "aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5"
- [methods] clustering approach: "Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38"
- [readme] bacLIFE modules: "Clustering module
	Predicts, clusters and annotates the genes of every input genome"
- [abstract] workflow context: "a streamlined computational workflow for genome annotation, large-scale comparative genomics, and prediction of lifestyle-associated genes (LAGs)"
