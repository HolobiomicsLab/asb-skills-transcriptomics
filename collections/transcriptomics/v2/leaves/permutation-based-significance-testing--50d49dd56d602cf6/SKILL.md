---
name: permutation-based-significance-testing
description: Use when you must test whether gene signatures (e.g., age-dynamic senescence
  genes in a cell type) exceed what null label permutations would produce, or set
  a data-driven overlap cutoff when merging SenePy hubs.
when_to_use_negative:
- You need negative (downregulated) senescence markers — the methodology only accounts
  for genes upregulated/positively correlated, and finding negative correlations in
  all pairwise gene combinations was computationally limiting
- Your goal is classical differential expression between conditions rather than in
  vivo senescence program detection — the paper notes DE analyses poorly capture in
  vivo cellular senescence
- Your data are already a merged or universal signature list (e.g., Universal_mouse.txt)
  with no per-hub occurrence counts to threshold
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs, merge_hubs (with calculate_thresh permutation-based thresholding),
    and hubs.metadata (n_sen, hyp) used to run the permutation tests and enrichment
    statistics
  repo: https://github.com/jaleesr/senepy
- name: Python
  role: Language in which SenePy is implemented and the permutation and hypergeometric
    testing workflows are executed
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the single-cell expression matrix and age labels over
    which GAD scores and null permutations are computed
- name: scanpy
  role: Single-cell preprocessing (normalize_total, log1p) upstream of hub scoring
    and significance assessment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:9271d33d486270e90d9d1fb4917f5261084625becff3c33e1cb84594d96f87d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# permutation-based-significance-testing

## Summary

Assess significance of gene-level senescence signals (age-dynamic genes, hub-gene occurrence thresholds) by comparing observed statistics against null permutations of age labels or a permutation-derived random-occurrence threshold, as implemented in the SenePy workflow on Tabula Muris Senis.

## When to use

Use when you must test whether gene signatures (e.g., age-dynamic senescence genes in a cell type) exceed what null label permutations would produce, or set a data-driven overlap cutoff when merging SenePy hubs.

## When NOT to use

- You need negative (downregulated) senescence markers — the methodology only accounts for genes upregulated/positively correlated, and finding negative correlations in all pairwise gene combinations was computationally limiting
- Your goal is classical differential expression between conditions rather than in vivo senescence program detection — the paper notes DE analyses poorly capture in vivo cellular senescence
- Your data are already a merged or universal signature list (e.g., Universal_mouse.txt) with no per-hub occurrence counts to threshold

## Inputs

- AnnData object of single-cell transcriptomes with age labels (e.g., Tabula Muris Senis, 328K cells, 19 tissues, ages 1–30 months)
- SenePy hubs.metadata dataframe of gene signatures per tissue, cell type, and hub_num
- List of literature-curated known senescence markers (180 markers) for hypergeometric enrichment

## Outputs

- Per-gene significance calls for age-dynamic (GAD) genes per cell type against permuted age labels
- Permutation-derived overlap threshold for merged hub signatures (calculate_thresh=True, e.g., p_thres=0.01)
- Hub metadata statistics: size, n_sen (known-marker count), hyp (hypergeometric enrichment p-value)

## How to apply

For each cell type in the Tabula Muris Senis AnnData object (328K cells, ages 1–30 months), compute age-dynamic (GAD) gene scores for candidate senescence-associated genes and assess significance against null permutations of the age labels rather than raw p-values, because differential expression poorly captures in vivo cellular senescence. When merging signatures with SenePy's merge_hubs, either set an explicit overlap_threshold (e.g., keep genes occurring at least 2 times) or set calculate_thresh=True to derive a threshold automatically by permutation: with p_thres=0.01 a 5% random-occurrence cutoff is computed and genes occurring less frequently than that are removed from the merged hub. Hub significance is then quantified in hubs.metadata fields n_sen (count of the 180 literature-curated known senescence markers present) and hyp (hypergeometric enrichment p-value). Validate by confirming each derived hub shows statistically significant enrichment of known senescence markers, consistent with the reported database of 75 mouse and 65 human signatures.

## Related tools

- **SenePy** (Provides load_hubs, merge_hubs (with calculate_thresh permutation-based thresholding), and hubs.metadata (n_sen, hyp) used to run the permutation tests and enrichment statistics) — https://github.com/jaleesr/senepy
- **Python** (Language in which SenePy is implemented and the permutation and hypergeometric testing workflows are executed) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the single-cell expression matrix and age labels over which GAD scores and null permutations are computed)
- **scanpy** (Single-cell preprocessing (normalize_total, log1p) upstream of hub scoring and significance assessment)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); hubs.merge_hubs(hubs.metadata, new_name='universal', calculate_thresh=True, p_thres=0.01)
```

## Evaluation signals

- Exactly 72 mouse putative CS signatures are produced, matching Supplementary Data 4 and 5
- Each hub's hyp (hypergeometric enrichment p-value) over the 180 known senescence markers is statistically significant, consistent with the reported 75-mouse-signature database
- When calculate_thresh=True with p_thres=0.01, genes in the merged hub occur at or above the permutation-derived random-occurrence count (5% random chance threshold)
- hubs.metadata rows carry non-empty tissue, cell type, and hub_num fields with size equal to the number of genes returned by get_genes
- GAD gene significance is stable under re-running age-label permutations (null-based, not DE-based)

## Limitations

- The methodology discounts genes that may be part of the CS program but overlap with the transcriptional shift of organismal aging
- Genes downregulated in senescent cells are not accounted for (no negative markers); testing negative correlations in all pairwise gene combinations was computationally limiting
- High dropout and limited sequencing depth make classical markers like Cdkn2a and Cdkn1a unreliable as sole indicators, motivating the permutation/enrichment approach
- An explicit overlap_threshold (e.g., 2) drastically reduces the number of genes in the resulting signature when merging only a handful of input signatures

## Evidence

- [methods] permutation threshold in merge_hubs: "We can also calculate a threshold automatically using a permutation-based method"
- [readme] 5% random chance threshold: "We then use a 5% random chance threshold and remove all the genes that occur less than that number of times."
- [other] GAD scores tested against null permutations: [evidence span withheld — anchor status: not_found]
- [readme] hypergeometric enrichment of known markers: "n_sen: number of 180 prior "known" senescence markers curated from literature that were found in the hub."
- [discussion] negative correlations computationally limiting: "to find negative correlations in all pairwise combinations of genes with this methodology was computationally limiting"
