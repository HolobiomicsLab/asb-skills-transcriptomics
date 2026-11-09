---
name: single-cell-signature-annotation
description: Use when single-cell RNA-seq data (AnnData object) must be scored or
  annotated for cellular senescence per cell type, or when literature-curated in vivo
  senescence gene signatures (e.g. Lung-fibroblast hubs) need to be extracted, searched,
  or merged.
when_to_use_negative:
- The data are already scaled — score_hub's default binarization and importance weighting
  assume raw or normalized (not scaled) expression; mimic scanpy.score_genes instead
  by setting binarize=False, importance=False after normalize_total and log1p.
- The goal is de novo discovery of senescence signatures from raw single-cell data
  rather than applying the published signatures — that requires the upstream GAD-scoring
  and co-positivity network workflow, not SenePy scoring.
- The input is bulk transcriptomic or non-single-cell data, since the signatures and
  scoring were developed and validated on single-cell transcriptomes.
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides the cell-type-specific senescence signature database and the load_hubs,
    get_genes, merge_hubs, search_hubs_by_genes, and score_hub functions used to extract,
    search, merge, and score signatures
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell expression object; score_hub results are written
    to adata.obs
- name: scanpy
  role: Optional preprocessing (scanpy.pp.normalize_total, sc.pp.log1p) when replicating
    scanpy.score_genes behavior with binarize=False, importance=False
- name: conda
  role: Environment setup for the Python 3.9 environment in which senepy is installed
    via pip
- name: pip
  role: Installs the senepy package (pip install senepy)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:13be6ff052f8eaf42eb19076f5e1f16b30b16abb1f54d1776b350552c4282633
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - conda
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-signature-annotation

## Summary

Annotate single-cell transcriptomic data with cell-type-specific senescence signatures using the SenePy database (75 mouse and 65 human signatures from 97 unique cell types), scoring cells via weighted hub gene signatures stored in AnnData/scanpy-compatible objects.

## When to use

Use when single-cell RNA-seq data (AnnData object) must be scored or annotated for cellular senescence per cell type, or when literature-curated in vivo senescence gene signatures (e.g. Lung-fibroblast hubs) need to be extracted, searched, or merged.

## When NOT to use

- The data are already scaled — score_hub's default binarization and importance weighting assume raw or normalized (not scaled) expression; mimic scanpy.score_genes instead by setting binarize=False, importance=False after normalize_total and log1p.
- The goal is de novo discovery of senescence signatures from raw single-cell data rather than applying the published signatures — that requires the upstream GAD-scoring and co-positivity network workflow, not SenePy scoring.
- The input is bulk transcriptomic or non-single-cell data, since the signatures and scoring were developed and validated on single-cell transcriptomes.

## Inputs

- AnnData object with single-cell gene expression matrix (raw or normalized, not scaled) for scoring
- Species selector ('Mouse' or 'Human') to load the SenePy hub database
- Optional gene list of interest (e.g. Cdkn2a, Cdkn1a, Il6, Cxcl13) for hub search
- Optional hubs.metadata subset (e.g. tissue == 'Lung') for signature merging

## Outputs

- Per-cell senescence scores stored as a new column in adata.obs (e.g. adata.obs['sp_score'])
- Gene signature lists keyed by (tissue, cell type, hub_num), e.g. hubs.get_genes(('Lung', 'fibroblast', 1))
- Hub metadata dataframe with size, n_sen, and hyp enrichment statistics per signature
- Merged tissue-specific or universal signatures with per-gene occurrence counts/weights
- Universal signature gene lists (Universal_mouse.txt, Universal_human.txt, FDR < 0.01)

## How to apply

Initialize the signature database with sp.load_hubs(species='Mouse' or 'Human'), which returns a hubs object whose .metadata dataframe lists tissue, cell type, and hub_num for each hub along with size, n_sen (count of the 180 literature-curated known senescence markers present), and hyp (hypergeometric enrichment p-value). Extract a specific signature via hubs.get_genes(('Lung', 'fibroblast', 1)) using the tissue-cell-hub_num key, or locate relevant hubs with hubs.search_hubs_by_genes on genes of interest such as Cdkn2a, Cdkn1a, Il6, and Cxcl13. Score cells with sp.score_hub(adata, hubs.hubs[key]), which by default binarizes the data and weighs each gene's importance by its number of connections in the hub network, writing the result to adata.obs (e.g. adata.obs['sp_score']); data need not be normalized but must not be scaled. To build tissue-specific or universal signatures, merge hubs with hubs.merge_hubs, optionally setting overlap_threshold (e.g. 2) or calculate_thresh=True with p_thres=0.01 for a permutation-based threshold; universal signatures are genes overrepresented at FDR p-value < 0.01.

## Related tools

- **SenePy** (Provides the cell-type-specific senescence signature database and the load_hubs, get_genes, merge_hubs, search_hubs_by_genes, and score_hub functions used to extract, search, merge, and score signatures) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell expression object; score_hub results are written to adata.obs)
- **scanpy** (Optional preprocessing (scanpy.pp.normalize_total, sc.pp.log1p) when replicating scanpy.score_genes behavior with binarize=False, importance=False)
- **conda** (Environment setup for the Python 3.9 environment in which senepy is installed via pip)
- **pip** (Installs the senepy package (pip install senepy))

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Loaded hubs.metadata matches the reported database scale: 75 mouse and 65 human signatures from 97 unique cell types
- Each hub's n_sen field reports a non-zero count of literature-curated markers from the 180-marker panel, and the hyp hypergeometric enrichment p-value is statistically significant
- get_genes(('Lung', 'fibroblast', 1)) returns the expected gene list for the Lung-fibroblast-1 hub and scores are written to adata.obs without altering expression values (data must not be scaled)
- Merged signatures respect the chosen cutoff: with calculate_thresh=True and p_thres=0.01, genes occurring below the 5% permutation-based threshold are removed from the universal signature
- Universal signature genes reproduce the FDR p-value < 0.01 overrepresentation criterion relative to Universal_mouse.txt / Universal_human.txt

## Limitations

- Universal signature files (Universal_mouse.txt, Universal_human.txt) carry no importance weights; weighted versions must be regenerated with merge_hubs(calculate_thresh=True).
- Merging a handful of input signatures with a raised overlap_threshold drastically reduces the number of genes in the resulting signature.
- With default settings score_hub binarizes data, making normalization and transformation optional but non-default; results differ from scanpy.score_genes unless binarize=False and importance=False are set on normalized, log-transformed data.
- The database does not account for genes downregulated in senescent cells (no negative markers), which limits sensitivity to signatures of CS repression.

## Evidence

- [readme] Database of 75 **mouse** and 65 **human** signatures of cellular senescence from 97 unique cell types: "Database of 75 **mouse** and 65 **human** signatures of cellular senescence from 97 unique cell types"
- [readme] n_sen marker counts and hyp enrichment p-value per hub: "n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub. hyp: enrichment p-value of the "known" and novel hub genes"
- [methods] score_hub default weighting and binarization: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation"
- [abstract] SenePy uses 72 mouse and 64 human weighted signatures: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] Universal signature FDR criterion: "These are lists of genes that are overrepresented in the senePy signatures (FDR p-value < 0.01), i.e., genes that are more universal across cell types."
