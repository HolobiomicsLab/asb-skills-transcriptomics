---
name: protein-embedding-distance-rank-statistics
description: Use when two species' protein sequences and ProtBERT embeddings are available
  and you need to quantify how embedding-based gene matches agree with BLAST -hit
  assignments and ranks (e.g., for TACTiCS gene-similarity input validation).
when_to_use_negative:
- You already have a gene similarity distance matrix and only need to run/evaluate
  TACTiCS cell-type matching — supply the matrix directly to the TACTiCS class instead
- Your goal is TACTiCS cell-type classification or transfer-learning label propagation
  across species — this skill covers only the gene-matching benchmark, not the downstream
  neural network or transfer steps
- Genes lack a one-to-one BLAST ortholog match between the species — the 94%/13,935-gene
  statistics apply only to one-to-one human–mouse matches
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_3307
tools:
- name: TACTiCS
  role: Framework whose genes.py provides embed_proteins and calc_dist for computing
    ProtBERT embedding-based gene similarity distances
  repo: https://github.com/kbiharie/TACTiCS
- name: transformers
  role: Hugging Face package used to embed protein sequences with ProtBERT when precomputed
    embeddings are not downloaded from Zenodo
- name: BLAST
  role: Sequence-alignment baseline producing best-hit assignments and per-gene match
    ranks to compare against ProtBERT distance rankings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:608cfd2b51d61aa66fdcbafcffafb931bff9dc65f6f88d5f4b8f5cd7dbe22fcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - TACTiCS
  - transformers
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Protein-embedding-distance-rank-statistics

## Summary

Benchmark gene-matching by ProtBERT protein-embedding distances against BLAST sequence-alignment best hits across species (human vs mouse motor cortex datasets), quantifying top-1 agreement, rank agreement, and expression-correlation arbitration of discordant genes.

## When to use

Use when two species' protein sequences and ProtBERT embeddings are available and you need to quantify how embedding-based gene matches agree with BLAST best-hit assignments and ranks (e.g., for TACTiCS gene-similarity input validation).

## When NOT to use

- You already have a gene similarity distance matrix and only need to run/evaluate TACTiCS cell-type matching — supply the matrix directly to the TACTiCS class instead
- Your goal is TACTiCS cell-type classification or transfer-learning label propagation across species — this skill covers only the gene-matching benchmark, not the downstream neural network or transfer steps
- Genes lack a one-to-one BLAST ortholog match between the species — the 94%/13,935-gene statistics apply only to one-to-one human–mouse matches

## Inputs

- Human and mouse protein sequence FASTA files (or Zenodo precomputed ProtBERT protein embeddings, 10.5281/zenodo.7582460)
- BLAST best-hit assignments and per-gene BLAST match ranks for human vs mouse proteins
- Raw scRNA-seq count data (primary motor cortex, human and mouse) for expression-correlation arbitration

## Outputs

- gene_matching_comparison.csv with one row per comparison metric (top-1 agreement count/fraction, rank-below-100 fraction, discordant-gene correlation tallies)
- Per-gene top-1 ProtBERT matches and their BLAST ranks
- Discordant-gene subset with expression correlations for ProtBERT and BLAST candidate matches (e.g., IL18R1: Il1r1 0.945 vs Il18r1 0.103)

## How to apply

Download human and mouse protein sequences plus precomputed ProtBERT protein embeddings from Zenodo deposit 10.5281/zenodo.7582460, or regenerate embeddings with `embed_proteins` in genes.py using the transformers package. Run BLAST on the human vs mouse protein FASTA files to obtain best-hit assignments and per-gene BLAST match ranks. Compute ProtBERT embedding similarities with `calc_dist` (built on embeddings from `embed_proteins`), then for each of the 13,935 human genes with a one-to-one BLAST mouse match identify the top-1 ProtBERT match: 13,050 (94%) agree with the BLAST best hit, and >98% of BLAST matches rank below ProtBERT rank 100. For the 818 discordant genes, arbitrate by computing expression correlation of each candidate match against the query gene from raw count data — BLAST is favored 568/818 (69%). Assemble all named metrics into gene_matching_comparison.csv with one row per comparison metric.

## Related tools

- **TACTiCS** (Framework whose genes.py provides embed_proteins and calc_dist for computing ProtBERT embedding-based gene similarity distances) — https://github.com/kbiharie/TACTiCS
- **transformers** (Hugging Face package used to embed protein sequences with ProtBERT when precomputed embeddings are not downloaded from Zenodo)
- **BLAST** (Sequence-alignment baseline producing best-hit assignments and per-gene match ranks to compare against ProtBERT distance rankings)

## Examples

```
from genes import embed_proteins, calc_dist; D = calc_dist(embed_proteins(human_mouse_protein_sequences))
```

## Evaluation signals

- Top-1 agreement invariant: 13,050 of 13,935 human genes with a one-to-one BLAST mouse match (94%) have identical BLAST and ProtBERT top-1 matches
- Rank invariant: >98% of BLAST top matches are ranked below rank 100 by ProtBERT embedding distance
- Discordance arbitration: for the 818 genes where ProtBERT and BLAST differ, expression correlation favors BLAST in 568/818 (69%) of cases
- Named case check: IL18R1's ProtBERT match Il1r1 has expression correlation 0.945 while the BLAST match Il18r1 has 0.103
- Output schema check: gene_matching_comparison.csv exists with exactly one row per named comparison metric

## Limitations

- Agreement statistics are conditioned on one-to-one BLAST orthologs (13,935 human genes); genes without one-to-one mouse matches are excluded
- ProtBERT embedding distances can disagree with BLAST for paralogous families (818 discordant genes; IL18R1 matched Il1r1 by embedding but Il18r1 by sequence), and expression correlation favors BLAST in most such cases
- Zenodo deposit 10.5281/zenodo.7582460 provides 50k-subset preprocessed data and trained models; the full unprocessed dataset requires a separate download (10.5281/zenodo.6786357)

## Evidence

- [abstract] our gene matching method results in better cell type matches than BLAST in our model: "our gene matching method results in better cell type matches than BLAST in our model"
- [methods] `calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`.: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [methods] As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities.: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [readme] TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences.: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
