---
name: gene-expression-correlation-evaluation
description: Use when two gene-matching approaches (e.g. protein-embedding distance
  and BLAST hits) disagree for a set of ortholog candidates and raw count scRNA-seq
  data is available to score candidates by expression correlation.
when_to_use_negative:
- ProtBERT and BLAST top-1 matches already agree (13,050/13,935 = 94% of one-to-one
  human-mouse BLAST matches), so correlation arbitration adds nothing
- No raw count scRNA-seq expression data is available for the query and candidate
  genes
- The goal is cell type matching itself rather than evaluating gene match assignments
  — use the TACTiCS model directly instead
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: TACTiCS
  role: Framework whose gene similarity distance matrix (from calc_dist on ProtBERT
    embeddings) produces the ProtBERT gene matches being evaluated; raw count data
    from the TACTiCS datasets supplies the expression values for correlation
  repo: https://github.com/kbiharie/TACTiCS
- name: BLAST
  role: Sequence-alignment baseline producing the best-hit gene assignments against
    which ProtBERT matches are compared via expression correlation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:84326592fbb51835099e21838c266e06e7c265261a20188e04684a52bc8fa27d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gene-expression-correlation-evaluation

## Summary

Resolves disagreements between gene-matching methods (ProtBERT embedding distance vs BLAST alignment) by computing expression correlation of each candidate match against the query gene from scRNA-seq raw count data. In the TACTiCS human–mouse motor cortex analysis, this arbitrates which ortholog assignment better reflects functional gene similarity.

## When to use

Use when two gene-matching approaches (e.g. protein-embedding distance and BLAST best hits) disagree for a set of ortholog candidates and raw count scRNA-seq data is available to score candidates by expression correlation.

## When NOT to use

- ProtBERT and BLAST top-1 matches already agree (13,050/13,935 = 94% of one-to-one human-mouse BLAST matches), so correlation arbitration adds nothing
- No raw count scRNA-seq expression data is available for the query and candidate genes
- The goal is cell type matching itself rather than evaluating gene match assignments — use the TACTiCS model directly instead

## Inputs

- List of query genes where ProtBERT top-1 match and BLAST best-hit match disagree (e.g. 818 human genes)
- ProtBERT top-1 gene match assignments computed from embedding distances via calc_dist
- BLAST best-hit and per-gene BLAST match rank assignments
- Raw count scRNA-seq data (human/mouse motor cortex, available on Zenodo 10.5281/zenodo.7582460)

## Outputs

- Per-gene expression correlations for each candidate match against the query gene
- Agreement counts of BLAST vs ProtBERT candidates favored by correlation (e.g. 568/818 = 69%)
- Named discordance case metrics (e.g. IL18R1: Il1r1 correlation 0.945 vs Il18r1 correlation 0.103)
- gene_matching_comparison.csv with one row per comparison metric

## How to apply

For each of the genes where the ProtBERT top-1 match differs from the BLAST best hit (818 of 13,935 human genes in the reference study), pull raw count data for the query gene and both candidate orthologs. Compute the expression correlation of each candidate match against the query gene across cells. Count how often the BLAST candidate is favored by higher correlation; in the reference run, BLAST was favored in 568/818 cases (69%). Extract a named discordance case for sanity-checking, e.g. for query gene IL18R1 the ProtBERT match Il1r1 had correlation 0.945 versus the BLAST match Il18r1 at 0.103, showing paralog matches can win on expression. Assemble all named metrics into a gene_matching_comparison.csv with one row per comparison metric.

## Related tools

- **TACTiCS** (Framework whose gene similarity distance matrix (from calc_dist on ProtBERT embeddings) produces the ProtBERT gene matches being evaluated; raw count data from the TACTiCS datasets supplies the expression values for correlation) — https://github.com/kbiharie/TACTiCS
- **BLAST** (Sequence-alignment baseline producing the best-hit gene assignments against which ProtBERT matches are compared via expression correlation)

## Examples

```
python -c "from genes import calc_dist, embed_proteins; D = calc_dist(embed_proteins('human_proteins.fasta','mouse_proteins.fasta')); eval_match_correlation(D, 'raw_counts', out='gene_matching_comparison.csv')"
```

## Evaluation signals

- For 13,050 of the 13,935 human genes with one-to-one BLAST mouse matches (94%), the BLAST and ProtBERT top-1 matches are identical
- More than 98% of BLAST matches are ranked below rank 100 by ProtBERT distance
- Of the 818 discordant genes, the BLAST candidate has higher expression correlation in 568 cases (69%)
- The IL18R1 sanity case shows the ProtBERT match Il1r1 at correlation 0.945 versus the BLAST match Il18r1 at 0.103, flagging paralog confusion
- gene_matching_comparison.csv contains one row per named comparison metric

## Limitations

- Expression correlation can favor paralogs over true orthologs: the ProtBERT match Il1r1 correlates 0.945 with IL18R1 while the true BLAST match Il18r1 correlates only 0.103
- Only the 818 discordant human genes (of 13,935 with one-to-one BLAST mouse matches) can be arbitrated this way; agreement cases need no evaluation
- Protein sequences and embeddings must come from the Zenodo deposit 10.5281/zenodo.7582460 or be regenerated with embed_proteins using the transformers package; TACTiCS additionally requires PyTorch

## Evidence

- [results] ProtBERT and BLAST top-1 gene matches are largely identical: for 13,050 of the 13,935 human genes with a one-to-one BLAST mouse match (94%), the BLAST match is identical to the ProtBERT match.: [evidence span withheld — anchor status: not_found]
- [readme] the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on Zenodo: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on Zenodo"
- [methods] calc_dist calculates the latter for embeddings created with embed_proteins in genes.py: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences.: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [abstract] our gene matching method results in better cell type matches than BLAST in our model: "our gene matching method results in better cell type matches than BLAST in our model"
