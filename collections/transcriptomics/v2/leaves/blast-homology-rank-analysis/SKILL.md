---
name: blast-homology-rank-analysis
description: 'Use when a cross-species gene-matching benchmark needs a sequence-alignment
  baseline: you have two species'' protein sequences (e.g. human and mouse) and must
  assign per-gene BLAST hits and match ranks to compare against embedding-based matches.'
when_to_use_negative:
- When the goal is cell-type matching itself rather than the gene-matching baseline
  — use the TACTiCS transfer pipeline (TACTiCS class in tactics.py) instead
- When only protein embeddings and no BLAST alignment are needed to build the gene-similarity
  distance matrix — `calc_dist` on `embed_proteins` output is the intended path within
  TACTiCS
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Sequence-alignment engine producing human-vs-mouse protein best-hit assignments
    and per-gene match ranks
- name: TACTiCS
  role: Framework whose gene-matching step is benchmarked against the BLAST baseline;
    consumes the gene-similarity distance matrix
  repo: https://github.com/kbiharie/TACTiCS
- name: transformers
  role: Generates the ProtBERT protein embeddings (via embed_proteins in genes.py)
    that BLAST ranks are compared against
  repo: https://github.com/kbiharie/TACTiCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:561b24c4ecd00d01bced51d7495ff1f1d623edf9ef68703beab2d3c7ef7fa017
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# blast-homology-rank-analysis

## Summary

Run BLAST on cross-species (human vs mouse) protein sequences to obtain per-gene best-hit assignments and full match rankings, providing the sequence-alignment baseline against which ProtBERT protein-embedding distance matches are compared.

## When to use

Use when a cross-species gene-matching benchmark needs a sequence-alignment baseline: you have two species' protein sequences (e.g. human and mouse) and must assign per-gene BLAST best hits and match ranks to compare against embedding-based matches.

## When NOT to use

- When the goal is cell-type matching itself rather than the gene-matching baseline — use the TACTiCS transfer pipeline (TACTiCS class in tactics.py) instead
- When only protein embeddings and no BLAST alignment are needed to build the gene-similarity distance matrix — `calc_dist` on `embed_proteins` output is the intended path within TACTiCS

## Inputs

- Human and mouse protein sequences (FASTA, from Zenodo doi:10.5281/zenodo.7582460)
- Precomputed ProtBERT protein embeddings (optional, for the comparison baseline; from the same Zenodo deposit)
- Raw scRNA-seq count data for expression-correlation adjudication of discordant genes

## Outputs

- Per-gene BLAST best-hit assignments (human-to-mouse)
- Per-gene BLAST match ranks against ProtBERT embedding-distance ranks
- gene_matching_comparison.csv with one row per comparison metric (e.g. 13050/13935 = 94% top-1 identity, >98% of BLAST matches ranked below rank 100 by ProtBERT distance, 568/818 = 69% discordance favoring BLAST)

## How to apply

Download the human and mouse protein sequences from the Zenodo deposit (doi:10.5281/zenodo.7582460) or the full unprocessed dataset (doi:10.5281/zenodo.6786357). Run BLAST on the human vs mouse protein sequences to obtain best-hit assignments and per-gene BLAST match ranks for each of the 13,935 human genes with a mouse ortholog candidate. For each human gene, record the BLAST top-1 match and the rank of that match; in the reference study 13,050 of these genes had a one-to-one BLAST mouse match, and the BLAST top-1 agreed with the ProtBERT top-1 match in 94% of cases. As the decision point for discordant genes, the 818 human genes whose ProtBERT and BLAST matches differ can be adjudicated by computing expression correlation of each candidate match against the query gene from the raw count data — BLAST was favored in 568/818 (69%) of such cases. Assemble all named metrics (best-hit identity fraction, rank-below-100 fraction, discordance counts, per-gene expression correlations such as IL18R1 vs Il1r1 0.945 vs Il18r1 0.103) into a one-row-per-metric CSV (e.g. gene_matching_comparison.csv).

## Related tools

- **BLAST** (Sequence-alignment engine producing human-vs-mouse protein best-hit assignments and per-gene match ranks)
- **TACTiCS** (Framework whose gene-matching step is benchmarked against the BLAST baseline; consumes the gene-similarity distance matrix) — https://github.com/kbiharie/TACTiCS
- **transformers** (Generates the ProtBERT protein embeddings (via embed_proteins in genes.py) that BLAST ranks are compared against) — https://github.com/kbiharie/TACTiCS

## Examples

```
from genes import embed_proteins, calc_dist; embs = embed_proteins(prot_seqs); D = calc_dist(embs)  # compare D-derived top-1 matches to per-gene BLAST best hits
```

## Evaluation signals

- Count of one-to-one human-mouse BLAST matches should reach 13,050 of 13,935 human genes in the reference setup
- BLAST top-1 vs ProtBERT top-1 agreement should be 13,050/13,935 = 94% when embeddings come from the same Zenodo deposit
- More than 98% of BLAST top-1 matches should rank below rank 100 when re-ranked by ProtBERT distance
- For the 818 discordant genes, expression-correlation adjudication should favor BLAST in 568/818 (69%) of cases
- The IL18R1 control case must reproduce: ProtBERT match Il1r1 correlation 0.945 vs BLAST match Il18r1 correlation 0.103

## Limitations

- BLAST best hits and ProtBERT embedding-distance matches are identical for 94% of genes but diverge for 818 human genes, requiring an independent criterion (expression correlation) to adjudicate
- The comparison is specific to the human-vs-mouse protein sequence set and the TACTiCS motor cortex scRNA-seq datasets (human, mouse, marmoset)
- The transformers package is only necessary for embedding protein sequences; BLAST ranking is an independent step and BLAST-based gene matching gave worse cell type matches than ProtBERT within TACTiCS

## Evidence

- [abstract] ProtBERT-based gene matching baseline: "our gene matching method results in better cell type matches than BLAST in our model"
- [methods] BLAST as comparison method: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] Embedding generation route: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] Data availability for protein sequences: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
- [readme] Package requirement scoping: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
