---
name: gmt-gene-set-signature-handling
description: Use when scoring single-cell AnnData with a curated signature (e.g.,
  a 12-gene injury signature) that must be encoded in a .gmt gene set file with _UP/_DN
  nomenclature before calling gssnng scoring functions.
when_to_use_negative:
- The signature is not a fixed gene list — e.g., a decoupler/Omnipath regulatory model,
  which should be scored via the Decoupler/Omnipath style API instead of a .gmt file
- The analysis calls for unranked aggregations such as mean_z, robust_std, or median_score
  on raw counts, where rank ordering and _UP/_DN rank-based handling are unnecessary
- The goal is group-level rather than per-cell scoring on already-pseudobulked profiles,
  where smoothing-based gssnng scoring is not the right entry point
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Reads the .gmt gene set file and scores AnnData cells per gene set into adata.obsm['gssnng_estimate']
    using scorers such as ssGSEA and rank_biased_overlap
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Provides the AnnData object (h5ad) that carries the scored gene set columns
- name: MSigDB
  role: 'Source of nomenclature convention: gene sets default to up, enabling _UP/_DN
    suffix encoding of two-part signatures'
- name: decoupleR
  role: Decoupler/Omnipath style API used to extract activities as an AnnData object
    via dc.get_acts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:f9baa6d753853055224a2e1fec62ffa66f942715499d5ff252dbbbe37fcc7966
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - MSigDB
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gmt-gene-set-signature-handling

## Summary

Prepare and supply gene set signatures as .gmt files for gssnng cell scoring, using MSigDB nomenclature where gene sets default to up and the _UP/_DN suffix convention to encode two-part (up-regulated and down-regulated) signatures for scorers such as rank_biased_overlap and ssGSEA.

## When to use

Use when scoring single-cell AnnData with a curated signature (e.g., a 12-gene injury signature) that must be encoded in a .gmt gene set file with _UP/_DN nomenclature before calling gssnng scoring functions.

## When NOT to use

- The signature is not a fixed gene list — e.g., a decoupler/Omnipath regulatory model, which should be scored via the Decoupler/Omnipath style API instead of a .gmt file
- The analysis calls for unranked aggregations such as mean_z, robust_std, or median_score on raw counts, where rank ordering and _UP/_DN rank-based handling are unnecessary
- The goal is group-level rather than per-cell scoring on already-pseudobulked profiles, where smoothing-based gssnng scoring is not the right entry point

## Inputs

- Gene set / signature definition (e.g., a 7-UP / 5-DN two-part injury signature) encoded as a .gmt file (one gene set per line, Broad GMT format)
- Scanpy AnnData object (h5ad) whose var_names contain the signature genes
- Optional method_params dict per scorer (e.g., {'omega': 0.75} for ssGSEA, {'rbo_depth': n} for rank_biased_overlap)

## Outputs

- Per-cell gene set score columns in adata.obsm['gssnng_estimate'], one column per gene set in the .gmt file
- Activities AnnData extracted via dc.get_acts(adata, obsm_key='gssnng_estimate')
- Downstream labeled score tables (e.g., auc_injury_signature.csv) computed from the scored columns

## How to apply

Format the signature as a .gmt file, one gene set per line, following the Broad/MSigDB GMT definition; gssnng follows the MSigDB nomenclature where gene sets default to up, so append '_DN' to a set name to mark a down-regulated component and '_UP' for the up-regulated component of a two-part signature. Feed the same file to gssnng's score_cells.with_gene_sets for both rank-based scorers: rank_biased_overlap (ranked=True, optional method_params {'rbo_depth': n}) and ssGSEA (ranked=True, method_params {'omega': 0.75}). Scores for each gene set are written per cell into adata.obsm['gssnng_estimate'], one column per gene set, which can be extracted as an AnnData of activities via decoupler's dc.get_acts. Decide whether rank-based scorers apply: ssGSEA, singscore, and rank_biased_overlap require ranked data, while mean_z, robust_std, median_score, and summed_up are recommended unranked.

## Related tools

- **gssnng** (Reads the .gmt gene set file and scores AnnData cells per gene set into adata.obsm['gssnng_estimate'] using scorers such as ssGSEA and rank_biased_overlap) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Provides the AnnData object (h5ad) that carries the scored gene set columns)
- **MSigDB** (Source of nomenclature convention: gene sets default to up, enabling _UP/_DN suffix encoding of two-part signatures)
- **decoupleR** (Decoupler/Omnipath style API used to extract activities as an AnnData object via dc.get_acts)

## Examples

```
from gssnng import score_cells; score_cells.with_gene_sets(adata, gene_set_file='injury_signature.gmt', groupby='cell_type', score_method='ssGSEA', method_params={'omega': 0.75}, ranked=True, smooth_mode='off', recompute_neighbors=0)
```

## Evaluation signals

- The .gmt file parses as valid Broad GMT: each line is one gene set with name and gene list, and _UP/_DN suffixes correctly designate the up- and down-regulated components
- After scoring, adata.obsm['gssnng_estimate'] contains exactly one column per gene set in the .gmt file
- Rank-requiring scorers (ssGSEA, singscore, rank_biased_overlap) are run with ranked=True; ssGSEA uses method_params={'omega': 0.75}
- Downstream AUC separation is sensible: for the Squair spinal-injury signature, smoothing lifted AUC from 0.62 to 0.71 (RBO) and 0.69 to 0.8 (ssGSEA)
- All computed scores with scorer and condition labels are fully written out (e.g., four labeled AUC values in auc_injury_signature.csv)

## Limitations

- The README's method_params section only documents options for singscore ({'normalization','theoretical'/'standard'}), rank_biased_overlap ({'rbo_depth'}), and ssGSEA ({'omega': 0.75}); other scorers' extra options are not documented there
- rank_biased_overlap descends only to a fixed rbo_depth n down the ranks, so scores depend on the chosen depth
- Two-part signatures rely on the _UP/_DN naming convention — a .gmt without this encoding scores sets as all-up by default under MSigDB nomenclature
- The source article's discussion section was empty, providing no changelog or extended caveats

## Evidence

- [readme] ssGSEA:         Single sample GSEA based on ranked data.: "ssGSEA:         Single sample GSEA based on ranked data."
- [readme] rank_biased_overlap:  RBO, Weighted average of agreement between sorted ranks and gene set.: "rank_biased_overlap:  RBO, Weighted average of agreement between sorted **ranks** and gene set."
- [methods] We are following the MSigDB nomenclature, where gene sets default to up: "We are following the MSigDB nomenclature, where gene sets default to up"
- [methods] 3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] ssGSEA: {'omega': 0.75}: "ssGSEA: {'omega': 0.75}"
