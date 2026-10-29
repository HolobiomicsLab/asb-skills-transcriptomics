---
name: protein-sequence-embedding
description: Use when aligning genes across species (e.g. human vs mouse scRNA-seq)
  by protein sequence similarity and you need embedding-based gene matches or a gene
  distance matrix for TACTiCS, as an alternative to BLAST -hit matching.
when_to_use_negative:
- Genes are being matched within a single species only — TACTiCS gene matching exists
  to align orthologs across species
- Only gene names/symbols are available without protein sequences — `embed_proteins`
  requires protein sequences as input
- A BLAST best-hit gene match set is already accepted for the analysis — though note
  the paper shows BLAST matches reduce TACTiCS ADS and recall relative to ProtBERT
  matches
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3474
tools:
- name: transformers
  role: Package used to embed protein sequences (ProtBERT) via `embed_proteins` in
    genes.py
  repo: https://github.com/kbiharie/TACTiCS
- name: TACTiCS
  role: Provides `embed_proteins` and `calc_dist` in genes.py to build the gene similarity
    distance matrix consumed by the TACTiCS class
  repo: https://github.com/kbiharie/TACTiCS
- name: BLAST
  role: Sequence-alignment baseline whose best-hit gene matches are compared against
    ProtBERT embedding top-1 matches
- name: PyTorch
  role: Required dependency for training/evaluating TACTiCS with the resulting distance
    matrix
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:91e184444ff111fd3b50368cbe6342237f541416427b67f86aada9b98db4c618
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# protein-sequence-embedding

## Summary

Generate fixed-length protein sequence embeddings (e.g. ProtBERT via the transformers package) with TACTiCS's `embed_proteins` in genes.py, then convert them into a gene similarity distance matrix with `calc_dist` to serve as the gene-matching input for cross-species cell type matching.

## When to use

Use when aligning genes across species (e.g. human vs mouse scRNA-seq) by protein sequence similarity and you need embedding-based gene matches or a gene distance matrix for TACTiCS, as an alternative to BLAST best-hit matching.

## When NOT to use

- Genes are being matched within a single species only — TACTiCS gene matching exists to align orthologs across species
- Only gene names/symbols are available without protein sequences — `embed_proteins` requires protein sequences as input
- A BLAST best-hit gene match set is already accepted for the analysis — though note the paper shows BLAST matches reduce TACTiCS ADS and recall relative to ProtBERT matches

## Inputs

- Protein sequences for the genes of each species (e.g. human and mouse; downloadable from Zenodo 10.5281/zenodo.7582460)
- Raw scRNA-seq count datasets for the two species being compared (e.g. primary motor cortex)
- Precomputed ProtBERT protein embeddings (optional, available on Zenodo as an alternative to re-embedding)

## Outputs

- Per-protein embedding vectors produced by `embed_proteins`
- A gene-by-gene similarity distance matrix computed by `calc_dist`, used as TACTiCS input

## How to apply

Obtain human and mouse protein sequences from Zenodo (10.5281/zenodo.7582460) or regenerate them, then run `embed_proteins` in genes.py (requires the transformers package, ProtBERT) to produce per-protein embeddings. Feed the embeddings to `calc_dist` to compute the gene similarity distance matrix, which together with two scRNA-seq datasets is the required input to the `TACTiCS` class in tactics.py. The tutorial.ipynb notebook walks through the full two-species workflow including embedding creation. Rationale: ProtBERT embedding distances capture functional similarity beyond sequence identity — 13,050 of 13,935 human genes (94%) have identical top-1 ProtBERT and BLAST mouse matches, yet ProtBERT matches yield better TACTiCS cell type matches (higher ADS and recall) than BLAST matches.

## Related tools

- **transformers** (Package used to embed protein sequences (ProtBERT) via `embed_proteins` in genes.py) — https://github.com/kbiharie/TACTiCS
- **TACTiCS** (Provides `embed_proteins` and `calc_dist` in genes.py to build the gene similarity distance matrix consumed by the TACTiCS class) — https://github.com/kbiharie/TACTiCS
- **BLAST** (Sequence-alignment baseline whose best-hit gene matches are compared against ProtBERT embedding top-1 matches)
- **PyTorch** (Required dependency for training/evaluating TACTiCS with the resulting distance matrix)

## Examples

```
from genes import embed_proteins, calc_dist; E = embed_proteins(sequences); D = calc_dist(E)
```

## Evaluation signals

- Top-1 ProtBERT matches should largely agree with BLAST best hits: ~13,050 of 13,935 human genes with a one-to-one BLAST mouse match (94%) identical
- More than 98% of BLAST best-hit matches should rank below rank 100 when genes are ranked by ProtBERT embedding distance
- The `calc_dist` output must be a square gene-similarity distance matrix covering the genes of both species, matching the two scRNA-seq datasets supplied to TACTiCS
- Downstream TACTiCS evaluation should show higher ADS and recall with ProtBERT-derived matches than with BLAST many-to-many or one-to-one matches across the human-mouse cross-species comparisons
- Embeddings and distance matrices can be cross-checked against the precomputed versions deposited on Zenodo (10.5281/zenodo.7582460)

## Limitations

- For 818 human genes, ProtBERT and BLAST top-1 matches differ; in 568/818 (69%) of those cases expression correlation favors the BLAST match, e.g. for IL18R1 the ProtBERT match Il1r1 has correlation 0.945 while the BLAST match Il18r1 has only 0.103
- The transformers package is needed only for this embedding step; the rest of the TACTiCS pipeline does not require it
- Full unprocessed data must come from a separate Zenodo deposit (10.5281/zenodo.6786357); the 7582460 deposit only holds 50k subsets with precomputed embeddings

## Evidence

- [readme] transformers needed only for embedding: "The transformers package is only necessary for embedding protein sequences."
- [readme] calc_dist on embed_proteins output: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [methods] TACTiCS input contract: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [abstract] ProtBERT beats BLAST for cell type matching: "our gene matching method results in better cell type matches than BLAST in our model"
- [readme] tutorial covers embedding step: "The `tutorial.ipynb` notebook explains step-by-step how to use TACTiCS for a comparison across two species, including how to create protein sequence embeddings."
