---
name: pdac-dataset-sample-filtering
description: Use when loading a public PDAC bulk expression dataset (e.g. TCGA-PAAD)
  from pdacR and the analysis requires only histologically confirmed adenocarcinoma
  samples; triggered by sample counts exceeding the whitelisted PDAC set or known
  contamination by non-PDAC tumors.
when_to_use_negative:
- The dataset has already been curated to histologically confirmed PDAC samples only
  (no non-PDAC labels remain in any sample track).
- The goal is to analyze non-adenocarcinoma samples present in the original repository
  data rather than tumor-cell-specific signatures.
- Analysis targets single-cell data (e.g. the rmoffitt/scOh atlas), which is outside
  pdacR's bulk-dataset filtering workflow.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: R package / RShiny app whose 'Filter samples by' option and 'Decision' sample
    track implement removal of non-whitelisted samples from parsed datasets such as
    TCGA-PAAD
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which pdacR is implemented and in which the filtered expression
    matrix can be subset programmatically before clustering or subtyping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:fddc61bfcd93fa0e5bb0319a79a71340ce8086a87d415e50db569fbdd29ee9c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - ConsensusClusterPlus
  - R
  - pdacR
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pdac-dataset-sample-filtering

## Summary

Applies the pdacR 'Filter samples by' mechanism to remove non-adenocarcinoma and low-purity cases from public PDAC transcriptomic datasets before downstream subtyping or signature analysis. This is essential because, e.g., TCGA-PAAD ships 182 mRNA samples of which only 150 are histologically confirmed PDAC.

## When to use

Use when loading a public PDAC bulk expression dataset (e.g. TCGA-PAAD) from pdacR and the analysis requires only histologically confirmed adenocarcinoma samples; triggered by sample counts exceeding the whitelisted PDAC set or known contamination by non-PDAC tumors.

## When NOT to use

- The dataset has already been curated to histologically confirmed PDAC samples only (no non-PDAC labels remain in any sample track).
- The goal is to analyze non-adenocarcinoma samples present in the original repository data rather than tumor-cell-specific signatures.
- Analysis targets single-cell data (e.g. the rmoffitt/scOh atlas), which is outside pdacR's bulk-dataset filtering workflow.

## Inputs

- pdacR parsed PDAC bulk expression dataset (e.g. TCGA-PAAD with 182 samples)
- Sample-level metadata / sample tracks (e.g. the 'Decision' track with 'whitelist' label)

## Outputs

- Filtered expression matrix restricted to the 150 whitelisted PDAC TCGA-PAAD samples
- Filtered sample tracks carried through to downstream heatmap, clustering, survival, and DE analyses

## How to apply

After selecting a dataset in pdacR (R package installed via devtools::install_github("rmoffitt/pdacR"), or its RShiny app), inspect the 'Filter samples by' column, which lists sample-level metadata fields for that dataset. For TCGA-PAAD, select the 'Decision' track and remove all variables except 'whitelist' so that only the 150 histologically confirmed PDAC samples of the original 182 are retained; checking a box removes all samples with that label from all subsequent analyses. Run this filtering before heatmap/clustering, Cartesian, survival, or differential expression steps, since unfiltered samples introduce non-specificity into gene signatures. For programmatic pipelines, apply the same Decision/whitelist subset to the expression matrix and sample tracks before writing per-sample outputs such as subtype assignments.

## Related tools

- **pdacR** (R package / RShiny app whose 'Filter samples by' option and 'Decision' sample track implement removal of non-whitelisted samples from parsed datasets such as TCGA-PAAD) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which pdacR is implemented and in which the filtered expression matrix can be subset programmatically before clustering or subtyping)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()  # then select TCGA-PAAD, 'Filter samples by' -> 'Decision', keep only 'whitelist'
```

## Evaluation signals

- TCGA-PAAD sample count drops from 182 to exactly 150 after applying the Decision/whitelist filter.
- All retained samples carry the 'whitelist' label on the 'Decision' sample track; no non-PDAC labels remain.
- Downstream subtype clustering is run only on the filtered matrix, so per-sample outputs (e.g. subtype_concordance.csv) contain ~150 TCGA rows, not 182.
- Filtering is applied before gene-signature derivation, consistent with the article's warning that including non-PDAC cases 'misleads signature derivation or validation'.

## Limitations

- Public PDAC transcriptomic data also contains low-purity cases that simple whitelist filtering may not fully exclude; these pitfalls 'can introduce non-specificity to gene signatures'.
- Filter labels are dataset-specific — the 'Decision'/whitelist track exists for TCGA-PAAD, and other datasets require inspecting their own 'Filter samples by' options.
- Bulk analyses limit the ability to fully separate cellular populations, so sample filtering cannot substitute for purity-aware methods.

## Evidence

- [readme] TCGA-PAAD has 182 samples, but only 150 are PDAC.: "TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [readme] To filter to the correct 150, select "Decision", and then remove all variables except "whitelist".: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [intro] The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases.: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases."
- [readme] Checking a box in this list will remove all samples with that label from all analyses.: "Checking a box in this list will remove all samples with that label from all analyses."
