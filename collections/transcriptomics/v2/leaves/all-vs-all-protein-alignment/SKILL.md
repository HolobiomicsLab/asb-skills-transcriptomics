---
name: all-vs-all-protein-alignment
description: Use when pooled protein FASTA files from annotated genomes must be converted
  into a bitscore-weighted gene similarity network to drive graph-based gene family
  clustering (e.g., MCL) and downstream absence/presence matrices.
when_to_use_negative:
- Input is already a gene cluster absence/presence matrix or an MCL-ready network
  file — proceed directly to lifestyle prediction or MCL instead of re-aligning.
- Nucleotide-level or genomic alignment is required (e.g., genome-level ANI redundancy
  reduction) — this skill operates on Prokka-predicted protein sequences only.
- Ortholog group calling with OrthoFinder is explicitly preferred — the article benchmarks
  bacLIFE's diamond+MCL clustering against OrthoFinder defaults as an alternative.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: linclust (MMseqs2)
  role: Pre-clusters pooled proteins at >90% identity, ≥80% coverage to select representatives,
    reducing the cost of the all-vs-all alignment
  repo: https://github.com/Carrion-lab/bacLIFE
- name: diamond
  role: Performs the blast all-vs-all alignment of linclust representatives with e-value
    cutoff 1e-5, producing the bitscore network
  repo: https://github.com/Carrion-lab/bacLIFE
- name: MCL (Markov clustering algorithm)
  role: Consumes the diamond bitscore network (inflation 3.0) to produce the final
    functional gene clusters
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Snakemake
  role: Orchestrates the bacLIFE clustering-module workflow steps, including linclust,
    diamond, and MCL
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:51f7e0d95fc558629d4616880f8d37f8f8e2854070a39b9f24834540f3ef958a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - MCL (Markov clustering algorithm)
  - Prokka
  - Snakemake
  - bacLIFE
  - diamond
  - linclust (MMseqs2)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# all-vs-all-protein-alignment

## Summary

Generates a weighted similarity network of protein sequences by aligning cluster representatives all-vs-all with diamond (e-value cutoff 1e-5), where the first two columns are gene nodes and the third column is the diamond bitscore. This network is the input required for Markov clustering (MCL) into functional gene clusters in the bacLIFE clustering module.

## When to use

Use when pooled protein FASTA files from annotated genomes must be converted into a bitscore-weighted gene similarity network to drive graph-based gene family clustering (e.g., MCL) and downstream absence/presence matrices.

## When NOT to use

- Input is already a gene cluster absence/presence matrix or an MCL-ready network file — proceed directly to lifestyle prediction or MCL instead of re-aligning.
- Nucleotide-level or genomic alignment is required (e.g., genome-level ANI redundancy reduction) — this skill operates on Prokka-predicted protein sequences only.
- Ortholog group calling with OrthoFinder is explicitly preferred — the article benchmarks bacLIFE's diamond+MCL clustering against OrthoFinder defaults as an alternative.

## Inputs

- Combined FASTA of Prokka-predicted protein sequences from all quality-filtered, non-redundant genomes (post 0.99 Mash-estimated ANI redundancy reduction)
- linclust cluster representative sequences (representatives from >90% identity / 80% coverage pre-clustering)

## Outputs

- Network-style file with gene-node pairs in columns 1–2 and diamond bitscore in column 3 (MCL input)
- Final gene clusters produced by MCL (inflation 3.0) on the diamond bitscore network
- Gene cluster absence/presence matrix (TSV, one row per non-redundant genome)

## How to apply

First pool the Prokka-predicted protein sequences from all quality-filtered, non-redundant genomes into a single combined FASTA and pre-cluster with MMseqs2 linclust using --min-seq-id 0.9 --cov-mode 0 -c 0.8 (>90% identity at ≥80% coverage) to reduce redundancy before graph clustering. Then align the linclust cluster representatives all-vs-all with diamond (blast all vs all mode) at an e-value cutoff of 1e-5. Format the diamond output as a network-style file whose first two columns are the gene nodes and whose third column is the diamond bitscore, so that it can be fed directly to MCL with inflation 3.0 (tuned against EC-number consistency and benchmarked against OrthoFinder defaults). Rationale: linclust pre-clustering avoids the cost of a full all-vs-all alignment of every protein, while the diamond bitscore network preserves similarity granularity needed for MCL to recover functionally consistent gene clusters.

## Related tools

- **linclust (MMseqs2)** (Pre-clusters pooled proteins at >90% identity, ≥80% coverage to select representatives, reducing the cost of the all-vs-all alignment) — https://github.com/Carrion-lab/bacLIFE
- **diamond** (Performs the blast all-vs-all alignment of linclust representatives with e-value cutoff 1e-5, producing the bitscore network) — https://github.com/Carrion-lab/bacLIFE
- **MCL (Markov clustering algorithm)** (Consumes the diamond bitscore network (inflation 3.0) to produce the final functional gene clusters) — https://github.com/Carrion-lab/bacLIFE
- **Snakemake** (Orchestrates the bacLIFE clustering-module workflow steps, including linclust, diamond, and MCL) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
mmseqs linclust combined_proteins.fasta clusters.tsv --min-seq-id 0.9 --cov-mode 0 -c 0.8
```

## Evaluation signals

- Network file format check: first two columns are gene nodes and the third column contains diamond bitscores (network-style layout accepted by MCL).
- Only linclust representatives were aligned, not all pooled proteins — verify representative count matches linclust cluster count.
- Downstream cluster totals reproduce reported values (157,696 gene clusters for Burkholderia/Paraburkholderia from 644+200 genomes; 191,300 for Pseudomonas from 2050 genomes).
- Absence/presence matrix is TSV with row count equal to non-redundant genome counts (844 Burkholderia/Paraburkholderia; 2050 Pseudomonas).
- Cluster functional consistency validated against EC numbers of the ENZYME database and compared with OrthoFinder defaults for consistency and granularity.

## Limitations

- Inflation value 3.0 and the e-value 1e-5 cutoff were tuned for this workflow; other datasets may require re-tuning against EC-number consistency.
- All-vs-all diamond alignment is expensive, so it is only tractable after linclust redundancy reduction and genome filtering (CheckM >90% completeness, <5% contamination; 0.99 Mash-estimated ANI redundancy reduction).
- Performance was benchmarked against OrthoFinder default settings; behavior relative to other clustering strategies is not established in the article.

## Evidence

- [methods] The representatives of the linclust clusters were aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5: "The representatives of the linclust clusters were aligned to each other with a blast all vs all using diamond89 with an e-value cutoff of 1e−5"
- [methods] Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38: "Gene clustering is performed using a combination of linclust39 and the Markov clustering algorithm38"
- [methods] cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8): "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
- [readme] Predicts, clusters and annotates the genes of every input genome: "Predicts, clusters and annotates the genes of every input genome"
