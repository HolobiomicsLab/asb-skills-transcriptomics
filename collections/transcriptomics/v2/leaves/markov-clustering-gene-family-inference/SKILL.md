---
name: markov-clustering-gene-family-inference
description: Use when a large multi-genome protein set (e.g., Prokka-predicted proteins
  from thousands of RefSeq genomes) must be reduced to functionally coherent gene
  clusters and an absence/presence matrix for downstream comparative or machine-learning
  analyses.
when_to_use_negative:
- Input is a small set of genomes where a single-genome orthology tool (e.g., OrthoFinder
  defaults) is sufficient without linclust pre-clustering
- Genomes have not passed quality filtering (>90% completeness, <5% contamination
  by CheckM) and 0.99 ANI redundancy reduction, as redundancy inflates cluster counts
- The goal is nucleotide-level or strain-level typing rather than protein functional
  clustering for absence/presence modeling
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Overall Snakemake-based workflow implementing the clustering module that generates
    the gene cluster absence/presence matrix
  repo: https://github.com/Carrion-lab/bacLIFE
- name: linclust (MMseqs2)
  role: K-mer- and alignment-based pre-clustering of pooled proteins at >90% identity,
    80% coverage to remove redundancy
- name: diamond
  role: All-vs-all blast alignment of linclust cluster representatives with e-value
    cutoff 1e-5 to build the bitscore network
- name: MCL (Markov clustering algorithm)
  role: Graph clustering of the diamond bitscore network at inflation 3.0 to define
    final gene clusters
- name: Prokka 1.14.6
  role: Gene prediction supplying the protein sequences that are pooled and clustered
- name: Snakemake
  role: Workflow engine orchestrating the annotation, clustering, and matrix-generation
    rules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:760f2c30ce7ef499163dada39a77b0f221f66625945544eedd8c8abd8c6f6b59
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
n_claims: 9
---

# markov-clustering-gene-family-inference

## Summary

Infers functional gene families from pooled protein FASTA files by pre-clustering with MMseqs2 linclust (>90% identity, 80% coverage), aligning cluster representatives all-vs-all with diamond (e-value 1e-5), and running Markov clustering (MCL, inflation 3.0) on the resulting bitscore network, as implemented in the bacLIFE clustering module.

## When to use

Use when a large multi-genome protein set (e.g., Prokka-predicted proteins from thousands of RefSeq genomes) must be reduced to functionally coherent gene clusters and an absence/presence matrix for downstream comparative or machine-learning analyses.

## When NOT to use

- Input is a small set of genomes where a single-genome orthology tool (e.g., OrthoFinder defaults) is sufficient without linclust pre-clustering
- Genomes have not passed quality filtering (>90% completeness, <5% contamination by CheckM) and 0.99 ANI redundancy reduction, as redundancy inflates cluster counts
- The goal is nucleotide-level or strain-level typing rather than protein functional clustering for absence/presence modeling

## Inputs

- Combined FASTA file of Prokka-predicted protein sequences from high-quality, non-redundant genomes (e.g., 844 Burkholderia/Paraburkholderia and 2050 Pseudomonas genomes)
- MCL inflation parameter and diamond e-value cutoff configuration (1e-5)

## Outputs

- Gene cluster assignments (functional gene families database)
- Gene cluster absence/presence matrix in TSV format, one row per non-redundant genome
- Intermediate network file of diamond bitscores (gene node pairs + bitscore)

## How to apply

Extract Prokka-predicted protein sequences from all quality-filtered, non-redundant genomes (post 0.99 Mash-estimated ANI redundancy reduction) and pool them into a single combined FASTA. Pre-cluster with MMseqs2 linclust using --min-seq-id 0.9 --cov-mode 0 -c 0.8 to remove redundancy before the expensive graph-clustering step. Align linclust representatives all-vs-all with diamond at an e-value cutoff of 1e-5, formatting the output as a network file whose first two columns are gene nodes and the third column is the diamond bitscore. Run MCL on this bitscore network with inflation 3.0, a value tuned against EC-number consistency from the ENZYME database and benchmarked against OrthoFinder default settings. Finally, build an absence/presence matrix of clusters per non-redundant genome with a custom R script (reported totals: 157,696 clusters for Burkholderia/Paraburkholderia; 191,300 for Pseudomonas).

## Related tools

- **bacLIFE** (Overall Snakemake-based workflow implementing the clustering module that generates the gene cluster absence/presence matrix) — https://github.com/Carrion-lab/bacLIFE
- **linclust (MMseqs2)** (K-mer- and alignment-based pre-clustering of pooled proteins at >90% identity, 80% coverage to remove redundancy)
- **diamond** (All-vs-all blast alignment of linclust cluster representatives with e-value cutoff 1e-5 to build the bitscore network)
- **MCL (Markov clustering algorithm)** (Graph clustering of the diamond bitscore network at inflation 3.0 to define final gene clusters)
- **Prokka 1.14.6** (Gene prediction supplying the protein sequences that are pooled and clustered)
- **Snakemake** (Workflow engine orchestrating the annotation, clustering, and matrix-generation rules)

## Examples

```
mmseqs linclust combined_proteins.fa linclust_db tmp --min-seq-id 0.9 --cov-mode 0 -c 0.8; diamond blastp reps.fa reps.dmnd -e 1e-5 --out network.tsv; mcl network.tsv -I 3.0 -o clusters.txt
```

## Evaluation signals

- Absence/presence matrix file exists and is in TSV format with one row per non-redundant input genome
- Row count matches expected non-redundant genome totals (844 Burkholderia/Paraburkholderia: 644 + 200; 2050 Pseudomonas)
- Total inferred clusters match reported values (157,696 for Burkholderia/Paraburkholderia; 191,300 for Pseudomonas)
- MCL inflation set to 3.0, validated for EC-number consistency against the ENZYME database and compared to OrthoFinder defaults
- Network file format check: first two columns are gene nodes, third column is the diamond bitscore

## Limitations

- Parameter tuning (inflation 3.0) was benchmarked specifically against EC-number consistency and OrthoFinder defaults; other inflation values may be needed for different granularities
- Pre-clustering with linclust at >90% identity / 80% coverage removes near-identical sequences, so clusters reflect functionally similar families rather than exact orthologs
- The absence/presence matrix requires genomes already filtered by CheckM (>90% completeness, <5% contamination) and 0.99 Mash ANI redundancy reduction; unfiltered input inflates counts

## Evidence

- [methods] linclust parameters: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] diamond alignment: "The representatives of the linclust clusters were aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5"
- [methods] MCL clustering: "Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38"
- [methods] OrthoFinder benchmark: "We evaluated the consistency and granularity of our method in comparison with OrthoFinder40 in default settings"
- [readme] README modules: "Clustering module: Predicts, clusters and annotates the genes of every input genome"
