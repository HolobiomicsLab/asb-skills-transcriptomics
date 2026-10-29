---
name: ontology-membership-contingency-analysis
description: Use when you have a fitted topic model's gene association vector (Ntopics
  x Ngenes beta matrix) and an ontology/cell-type gene membership list, and need to
  test if a topic's top genes capture a known cell identity program.
when_to_use_negative:
- When the gene set of interest is not ranked by model association weights (e.g.,
  it is an arbitrary feature table or a differential-expression list with no topic
  beta scores) — a simple ORA is sufficient and the topic-LFC correlation is not defined.
- When testing enrichment for pathways rather than a cell-type identity program validated
  by LFC correlation — the correlation-to-LFC check (Fig. 3f, g trend) is the paper's
  identity confirmation and would not apply.
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MIRA
  role: Provides the topic model whose Ntopics x Ngenes beta association matrix supplies
    the gene ranking to test; CODAL inference is built into MIRA topic models
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Variational topic-modeling extension that yields interpretable biological
    modules (topics) whose gene associations are tested for ontology enrichment
  repo: https://mira-multiome.readthedocs.io
- name: SciPy
  role: Supplies scipy.stats.fisher_exact with alternative='greater' for the one-sided
    enrichment test on the 2x2 contingency table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:5b11a9790d32fe062a0c0e4620fc5a2f36b65af2f6847fac92207dd2e4ba8667
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - SciPy
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ontology-membership-contingency-analysis

## Summary

Tests whether a gene set ranked by a topic model's association weights (e.g., CODAL/MIRA Topic 14 beta weights) is enriched for members of a cell-type ontology (e.g., HuBMAP CD16+ monocyte genes) via a one-sided Fisher's exact test on a 2x2 contingency table, and correlates topic scores with cell-type log-fold changes to confirm the identity program.

## When to use

Use when you have a fitted topic model's gene association vector (Ntopics x Ngenes beta matrix) and an ontology/cell-type gene membership list, and need to test if a topic's top genes capture a known cell identity program.

## When NOT to use

- When the gene set of interest is not ranked by model association weights (e.g., it is an arbitrary feature table or a differential-expression list with no topic beta scores) — a simple ORA is sufficient and the topic-LFC correlation is not defined.
- When testing enrichment for pathways rather than a cell-type identity program validated by LFC correlation — the correlation-to-LFC check (Fig. 3f, g trend) is the paper's identity confirmation and would not apply.

## Inputs

- Fitted CODAL/MIRA expression topic model with an Ntopics x Ngenes beta association matrix
- HuBMAP cell-type ontology gene membership list (e.g., CD16+ monocyte ontology genes)
- Gene log-fold-change table for the target cell type (e.g., CD16+ monocytes)

## Outputs

- Top-200 gene ranking by topic weight
- 2x2 contingency table of top-200 membership vs. ontology membership
- One-sided Fisher's exact test enrichment p-value (expected 1.5e-19 for Topic 14 / CD16+ monocytes)
- Topic-score vs. gene log-fold-change correlation (Spearman/Pearson)
- topic14_enrichment_results.csv

## How to apply

Load the fitted CODAL/MIRA expression topic model and extract the Ntopics x Ngenes topic association matrix (beta). Rank all genes by their target topic weight and retain the top 200 genes, mirroring the paper's top-200-genes-per-topic selection convention. Build a 2x2 contingency table crossing the top-200 gene set (versus all remaining modeled genes) against HuBMAP cell-type ontology membership, then run a one-sided Fisher's exact test with scipy.stats.fisher_exact(alternative='greater') to obtain the enrichment p-value. As corroboration, compute the Spearman/Pearson correlation between the topic association scores and each gene's log-fold-change in the target cell type (reproducing the Fig. 3f, g comparison); a significant enrichment paired with correlated LFCs indicates the topic describes that cell type's identity program.

## Related tools

- **MIRA** (Provides the topic model whose Ntopics x Ngenes beta association matrix supplies the gene ranking to test; CODAL inference is built into MIRA topic models) — https://github.com/cistrome/MIRA
- **CODAL** (Variational topic-modeling extension that yields interpretable biological modules (topics) whose gene associations are tested for ontology enrichment) — https://mira-multiome.readthedocs.io
- **SciPy** (Supplies scipy.stats.fisher_exact with alternative='greater' for the one-sided enrichment test on the 2x2 contingency table)

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p = fisher_exact([[n_top_and_ontology, n_top_only],[n_ontology_only, n_rest]], alternative='greater')
```

## Evaluation signals

- The one-sided Fisher's exact p-value must reproduce the reported 1.5e-19 for Topic 14 top-200 genes vs. HuBMAP CD16+ monocyte ontology membership.
- The computed topic-association vs. gene log-fold-change correlation must match the trend displayed in Fig. 3f, g.
- The top-200 gene set must be a subset of the modeled (e.g., highly variable, ~3500-gene) feature space, with the contingency table second column populated by the remaining modeled genes.
- Output file topic14_enrichment_results.csv must contain the top-200 gene ranking, the Fisher p-value, and the topic-vs-LFC correlation.
- The contingency table margins must reconcile: top-200 count plus all remaining modeled genes equals the full modeled gene universe.

## Limitations

- Enrichment of ontology genes among topic-associated genes alone should be corroborated with orthogonal biological evidence; the authors caution that new or perturbation-induced states should be validated with orthogonal evidence to rule out under-correction of technical effects.
- The exact p-value (1.5e-19) depends on the paper's top-200-genes-per-topic convention and the specific ontology membership definition; deviating from these choices changes the result.
- A significant Fisher's exact test does not by itself establish the identity program — the topic-vs-LFC correlation is a required companion check.

## Evidence

- [other] Task card task_005 finding: [evidence span withheld — anchor status: not_found]
- [other] Workflow step 2: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract: "CODAL improves the representation of RNA-seq and ATAC-seq modalities, yields interpretable modules of biological variation"
- [intro] Article intro on MIRA/CODAL relationship: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [discussion] Article discussion validation guidance: "the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out the under-correction of technical effects"
