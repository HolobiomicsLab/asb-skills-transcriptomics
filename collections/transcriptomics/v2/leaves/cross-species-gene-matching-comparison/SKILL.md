---
name: cross-species-gene-matching-comparison
description: Use when comparing how ortholog/gene matches from protein-embedding distances
  (ProtBERT) differ from BLAST sequence-alignment hits across two species, e.g. human
  vs mouse protein FASTA plus embeddings.
when_to_use_negative:
- When you only need cell-type transfer rather than a gene-matching benchmark — use
  TACTiCS end-to-end (TACTiCS class in tactics.py) instead of the gene-matching comparison
  alone.
- When protein sequences or embeddings for the species pair are unavailable and only
  nucleotide-level data exist; this comparison is defined on protein sequences/ProtBERT
  embeddings.
- When you need the full unprocessed scRNA-seq datasets rather than the 50k subsets
  — fetch Zenodo 10.5281/zenodo.6786357 instead of the precomputed comparison inputs.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: TACTiCS
  role: Downstream consumer of the gene similarity distance matrix; provides calc_dist
    and genes.py for computing ProtBERT embedding distances
  repo: https://github.com/kbiharie/TACTiCS
- name: BLAST
  role: Sequence-alignment baseline producing best-hit assignments and per-gene match
    ranks for human vs mouse proteins
- name: transformers
  role: Used to embed protein sequences (embed_proteins in genes.py) when precomputed
    ProtBERT embeddings are not downloaded
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:04b371d08be211fcad1bbda86947e0666b777c95fd601f7b8e3a2bb3ccaef5e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-species-gene-matching-comparison

## Summary

Benchmark protein-embedding-based gene matching (ProtBERT distances) against BLAST sequence-alignment best hits for cross-species (human vs mouse) ortholog assignment, using expression correlation to adjudicate discordant pairs.

## When to use

Use when comparing how ortholog/gene matches from protein-embedding distances (ProtBERT) differ from BLAST sequence-alignment best hits across two species, e.g. human vs mouse protein FASTA plus embeddings.

## When NOT to use

- When you only need cell-type transfer rather than a gene-matching benchmark — use TACTiCS end-to-end (TACTiCS class in tactics.py) instead of the gene-matching comparison alone.
- When protein sequences or embeddings for the species pair are unavailable and only nucleotide-level data exist; this comparison is defined on protein sequences/ProtBERT embeddings.
- When you need the full unprocessed scRNA-seq datasets rather than the 50k subsets — fetch Zenodo 10.5281/zenodo.6786357 instead of the precomputed comparison inputs.

## Inputs

- Human and mouse protein sequences (FASTA)
- ProtBERT protein embeddings (from Zenodo 10.5281/zenodo.7582460 or generated via embed_proteins with the transformers package)
- Raw scRNA-seq count data for the queried species (used to compute expression correlations for discordant genes)
- BLAST best-hit assignments and per-gene BLAST match ranks from human vs mouse protein BLAST

## Outputs

- Top-1 ProtBERT gene match per human gene (13,935 genes) from the calc_dist distance matrix
- gene_matching_comparison.csv with one row per comparison metric (top-1 agreement, rank coverage, discordance adjudication, case studies)
- Set of discordant genes (818) with expression-correlation verdicts favoring BLAST (568/818) or ProtBERT

## How to apply

Download human and mouse protein sequences and precomputed ProtBERT embeddings from Zenodo deposit 10.5281/zenodo.7582460, or regenerate embeddings with `embed_proteins` in `genes.py` (transformers package). Run BLAST on the human vs mouse protein sequences to obtain best-hit assignments and per-gene BLAST match ranks, and compute ProtBERT embedding-based gene similarities with `calc_dist`; for each of the 13,935 human genes take the top-1 ProtBERT match and count agreement with the BLAST best hit (13,050/13,935 = 94% identical). For the 818 human genes where ProtBERT and BLAST disagree, compute expression correlation of each candidate against the query gene from raw count data — BLAST is favored in 568/818 (69%) of cases (e.g. IL18R1: ProtBERT match Il1r1 corr 0.945 vs BLAST match Il18r1 corr 0.103). Assemble all named metrics (agreement rate, rank-100 coverage, discordance adjudication) into gene_matching_comparison.csv with one row per metric.

## Related tools

- **TACTiCS** (Downstream consumer of the gene similarity distance matrix; provides calc_dist and genes.py for computing ProtBERT embedding distances) — https://github.com/kbiharie/TACTiCS
- **BLAST** (Sequence-alignment baseline producing best-hit assignments and per-gene match ranks for human vs mouse proteins)
- **transformers** (Used to embed protein sequences (embed_proteins in genes.py) when precomputed ProtBERT embeddings are not downloaded)

## Examples

```
from genes import embed_proteins, calc_dist; emb = embed_proteins(human_faa, mouse_faa); D = calc_dist(emb)  # then rank top-1 matches vs BLAST best hits
```

## Evaluation signals

- Top-1 agreement between BLAST best hit and ProtBERT match should approach the reported 13,050/13,935 (94%) for human vs mouse one-to-one BLAST matches.
- Over 98% of BLAST best matches should be ranked below rank 100 by ProtBERT distance.
- Among discordant genes, expression-correlation adjudication should favor BLAST in roughly 568/818 (69%) of cases.
- Spot-check the IL18R1 case: ProtBERT match Il1r1 should show expression correlation ~0.945 while BLAST match Il18r1 shows ~0.103.
- gene_matching_comparison.csv must contain one row per named comparison metric with all counts (13935, 13050, 818, 568) traceable to the run.

## Limitations

- The reported agreement (94% top-1 identity) is specific to the 13,935 human genes with a one-to-one BLAST mouse match in the motor cortex datasets; other species pairs or many-to-many orthologies may differ.
- ProtBERT and BLAST disagree for 818 human genes, and in the majority of those (568/818 = 69%) expression correlation favors the BLAST match — so ProtBERT distance alone is not always the better ortholog signal.
- Protein sequences, embeddings, raw counts and trained models on Zenodo cover 50k cell subsets; the full unprocessed dataset requires a separate deposit (10.5281/zenodo.6786357).

## Evidence

- [results] ProtBERT and BLAST top-1 gene matches are largely identical: for 13,050 of the 13,935 human genes with a one-to-one BLAST mouse match (94%), the BLAST match is identical to the ProtBERT match.: [evidence span withheld — anchor status: not_found]
- [methods] calc_dist computes the gene similarity distance matrix from ProtBERT embeddings.: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] README confirms calc_dist/embed_proteins workflow and Zenodo availability of protein sequences and embeddings.: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
- [abstract] Article finding that ProtBERT-based gene matching outperforms BLAST within TACTiCS.: "our gene matching method results in better cell type matches than BLAST in our model"
