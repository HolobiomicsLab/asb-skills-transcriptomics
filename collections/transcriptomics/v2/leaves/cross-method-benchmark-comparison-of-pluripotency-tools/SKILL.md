---
name: cross-method-benchmark-comparison-of-pluripotency-tools
description: Use when per-cell stemness/pluripotency score tables (ORIGINS activity,
  activity-HVF, LandSCENT SR, CytoTRACE) already exist for multiple datasets and the
  goal is to quantify agreement between scoring methods via mean Pearson correlations.
when_to_use_negative:
- When the per-cell pluripotency scores have not yet been computed — this skill correlates
  existing score tables; score computation itself is done via ORIGINS' activity()
  function on the expression matrix and differentiation_edges PPIN.
- When raw count matrices (genes x cells) are the only input and the goal is trajectory
  inference rather than cross-method agreement — use the ORIGINS workflow to identify
  stem/progenitor origins instead.
- When datasets or tool combinations differ from the ORIGINS/LandSCENT/CytoTRACE benchmark,
  the specific benchmark averages (0.77, 0.44, 0.63, 0.67) are not valid validation
  targets.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Provides the differentiation PPIN activity score (and activity-HVF variant)
    that is one side of the correlation comparison; installed as an R package via
    remotes.
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Supplies the signaling resource (SR) pluripotency score per cell, correlated
    against ORIGINS activity and CytoTRACE.
- name: CytoTRACE
  role: Supplies a third per-cell pluripotency score used in the pairwise correlation
    benchmark.
- name: dplyr
  role: Used to summarize/average the per-dataset Pearson coefficients across the
    four datasets (an ORIGINS package import).
  repo: https://github.com/danielasenraoka/ORIGINS
- name: R
  role: Execution environment; cor() computes the Pearson correlations between score
    vectors.
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:ca27f6baf5d8763a3ccde183011608b5b81e1076699f9d70a85edd5b53ae9cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - dplyr
  - magrittr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# cross-method-benchmark-comparison-of-pluripotency-tools

## Summary

A reproducible R benchmarking procedure that computes pairwise Pearson correlations between per-cell pluripotency scores from ORIGINS (activity), LandSCENT (SR), and CytoTRACE across four healthy human scRNA-seq datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383) and averages them per score pair to compare methods.

## When to use

Use when per-cell stemness/pluripotency score tables (ORIGINS activity, activity-HVF, LandSCENT SR, CytoTRACE) already exist for multiple datasets and the goal is to quantify agreement between scoring methods via mean Pearson correlations.

## When NOT to use

- When the per-cell pluripotency scores have not yet been computed — this skill correlates existing score tables; score computation itself is done via ORIGINS' activity() function on the expression matrix and differentiation_edges PPIN.
- When raw count matrices (genes x cells) are the only input and the goal is trajectory inference rather than cross-method agreement — use the ORIGINS workflow to identify stem/progenitor origins instead.
- When datasets or tool combinations differ from the ORIGINS/LandSCENT/CytoTRACE benchmark, the specific benchmark averages (0.77, 0.44, 0.63, 0.67) are not valid validation targets.

## Inputs

- Per-cell score table for ORIGINS activity (one score per cell, per dataset)
- Per-cell score table for ORIGINS activity-HVF
- Per-cell score table for LandSCENT SR (signaling resource score)
- Per-cell score table for CytoTRACE scores
- Four healthy human scRNA-seq datasets: breast (GSE113197), colon (GSE125970), hematopoietic (GSE117498), lung (GSE161383)

## Outputs

- Pairwise Pearson correlation coefficients (activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, activity vs activity-HVF) per dataset
- Mean Pearson correlation per score pair averaged across the four datasets
- correlation_summary.csv containing computed and benchmark correlation values

## How to apply

Load the previously computed per-cell score tables for ORIGINS activity, activity-HVF, LandSCENT SR, and CytoTRACE for the breast (GSE113197), colon (GSE125970), hematopoietic (GSE117498), and lung (GSE161383) datasets into R. Use the cor() function to compute pairwise Pearson coefficients for activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, and activity vs activity-HVF within each dataset. Aggregate the per-dataset coefficients per score pair with dplyr::summarise (piping with magrittr is consistent with the ORIGINS package's imports). Compare each computed mean against the reported benchmark averages (activity vs SR ~0.77, activity vs CytoTRACE ~0.44, SR vs CytoTRACE ~0.63, activity vs activity-HVF ~0.67) within a small numerical tolerance and write the values to correlation_summary.csv.

## Related tools

- **ORIGINS** (Provides the differentiation PPIN activity score (and activity-HVF variant) that is one side of the correlation comparison; installed as an R package via remotes.) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Supplies the signaling resource (SR) pluripotency score per cell, correlated against ORIGINS activity and CytoTRACE.)
- **CytoTRACE** (Supplies a third per-cell pluripotency score used in the pairwise correlation benchmark.)
- **dplyr** (Used to summarize/average the per-dataset Pearson coefficients across the four datasets (an ORIGINS package import).) — https://github.com/danielasenraoka/ORIGINS
- **R** (Execution environment; cor() computes the Pearson correlations between score vectors.) — https://github.com/danielasenraoka/ORIGINS

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- Mean Pearson correlation for activity (ORIGINS) vs SR (LandSCENT) reproduces ~0.77 within small numerical tolerance
- Mean Pearson correlation for activity (ORIGINS) vs CytoTRACE reproduces ~0.44 within tolerance
- Mean Pearson correlation for SR (LandSCENT) vs CytoTRACE reproduces ~0.63 within tolerance
- Mean Pearson correlation for activity vs activity-HVF reproduces ~0.67 within tolerance
- All four per-dataset coefficient sets are present before averaging, and results are written to correlation_summary.csv

## Limitations

- The benchmark averages were derived only from four healthy human datasets (breast, colon, hematopoietic, lung); they are not guarantees for diseased, other-species, or other tissue data.
- Correlation agreement does not imply methodological equivalence — the article shows the methods differ in efficiency and RAM requirements (ORIGINS is more efficient and requires less RAM than LandSCENT and CytoTRACE).
- The correlation step requires previously computed score tables; no guidance here for computing LandSCENT SR or CytoTRACE scores beyond their citation.

## Evidence

- [abstract] Benchmark against LandSCENT and CytoTRACE: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
- [abstract] ORIGINS efficiency claim: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [methods] ORIGINS activity computation call: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [supplementary] GEO accession codes for benchmark datasets: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] dplyr dependency: "If you use this code in your research, please cite our publication: [Senra, D., Guisoni, N., & Diambra, L. A. (2022). ORIGINS: A protein network-based approach to quantify cell pluripotency from"
