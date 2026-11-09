---
name: per-sample-outlier-detection
description: Use when per-cell SenePy hub scores must be summarized per biological
  sample and tested between disease groups, or when individual samples may distort
  group-level senescence comparisons.
when_to_use_negative:
- Input expression values are scaled — SenePy scoring defaults assume raw or normalized
  (optionally log-transformed) data, and scaled values invalidate the binarized hub
  scoring
- The analysis goal is scoring senescent cells against arbitrary gene lists rather
  than published senescence signatures from the SenePy database
- Only one sample per condition exists, so per-sample aggregation and group-level
  Mann-Whitney comparison are not possible
tools:
- name: SenePy
  role: Loads human lung epithelial senescence hubs and scores single cells (score_hub)
    to produce the per-cell sp_score values aggregated per sample
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Holds the lung scRNAseq expression matrix and stores per-cell sp_score values
    in adata.obs for per-sample aggregation
- name: scanpy
  role: Python ecosystem used alongside SenePy to load and preprocess the AnnData
    lung datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:35c0f6e46e4128e9902fb1ff75557a86b376c222d27dab81885a93d7c63f6b6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Per-sample outlier detection

## Summary

Aggregates SenePy per-cell senescence scores (sp_score) to per-sample mean epithelial scores, flags samples deviating from the cohort distribution as outliers, and statistically compares COVID-19 decedent vs. control per-sample scores with a two-sided Mann-Whitney U test.

## When to use

Use when per-cell SenePy hub scores must be summarized per biological sample and tested between disease groups, or when individual samples may distort group-level senescence comparisons.

## When NOT to use

- Input expression values are scaled — SenePy scoring defaults assume raw or normalized (optionally log-transformed) data, and scaled values invalidate the binarized hub scoring
- The analysis goal is scoring senescent cells against arbitrary gene lists rather than published senescence signatures from the SenePy database
- Only one sample per condition exists, so per-sample aggregation and group-level Mann-Whitney comparison are not possible

## Inputs

- AnnData scRNAseq object of COVID-19 decedent and age-matched control human lung, raw or normalized but NOT scaled
- SenePy hub object with human lung epithelial (AT1, AT2, airway) signatures from hubs.metadata
- Per-cell senescence scores stored in adata.obs['sp_score']
- Sample-to-condition annotation (decedent vs. control)

## Outputs

- Per-sample mean epithelial senescence scores
- Outlier annotations flagging samples deviating from the cohort distribution
- Two-sided Mann-Whitney U test result comparing decedent vs. control per-sample scores
- Consolidated results table, e.g., covid_epithelial_senescence_scores.csv

## How to apply

Score cells per sample using sp.score_hub on lung epithelial hubs from sp.load_hubs(species='Human'), storing results in adata.obs['sp_score'] with default binarize=True and hub-network importance weighting. Aggregate per-cell scores to a single mean senescence score per sample, then flag outlier samples that deviate from the cohort distribution of per-sample mean scores before group testing. Compare COVID-19 decedent vs. age-matched control per-sample scores with a two-sided Mann-Whitney U test; the reported validation result is increased senescent AT1/AT2/airway epithelial cells in decedents at approximately p = 0.004. Persist per-sample scores, outlier annotations, and the Mann-Whitney result (e.g., covid_epithelial_senescence_scores.csv) so the aggregation and outlier decisions are auditable.

## Related tools

- **SenePy** (Loads human lung epithelial senescence hubs and scores single cells (score_hub) to produce the per-cell sp_score values aggregated per sample) — https://github.com/jaleesr/senepy
- **AnnData** (Holds the lung scRNAseq expression matrix and stores per-cell sp_score values in adata.obs for per-sample aggregation)
- **scanpy** (Python ecosystem used alongside SenePy to load and preprocess the AnnData lung datasets)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Human'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'epithelial', 1)]); from scipy.stats import mannwhitneyu; mannwhitneyu(covid_sample_scores, control_sample_scores, alternative='two-sided')
```

## Evaluation signals

- Two-sided Mann-Whitney U test on per-sample epithelial senescence scores reproduces the reported increased senescence in COVID-19 decedents at approximately p = 0.004
- Senescent epithelial signal is present in AT1, AT2, and airway epithelial populations, not restricted to a single subpopulation
- adata.obs contains an 'sp_score' column for every scored cell, with defaults binarize=True and importance weighting applied
- Every per-sample score in the output CSV traces back to a non-empty set of per-cell scores, and outlier samples are explicitly annotated rather than silently dropped
- Input adata values are raw or normalized but not scaled before score_hub was called

## Limitations

- score_hub defaults binarize the data; switching to scanpy.score_genes-style behavior requires normalize_total plus log1p and setting binarize=False, importance=False, so aggregation statistics are sensitive to these parameter choices
- Merged/universal signatures lose importance weights unless derived via merge_hubs, limiting weighted scoring strategies
- High dropout and transient marker expression (e.g., Cdkn2a/p16) in single-cell data make classical markers unreliable, so per-sample conclusions depend on the multi-gene hub scores rather than individual genes

## Evidence

- [results] COVID-19 mortality and epithelial senescence: [evidence span withheld — anchor status: not_found]
- [readme] score_hub default behavior: "Example scoring and saving as a new column in anndata.obs. With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [readme] score_hub options affecting aggregation: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data"
- [methods] hub metadata structure: "hubs.metadata is a dataframe of hubs. It contains which tissue and cell type the hub was derived from as well as the hub_num if there were multiple senescence hubs within that cell type."
- [discussion] clinical dropout limitation: "the high rate of dropout and limited sequencing depth of the technology poses a challenge for using classical CS markers such as Cdkn2a and Cdkn1a as sole indicators"
