---
name: gene-set-enrichment-testing-via-fishers-exact-test
description: Use when a fitted topic model (e.g., CODAL/MIRA β matrix) yields a ranked
  gene list and you need to test overrepresentation of a curated cell-type ontology
  set — e.g., HuBMAP CD16+ monocyte membership — among the top-ranked genes.
when_to_use_negative:
- The gene ranking is not from an interpretable topic/β matrix — e.g., input is already
  a feature table or differential-expression list where a dedicated overrepresentation
  tool is more appropriate.
- The query gene set is not defined against a fixed universe of modeled genes (contingency
  table requires a well-defined background of all remaining modeled genes).
- You need batch-corrected latent representations or batch-confounded cell-state detection
  rather than set-enrichment statistics — use CODAL's disentangling workflow instead.
edam_operation: http://edamontology.org/operation_3684
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: MIRA
  role: Source of the fitted topic model from which the Ntopics x Ngenes β association
    matrix is extracted for gene ranking
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Batch-corrected variational topic modeling (extends MIRA) whose Topic 14 gene
    associations are the input to the enrichment test
  repo: https://github.com/cistrome/MIRA
- name: SciPy
  role: Provides scipy.stats.fisher_exact (alternative='greater') for the one-sided
    2x2 enrichment test
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:653b3a7f0c174e3e432079696a4acae862f7fe910aed101303b06deaca155d00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - SciPy
  edam_operation: http://edamontology.org/operation_3684
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Gene-set enrichment testing via Fisher's exact test

## Summary

Tests whether a ranked gene set (e.g., the top 200 genes associated with a CODAL/MIRA topic, ranked by their β-matrix weight) is significantly overrepresented for members of a curated ontology set, using a one-sided Fisher's exact test on a 2x2 contingency table. Used here to confirm Topic 14 captures the HuBMAP CD16+ monocyte identity program (p = 1.5e-19).

## When to use

Use when a fitted topic model (e.g., CODAL/MIRA β matrix) yields a ranked gene list and you need to test overrepresentation of a curated cell-type ontology set — e.g., HuBMAP CD16+ monocyte membership — among the top-ranked genes.

## When NOT to use

- The gene ranking is not from an interpretable topic/β matrix — e.g., input is already a feature table or differential-expression list where a dedicated overrepresentation tool is more appropriate.
- The query gene set is not defined against a fixed universe of modeled genes (contingency table requires a well-defined background of all remaining modeled genes).
- You need batch-corrected latent representations or batch-confounded cell-state detection rather than set-enrichment statistics — use CODAL's disentangling workflow instead.

## Inputs

- Fitted CODAL/MIRA expression topic model β matrix (Ntopics x Ngenes)
- Ranked gene list per topic (top 200 genes by Topic 14 weight)
- HuBMAP CD16+ monocyte ontology membership annotations (gene-to-set binary mapping)
- Gene log-fold-change (LFC) values in CD16+ monocytes

## Outputs

- 2x2 contingency table (top-200 vs. remaining genes x ontology membership)
- One-sided Fisher's exact enrichment p-value (expected ≈ 1.5e-19)
- Topic association-vs-LFC correlation statistic (Spearman/Pearson)
- topic14_enrichment_results.csv with top-200 gene ranking, p-value, and correlation

## How to apply

Extract the Ntopics x Ngenes topic association matrix (beta) from the fitted CODAL/MIRA topic model, rank all modeled genes by their Topic 14 weight, and retain the top 200 genes following the paper's top-200-genes-per-topic convention. Build a 2x2 contingency table crossing the top-200 gene set versus all remaining modeled genes against HuBMAP CD16+ monocyte ontology membership, then run a one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater') to obtain the enrichment p-value. As a complementary check of functional agreement, compute the Spearman/Pearson correlation between Topic 14 association scores and each gene's log-fold-change in CD16+ monocytes, reproducing the Fig. 3f, g comparison. Write the top-200 ranking, p-value, and correlation to topic14_enrichment_results.csv.

## Related tools

- **MIRA** (Source of the fitted topic model from which the Ntopics x Ngenes β association matrix is extracted for gene ranking) — https://github.com/cistrome/MIRA
- **CODAL** (Batch-corrected variational topic modeling (extends MIRA) whose Topic 14 gene associations are the input to the enrichment test) — https://github.com/cistrome/MIRA
- **SciPy** (Provides scipy.stats.fisher_exact (alternative='greater') for the one-sided 2x2 enrichment test)

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p = fisher_exact([[n_top_in_ontology, n_top_not], [n_rest_in_ontology, n_rest_not]], alternative='greater')
```

## Evaluation signals

- The one-sided Fisher's exact p-value reproduces the reported 1.5e-19 for CD16+ monocyte ontology enrichment among the top 200 Topic 14-associated genes.
- The computed Topic 14 association-vs-LFC correlation matches the trend displayed in Fig. 3f, g.
- The top-200 gene selection mirrors the paper's top-200-genes-per-topic convention (not top-N by another threshold).
- The contingency table background comprises all remaining modeled genes (e.g., the 3500 highly variable genes entering the model), not an arbitrary gene universe.
- Output CSV topic14_enrichment_results.csv contains the ranking, p-value, and correlation columns.

## Limitations

- The reported p-value (1.5e-19) is specific to this dataset/model fit; other topics or datasets will yield different values, and the validation target applies only to Topic 14 with the HuBMAP CD16+ monocyte set.
- Enrichment of topic-associated genes in an ontology set should be corroborated with orthogonal evidence (e.g., LFC correlation) rather than treated as standalone functional validation.
- Results depend on upstream CODAL/MIRA model fit quality; the article cautions that batch-confounded states require orthogonal biological validation to rule out under-correction of technical effects.

## Evidence

- [methods] one-sided Fisher's exact test on 2x2 ontology contingency table: [evidence span withheld — anchor status: not_found]
- [results] reported enrichment result for Topic 14: [evidence span withheld — anchor status: not_found]
- [intro] topic model provenance of the gene ranking: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [readme] MIRA provides functional enrichment methods used by this skill: "* Multimodal topic modeling
* Construction a joint representation of cells
* Regulator and functional enrichment"
