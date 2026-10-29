---
name: mcl-inflation-parameter-tuning
description: Use when a comparative-genomics pipeline needs proteins grouped into
  similar-function gene clusters to build an absence/presence matrix, and the Markov
  clustering inflation value must be selected and justified.
when_to_use_negative:
- When orthologous (one-to-one) gene families rather than similar-function clusters
  are required — OrthoFinder was used as the comparison baseline, not the production
  method
- When input proteins have not been redundancy-reduced or quality-filtered — the linclust
  pre-clustering at >90% identity/80% coverage and CheckM filtering presuppose those
  upstream steps
- When the goal is nucleotide-level genome comparison rather than protein-function
  clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: linclust (MMseqs2)
  role: K-mer/alignment-based pre-clustering of pooled proteins at >90% identity,
    ≥80% coverage before MCL graph clustering
  repo: https://github.com/Carrion-lab/bacLIFE
- name: diamond
  role: All-vs-all BLAST-like alignment of linclust representatives (e-value cutoff
    1e−5); bitscores become MCL network edge weights
- name: MCL (Markov clustering algorithm)
  role: Graph clustering of the diamond bitscore network at inflation 3.0 to produce
    the final gene clusters
  repo: https://github.com/Carrion-lab/bacLIFE
- name: OrthoFinder
  role: Reference method (default settings) used to benchmark consistency and granularity
    of the MCL-based clustering
- name: bacLIFE
  role: Snakemake-based workflow wrapping the clustering module end-to-end
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:e265bd441015bd98005473710b168fbc5831fcce8424a3d57ee8456efbfc681f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - MCL (Markov clustering algorithm)
  - OrthoFinder
  - Prokka
  - Snakemake
  - bacLIFE
  - diamond
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# MCL inflation parameter tuning

## Summary

Skill for building functional gene clusters from pooled protein FASTA via MMseqs2 linclust pre-clustering, diamond all-vs-all bitscore networks, and Markov clustering with an inflation parameter tuned against EC-number consistency and benchmarked against OrthoFinder defaults.

## When to use

Use when a comparative-genomics pipeline needs proteins grouped into similar-function gene clusters to build an absence/presence matrix, and the Markov clustering inflation value must be selected and justified.

## When NOT to use

- When orthologous (one-to-one) gene families rather than similar-function clusters are required — OrthoFinder was used as the comparison baseline, not the production method
- When input proteins have not been redundancy-reduced or quality-filtered — the linclust pre-clustering at >90% identity/80% coverage and CheckM filtering presuppose those upstream steps
- When the goal is nucleotide-level genome comparison rather than protein-function clustering

## Inputs

- Combined FASTA of Prokka-predicted protein sequences from all high-quality, non-redundant genomes (CheckM >90% completeness, <5% contamination)
- Genome set reduced at 0.99 Mash-estimated ANI (e.g. 644 + 200 Burkholderia/Paraburkholderia; 2050 Pseudomonas)

## Outputs

- Gene-cluster assignments (MCL clusters over the diamond bitscore network)
- TSV absence/presence matrix of gene clusters per non-redundant genome (cluster totals e.g. 157,696 for Burkholderia/Paraburkholderia and 191,300 for Pseudomonas)

## How to apply

Pool Prokka-predicted proteins from quality-filtered, non-redundant genomes (post 0.99 Mash-estimated ANI redundancy reduction) into a single combined FASTA. Pre-cluster with MMseqs2 linclust using --min-seq-id 0.9 --cov-mode 0 -c 0.8 (>90% identity at ≥80% coverage) to cut redundancy before the costly graph-clustering step. Align linclust representatives all-vs-all with diamond at an e-value cutoff of 1e−5 and format as a network file whose first two columns are gene nodes and the third column is the diamond bitscore. Run MCL on this bitscore network with inflation set to 3.0 — the value the authors selected by tuning against consistency with ENZYME-database EC numbers and granularity, benchmarked against OrthoFinder defaults. Decide to accept inflation=3.0 only if cluster consistency/granularity matches or exceeds the OrthoFinder baseline; otherwise re-tune. Finally, build a TSV absence/presence matrix (one row per non-redundant genome) with a custom R script to feed downstream lifestyle prediction.

## Related tools

- **linclust (MMseqs2)** (K-mer/alignment-based pre-clustering of pooled proteins at >90% identity, ≥80% coverage before MCL graph clustering) — https://github.com/Carrion-lab/bacLIFE
- **diamond** (All-vs-all BLAST-like alignment of linclust representatives (e-value cutoff 1e−5); bitscores become MCL network edge weights)
- **MCL (Markov clustering algorithm)** (Graph clustering of the diamond bitscore network at inflation 3.0 to produce the final gene clusters) — https://github.com/Carrion-lab/bacLIFE
- **OrthoFinder** (Reference method (default settings) used to benchmark consistency and granularity of the MCL-based clustering)
- **bacLIFE** (Snakemake-based workflow wrapping the clustering module end-to-end) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
mmseqs easy-linclust proteins.fasta clu tmp --min-seq-id 0.9 --cov-mode 0 -c 0.8 && diamond makedb --in reps.fa -d reps && diamond blastp -d reps -q reps -e 1e-5 -o network.txt && mcl network.txt -I 3.0 -o clusters.txt
```

## Evaluation signals

- MCL clusters show consistency with enzyme commission (EC) numbers from the ENZYME database at inflation 3.0
- Consistency and granularity of clusters match or exceed OrthoFinder run in default settings
- Absence/presence matrix row count equals the reported non-redundant genome counts (644 + 200 Burkholderia/Paraburkholderia; 2050 Pseudomonas)
- Inferred cluster totals match the reported 157,696 (Burkholderia/Paraburkholderia) and 191,300 (Pseudomonas) gene clusters
- linclust log/output confirms parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8 were applied

## Limitations

- Inflation 3.0 was tuned for EC-number consistency/granularity on the authors' protein sets; other datasets may require re-tuning
- Clustering operates on similar-function gene clusters, not strict orthology, so downstream lifestyle predictions depend on this granularity choice
- MCL scales poorly without the linclust pre-clustering step; skipping it is expensive at 16,846-genome scale

## Evidence

- [methods] linclust clustering parameters: "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [methods] MCL combination: "Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38"
- [methods] OrthoFinder benchmark: "We evaluated the consistency and granularity of our method in comparison with OrthoFinder40 in default settings"
- [methods] EC-number consistency: "the consistency of the MCL clusters with respect to enzyme commission (EC) numbers of the ENZYME database91"
- [readme] clustering module role: "Predicts, clusters and annotates the genes of every input genome"
