---
name: topic-gene-ranking
description: Use when you have an Ntopics x Ngenes topic association matrix (beta)
  from a fitted CODAL/MIRA topic model and need to interpret an individual topic as
  a gene program, e.
when_to_use_negative:
- The topic model was not fit with CODAL/MIRA and no beta (topic association) matrix
  exists — topic weights from non-interpretable latent embeddings cannot be ranked
  as gene programs.
- No ontology-defined gene membership set exists for the target cell type, since the
  Fisher's exact contingency table requires a defined gene universe and ontology-member
  counts.
- You want batch-effect correction itself rather than interpretation of an already-fitted
  topic model — use CODAL integration as a preprocessing step instead.
tools:
- name: MIRA
  role: Provides the interpretable expression topic model from which the Ntopics x
    Ngenes beta matrix and per-topic gene rankings are extracted
  repo: github.com/cistrome/MIRA
- name: CODAL
  role: Batch-disentangling algorithm used for inference in the MIRA topic model whose
    Topic 14 beta weights are ranked and tested
  repo: https://mira-multiome.readthedocs.io
- name: SciPy
  role: Supplies scipy.stats.fisher_exact (alternative='greater') for the one-sided
    ontology enrichment test
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:aa4d86f7dbeab99407615327499a65a423808b6a8ac4bb0cfceef66ebdf2520f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - MIRA
  - SciPy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# topic-gene-ranking

## Summary

Ranks genes by their weight in a specific topic (e.g., Topic 14) from a fitted CODAL/MIRA expression topic model β matrix, retains the top 200 genes per the paper's convention, and tests whether that gene set is enriched for ontology-defined cell-identity programs (e.g., HuBMAP CD16+ monocyte genes) via a one-sided Fisher's exact test.

## When to use

Use when you have an Ntopics x Ngenes topic association matrix (beta) from a fitted CODAL/MIRA topic model and need to interpret an individual topic as a gene program, e.g., testing whether its top-associated genes capture a specific cell-type ontology membership or correlate with cell-type log-fold changes.

## When NOT to use

- The topic model was not fit with CODAL/MIRA and no beta (topic association) matrix exists — topic weights from non-interpretable latent embeddings cannot be ranked as gene programs.
- No ontology-defined gene membership set exists for the target cell type, since the Fisher's exact contingency table requires a defined gene universe and ontology-member counts.
- You want batch-effect correction itself rather than interpretation of an already-fitted topic model — use CODAL integration as a preprocessing step instead.

## Inputs

- Fitted CODAL/MIRA expression topic model with its Ntopics x Ngenes beta (topic association) matrix
- Ontology gene membership list for the target cell type (e.g., HuBMAP CD16+ monocyte ontology genes)
- Gene log-fold-change values in the target cell type (for the topic-score vs. LFC correlation)

## Outputs

- Top-200 gene ranking for the target topic, with topic association scores
- One-sided Fisher's exact test enrichment p-value for ontology membership among the top-200 genes (expected ~1.5e-19 for Topic 14 / CD16+ monocytes)
- Topic-vs-log-fold-change correlation statistic (Spearman/Pearson)
- topic14_enrichment_results.csv containing the ranking, p-value, and correlation

## How to apply

Load the fitted CODAL/MIRA expression topic model and extract the Ntopics x Ngenes topic association matrix (beta), then rank all genes by the target topic's weight and retain the top 200 genes, mirroring the paper's top-200-genes-per-topic selection convention. Build a 2x2 contingency table crossing the top-200 gene set (vs. all remaining modeled genes) against ontology membership (e.g., HuBMAP CD16+ monocyte ontology genes) and run a one-sided Fisher's exact test with scipy.stats.fisher_exact(alternative='greater') to obtain the enrichment p-value. Optionally, compute a Spearman/Pearson correlation between the topic's gene association scores and the genes' log-fold-change values in the target cell type (Fig. 3f, g comparison). Rationale: topic weights are interpretable gene co-regulation modules, so high-weight genes should be specifically associated with the biological program the topic describes — a significant enrichment (expected ~1.5e-19 for Topic 14 vs. CD16+ monocytes) confirms the topic captures that identity program rather than a batch artifact.

## Related tools

- **MIRA** (Provides the interpretable expression topic model from which the Ntopics x Ngenes beta matrix and per-topic gene rankings are extracted) — github.com/cistrome/MIRA
- **CODAL** (Batch-disentangling algorithm used for inference in the MIRA topic model whose Topic 14 beta weights are ranked and tested) — https://mira-multiome.readthedocs.io
- **SciPy** (Supplies scipy.stats.fisher_exact (alternative='greater') for the one-sided ontology enrichment test)

## Examples

```
odds, p = scipy.stats.fisher_exact([[n_top200_in_ontology, 200 - n_top200_in_ontology], [n_rest_in_ontology, n_rest_not_in_ontology]], alternative='greater')
```

## Evaluation signals

- The one-sided Fisher's exact p-value must reproduce the reported value of 1.5e-19 for Topic 14 vs. HuBMAP CD16+ monocyte ontology membership.
- The computed topic-association-vs-log-fold-change correlation must match the trend displayed in Fig. 3f, g.
- The ranked gene set must be restricted to the top 200 genes by Topic 14 beta weight, matching the paper's top-200-genes-per-topic convention.
- The contingency table must use the full set of modeled genes (top-200 vs. all remaining modeled genes), not a smaller gene universe.
- topic14_enrichment_results.csv must contain the top-200 ranking, the Fisher's exact p-value, and the topic-vs-LFC correlation.

## Limitations

- Only genes in the modeled gene set (e.g., the highly variable genes retained by the pipeline) can appear in the top-200 ranking; ontology genes outside the model are invisible to the test.
- Topic-gene associations describe co-regulation modules, but new or perturbation-induced cell states should be validated with orthogonal biological evidence to rule out under-correction of technical effects.
- The enrichment is one-sided and assumes ontology membership among modeled genes is the relevant background; different gene universes or membership definitions will change the p-value.

## Evidence

- [intro] The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data.: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
- [readme] Multimodal topic modeling: "* Multimodal topic modeling
* Construction a joint representation of cells"
- [other] Run a one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater') to obtain the enrichment p-value: [evidence span withheld — anchor status: not_found]
