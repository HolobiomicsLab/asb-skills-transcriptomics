---
name: nonnegative-matrix-factorization-decomposition
description: Use when comparing factorization choices for single-cell program recovery
  (NMF vs ICA), or when decomposing nonnegative scRNA count matrices into latent gene
  expression programs against simulation ground truth.
when_to_use_negative:
- Input matrices contain negative values (NMF requires nonnegative factorization inputs).
- The goal is joint low-dimensional representation across already-collected real cohorts
  rather than program recovery benchmarking — the article's discussion points to integration
  algorithms such as Harmony and scArches for that setting.
- Independent-component structure among programs is the explicit objective — MetaTiME
  used ICA 'to optimize information independence among components', and ICA slightly
  outperformed NMF at recovering pre-embedded GEPs in simulation.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides the NMF implementation used for the NMF-based program-recovery pipeline
    on pooled simulated count matrices
- name: scsim
  role: Simulator used to generate synthetic single-cell count matrices with pre-embedded
    gene expression programs to serve as NMF-recovery ground truth
- name: MetaTiME
  role: ICA-based meta-component recovery pipeline against which the NMF decomposition
    is benchmarked
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:9dc3b34b7891b673764066471119d830d18f759c9fdff87e44abf7b21c3fc8c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - scikit-learn
  - scsim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nonnegative-matrix-factorization-decomposition

## Summary

Applying NMF-based matrix factorization to decompose pooled single-cell expression matrices into nonnegative gene expression programs (GEPs), used in the MetaTiME work as the comparator to ICA for recovering pre-embedded transcriptional programs in simulated multi-cohort data.

## When to use

Use when comparing factorization choices for single-cell program recovery (NMF vs ICA), or when decomposing nonnegative scRNA count matrices into latent gene expression programs against simulation ground truth.

## When NOT to use

- Input matrices contain negative values (NMF requires nonnegative factorization inputs).
- The goal is joint low-dimensional representation across already-collected real cohorts rather than program recovery benchmarking — the article's discussion points to integration algorithms such as Harmony and scArches for that setting.
- Independent-component structure among programs is the explicit objective — MetaTiME used ICA 'to optimize information independence among components', and ICA slightly outperformed NMF at recovering pre-embedded GEPs in simulation.

## Inputs

- Nonnegative single-cell count matrices from individual simulated or real cohorts (e.g. scsim-generated synthetic scRNA-seq with embedded GEPs)
- Ground-truth gene expression program definitions (14 programs per simulation: 13 cell-type-specific + 1 signaling)
- Cohort-pooling configuration (settings of 5, 10, and 20 pooled datasets)

## Outputs

- Recovered NMF components matched to known GEPs with per-program recovery scores
- Comparison table ICA vs NMF across cohort sizes 5/10/20 (simulation_benchmark_results.csv)

## How to apply

Generate or load nonnegative count matrices (here: 20 scsim-simulated cohorts, each with 14 embedded GEPs — 13 cell-type-specific plus 1 signaling program). Pool datasets at cohort-size settings of 5, 10, and 20, and run the NMF pipeline on each pooled matrix, using the same number of components as embedded GEPs (14). Match each recovered NMF component back to the 14 known GEPs per simulation (e.g. correlation-based assignment) and compute per-program recovery scores at each cohort-size setting. Aggregate scores into an ICA vs NMF and 5 vs 10 vs 20 cohort comparison table written to simulation_benchmark_results.csv. Decision point: in the MetaTiME benchmark ICA recovered the true GEPs slightly better than NMF, so NMF is the baseline to beat, and recovery should improve monotonically as pooled cohorts increase from 5 to 10 to 20 — deviations flag a pipeline or matching bug.

## Related tools

- **scikit-learn** (Provides the NMF implementation used for the NMF-based program-recovery pipeline on pooled simulated count matrices)
- **scsim** (Simulator used to generate synthetic single-cell count matrices with pre-embedded gene expression programs to serve as NMF-recovery ground truth)
- **MetaTiME** (ICA-based meta-component recovery pipeline against which the NMF decomposition is benchmarked) — github.com/yi-zhang/MetaTiME

## Examples

```
from sklearn.decomposition import NMF; W = NMF(n_components=14, init='nndsvda', random_state=0).fit_transform(pooled_counts)
```

## Evaluation signals

- Recovered NMF components match back to the 14 known GEPs embedded by scsim (13 cell-type-specific + 1 signaling program) in each simulation.
- Per-program recovery scores for NMF are computed at all three cohort settings (5, 10, 20 pooled datasets) and are directly comparable to the ICA pipeline's scores.
- Recovery improves monotonically as pooled cohort count increases from 5 to 10 to 20.
- NMF recovery scores are slightly below the ICA pipeline's scores, consistent with the reported benchmark outcome.
- Aggregated comparison table exists as simulation_benchmark_results.csv with ICA vs NMF and cohort-size columns populated.

## Limitations

- NMF slightly underperformed ICA at recovering true gene expression programs in the MetaTiME simulation benchmark, which motivated MetaTiME's choice of ICA.
- The benchmark conclusion rests on scsim-simulated data with pre-embedded signatures; performance on real tumor scRNA-seq cohorts is not established by this simulation.
- MetaTiME's factorization rationale is component independence (ICA), so NMF lacks that independence-optimization property.

## Evidence

- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
