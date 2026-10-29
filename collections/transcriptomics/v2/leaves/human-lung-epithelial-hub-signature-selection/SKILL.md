---
name: human-lung-epithelial-hub-signature-selection
description: Use when scoring senescence in human lung scRNAseq (AnnData) with SenePy,
  e.g., to compare senescent epithelial cell burden between COVID-19 decedents and
  age-matched controls.
when_to_use_negative:
- The adata.X matrix is already z-score scaled — score_hub defaults assume unscaled
  expression and scaling will corrupt the binarize/importance-weighted scoring
- The data are mouse scRNAseq — this skill uses sp.load_hubs(species='Human') and
  human gene symbols; Tabula Muris Senis-style mouse data requires the Mouse hub set
- The goal is scoring a non-lung cell type or tissue — instead select the matching
  hub from hubs.metadata or build a merged signature with hubs.merge_hubs()
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3391
tools:
- name: SenePy
  role: Provides load_hubs/get_genes/score_hub to select the human lung epithelial
    hub signature and score cells
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell lung expression matrix and obs metadata that
    receives the sp_score column
- name: scanpy
  role: Optional preprocessing (normalize_total, log1p) of the AnnData object; data
    must not be scaled
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:cb4b101e62dbee044e853254927c1bd41a70ec166c0a100e3fd5713a304108e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# human-lung-epithelial-hub-signature-selection

## Summary

Selects the appropriate human lung epithelial senescence hub signature(s) from the SenePy database and applies them to score single-cell transcriptomes, enabling quantification of cellular senescence burden in disease contexts such as COVID-19 lung datasets.

## When to use

Use when scoring senescence in human lung scRNAseq (AnnData) with SenePy, e.g., to compare senescent epithelial cell burden between COVID-19 decedents and age-matched controls.

## When NOT to use

- The adata.X matrix is already z-score scaled — score_hub defaults assume unscaled expression and scaling will corrupt the binarize/importance-weighted scoring
- The data are mouse scRNAseq — this skill uses sp.load_hubs(species='Human') and human gene symbols; Tabula Muris Senis-style mouse data requires the Mouse hub set
- The goal is scoring a non-lung cell type or tissue — instead select the matching hub from hubs.metadata or build a merged signature with hubs.merge_hubs()

## Inputs

- Human single-cell gene expression AnnData object (raw or normalized/log1p, NOT scaled)
- SenePy human hub database loaded via sp.load_hubs(species='Human')
- Per-cell metadata in adata.obs including sample ID and condition labels (COVID-19 decedent vs control)

## Outputs

- Per-cell senescence scores stored in adata.obs['sp_score']
- Per-sample mean epithelial senescence scores with outlier annotations (covid_epithelial_senescence_scores.csv)
- Mann-Whitney U test p-value comparing decedent vs control per-sample scores (expected ~0.004)

## How to apply

Load the human hub database with sp.load_hubs(species='Human') and inspect hubs.metadata to identify lung epithelial hubs (tissue, cell type, hub_num). Extract the desired signature via hubs.get_genes(('Lung', cell, hub_num)) or index hubs.hubs[(tissue, cell, hub_num)] directly. Score the AnnData object with sp.score_hub(adata, hub), keeping the defaults (binarize=True and importance weighting by hub-network gene connections); the input expression values may be raw or normalized but must NOT be scaled. Store per-cell scores in adata.obs['sp_score'], aggregate to per-sample means, and compare groups (e.g., COVID-19 decedent vs control) with a two-sided Mann-Whitney U test. The COVID-19 lung epithelial analysis in the article should reproduce a significant increase in senescent AT1/AT2/airway cells in decedents at approximately p = 0.004.

## Related tools

- **SenePy** (Provides load_hubs/get_genes/score_hub to select the human lung epithelial hub signature and score cells) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell lung expression matrix and obs metadata that receives the sp_score column)
- **scanpy** (Optional preprocessing (normalize_total, log1p) of the AnnData object; data must not be scaled)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Human'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'AT2', 1)])
```

## Evaluation signals

- hubs.metadata rows confirm the selected signature is a (Lung, epithelial cell type, hub_num) human hub
- Scored cells have a finite numeric adata.obs['sp_score'] and the AnnData expression matrix was not scaled
- Per-sample mean epithelial scores for COVID-19 decedents are significantly higher than controls (two-sided Mann-Whitney, p ≈ 0.004)
- Increased senescence is detected in AT1, AT2, and airway epithelial populations, not restricted to one subtype
- covid_epithelial_senescence_scores.csv contains per-sample scores, outlier annotations, and the Mann-Whitney result

## Limitations

- SenePy accounts only for genes upregulated in senescent cells — downregulated genes are not represented (no negative markers)
- Classical markers like Cdkn2a/Cdkn1a have low detectability in single-cell data due to transient p16 expression and gene dropout, motivating hub-based scoring instead
- The high dropout rate of single-cell transcriptomics makes single-gene senescence markers unreliable, so scores depend on multi-gene hub coverage

## Evidence

- [readme] Example scoring and saving as a new column in anndata.obs. With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [methods] Binarize and importance weighting defaults: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] Importance weighting rationale: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [methods] Hub extraction convention: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [abstract] SenePy designed for disease-context scoring: "We use SenePy to map the kinetics of senescent cell accumulation in healthy aging as well as multiple disease contexts, including tumorigenesis, inflammation, and myocardial infarction."
