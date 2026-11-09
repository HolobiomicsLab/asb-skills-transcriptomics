---
name: cell-type-identification-performance-metrics
description: Use when comparing or tuning a scRNA-seq cell type identifier on annotated
  datasets and you need to quantify correct/erroneous assignments for known and unknown
  cell populations (C, E, EA, CUA, EUA).
when_to_use_negative:
- Datasets lack cell type annotations, so cross-evaluation ground truth and rejection
  thresholds cannot be derived from reference data
- Only one annotated dataset is available, making circular train/test role shifting
  infeasible
- The goal is hierarchical subtype annotation with a fixed marker DB (HiCAT's marker-based
  mode), not identifier performance benchmarking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Reference-based identifier (MarkerCount_Ref) being evaluated; computes binarized
    marker counts, rejection thresholds, and cluster-basis reassignment with PCA (15
    components) and GMM
  repo: github.com/combio-dku/MarkerCount
- name: HiCAT
  role: Improved marker-based successor of MarkerCount, mentioned as the merged version;
    not the evaluation target in reference-based mode
  repo: github.com/combio-dku/HiCAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:496f5ce7bb1cc7c1855b31dd418898dfdd65f9968c26ed8a38e010c2939af40d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  tools:
  - HiCAT
  - MarkerCount
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# cell-type-identification-performance-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates single-cell RNA-seq cell type identifiers using the MarkerCount paper's five-part performance framework (C, E, EA, CUA, EUA) over cross-evaluated reference datasets, including hyper-parameter sweeps for marker count N and reassignment thresholds.

## When to use

Use when comparing or tuning a scRNA-seq cell type identifier on annotated datasets and you need to quantify correct/erroneous assignments for known and unknown cell populations (C, E, EA, CUA, EUA).

## When NOT to use

- Datasets lack cell type annotations, so cross-evaluation ground truth and rejection thresholds cannot be derived from reference data
- Only one annotated dataset is available, making circular train/test role shifting infeasible
- The goal is hierarchical subtype annotation with a fixed marker DB (HiCAT's marker-based mode), not identifier performance benchmarking

## Inputs

- Annotated single-cell RNA-seq count matrices (e.g. GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322)
- Candidate hyper-parameter values to sweep (e.g. N markers per cell type, unassigned-portion reassignment threshold)
- Reference-mode identifier configuration (e.g. MarkerCount_Ref with fth=0.9, weight exponent n=2)

## Outputs

- Performance-vs-N sweep table (marker_count_sweep.csv) with C, E, EA, CUA, EUA per dataset and N
- Selected hyper-parameter setting (e.g. N=18 markers per cell type) with per-dataset justification
- Per-role cross-evaluation predictions and reassignment outcomes

## How to apply

Run cross-evaluation over annotated scRNA-seq datasets (e.g. GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322): take one dataset as test and the rest as reference, circularly shifting roles, obtaining rejection thresholds from the reference data per cell type. For the identifier being tuned, sweep the marker-count hyper-parameter N (MarkerCount computes marker scores s_m,j with occurrence-frequency threshold fth=0.9 and relative-weight exponent n=2, selecting the top-N genes per cell type) and record the five measures: C (correctly assigned), E (erroneously assigned), EA (erroneously assigned among assigned), CUA (correctly unassigned), and EUA (erroneously unassigned). Apply MarkerCount's cluster-basis reassignment before scoring: PCA with 15 components, GMM clustering, and reassignment when the portion of unassigned cells falls below 0.8 (best in the 0.1–0.9 range). Assemble the sweep results (e.g. marker_count_sweep.csv) and pick N maximizing performance across datasets — in the source paper N=18 was fixed for all cell types because it performed best on most datasets. Note the known tradeoff: conservative rejection thresholds lower erroneous assignment but raise erroneously unassigned cells, so report CUA and EUA alongside C and E.

## Related tools

- **MarkerCount** (Reference-based identifier (MarkerCount_Ref) being evaluated; computes binarized marker counts, rejection thresholds, and cluster-basis reassignment with PCA (15 components) and GMM) — github.com/combio-dku/MarkerCount
- **HiCAT** (Improved marker-based successor of MarkerCount, mentioned as the merged version; not the evaluation target in reference-based mode) — github.com/combio-dku/HiCAT

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = mkrcnt$MarkerCount_Ref(.. arguments ..)  # or in Python: df_res = MarkerCount_Ref(test_data, ref_data, ...)
```

## Evaluation signals

- The performance-vs-N sweep shows N=18 yielding the best performance for most of the eight datasets, matching the reported fixed hyper-parameter
- All five measures (C, E, EA, CUA, EUA) are reported per test dataset, with the C/E versus CUA/EUA tradeoff visible when the rejection threshold is varied
- Reassignment uses exactly PCA with 15 components, GMM clustering, and a 0.8 unassigned-portion threshold before scoring
- Cross-evaluation covers every dataset in the test role exactly once via circular role shifting
- Marker selection uses occurrence-frequency threshold fth=0.9 and penalty-weight exponent n=2 before taking the top-N genes per cell type

## Limitations

- Existing rejection functions involve a tradeoff between erroneously unassigned and erroneously assigned measures; no single threshold optimizes both
- For cell types such as beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, and T cells in lung, identification via normalized marker counts alone yields considerable errors even with reassignment
- Proper identification requires well-clustered cell populations, which is not always the case; handling not-clearly-separable sub-populations (monocytes, macrophage, dendritic cells) requires further study

## Evidence

- [methods] N=18 markers per cell type showed the best performance for most datasets: "The integer N is also a hyper-parameter, which we fixed to 18 for all cell types. The number 18 was chosen, as it showed the best performance for most datasets used in this work."
- [methods] Reassignment threshold 0.8 chosen from 0.1-0.9 sweep: "Reassignment is performed if the portion of unassigned cells is below a certain threshold, say 0.8. This value was chosen as it exhibited the best performance in the range of 0.1–0.9 at 0.1 steps."
- [methods] Occurrence frequency threshold fth=0.9: "for some occurrence frequency threshold fth, which we opted to set at 0.9 as a heuristic choice."
- [intro] Tradeoff between erroneous assignment and unassignment: [evidence span withheld — anchor status: not_found]
- [readme] Tool installation and entry points for reference-based mode: "One can import the function by adding a line in your script, i.e., `from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount`"
