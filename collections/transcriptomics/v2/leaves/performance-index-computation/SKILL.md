---
name: performance-index-computation
description: Use when benchmarking marker-gene detection methods on a labeled expression
  matrix (e.g. GSE28490) and you need a single quantitative quality score (P1) per
  method, compared against a known a priori marker-gene subset.
when_to_use_negative:
- The goal is marker detection itself rather than benchmarking — use the COT detection
  pipeline directly instead of scoring with P1.
- No a priori/reference marker-gene subset exists for the dataset, so there is nothing
  to compute P1 against.
- The expression data are not batch-corrected, since COT's input must be batch-corrected
  before marker lists (and hence P1) are computed.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: COT
  role: produces the marker-gene lists (cot.markers) whose quality is scored by P1;
    installed via pip from GitHub
  repo: https://github.com/MintaYLu/COT
- name: pandas
  role: holds the GSE28490 batch-corrected expression matrix and subtype labels used
    for marker detection and P1 computation
- name: SciPy
  role: computes one-vs-rest t-test marker lists per subtype as the comparison baseline
- name: R
  role: runs limma and edgeR (R/Bioconductor) per subtype to obtain comparison marker-gene
    lists at the equivalent significance threshold
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:5dbd64e0309153a63d4c673baaa9f617a95e35409d4c1647b1aafdc1dca604dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - R
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# performance-index-computation

## Summary

Computes the quantitative performance index P1 (Eq 4 of the COT paper) to score the quality of detected marker-gene lists against an a priori marker-gene reference, enabling method benchmarking such as COT vs OVR t-test/limma/edgeR on the GSE28490 dataset.

## When to use

Use when benchmarking marker-gene detection methods on a labeled expression matrix (e.g. GSE28490) and you need a single quantitative quality score (P1) per method, compared against a known a priori marker-gene subset.

## When NOT to use

- The goal is marker detection itself rather than benchmarking — use the COT detection pipeline directly instead of scoring with P1.
- No a priori/reference marker-gene subset exists for the dataset, so there is nothing to compute P1 against.
- The expression data are not batch-corrected, since COT's input must be batch-corrected before marker lists (and hence P1) are computed.

## Inputs

- batch-corrected gene expression matrix (pandas DataFrame, genes x samples) for GSE28490 downloaded from GEO
- per-sample subtype labels (e.g. subtype_label list, one label per sample)
- a priori marker-gene subset defined for GSE28490 (reference marker list)
- per-method marker-gene lists (COT cot.markers, OVR t-test, limma, edgeR) at the chosen significance threshold

## Outputs

- P1 performance index value (Eq 4) per method: COT, OVR t-test, limma, edgeR, and a priori subset
- relative percent reductions in P1 (e.g. 72.5% and 83.9%)
- per-method marker lists and P1 values written to cot_p1_benchmark.csv

## How to apply

Load the batch-corrected GSE28490 expression matrix into a pandas DataFrame with per-sample subtype labels (input to COT must be batch-corrected). Generate each method's marker lists: run the COT pipeline (generate_subtype_means, generate_cos_values, estimate_p_values, obtain_subtype_markers) and call markers at the q-value significance threshold; compute one-vs-rest t-tests per subtype with scipy.stats; run limma and edgeR in R/Bioconductor at an equivalent significance threshold. For each method's list and for the a priori marker-gene subset, evaluate the P1 performance index per Eq 4 of the paper. Judge success by reproduction of the reported values: COT P1 = 0.019, OVR t-test P1 = 0.069 (72.5% relative reduction), and the comparison set P1 = 0.118 (83.9% relative reduction).

## Related tools

- **COT** (produces the marker-gene lists (cot.markers) whose quality is scored by P1; installed via pip from GitHub) — https://github.com/MintaYLu/COT
- **pandas** (holds the GSE28490 batch-corrected expression matrix and subtype labels used for marker detection and P1 computation)
- **SciPy** (computes one-vs-rest t-test marker lists per subtype as the comparison baseline)
- **R** (runs limma and edgeR (R/Bioconductor) per subtype to obtain comparison marker-gene lists at the equivalent significance threshold)

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.cos_pipeline(subtype_label=subtype_label, top=2)
```

## Evaluation signals

- Reproduced P1 values: COT = 0.019, OVR t-test = 0.069, comparison set = 0.118 on GSE28490.
- Relative reductions match the reported 72.5% (vs OVR t-test) and 83.9% (vs the 0.118 set).
- All per-method markers and P1 values are written to cot_p1_benchmark.csv.
- COT markers are called at the q-value significance threshold from cot.obtain_subtype_markers() output (cot.markers dict per subtype).
- Input matrix was batch-corrected before running COT, per the tool's requirement.

## Limitations

- COT cannot estimate p-values when the gene set is too small ('Cannot calculate p-values due to the limited genes numbers'); NaNs must be removed before estimate_p_values, which blocks downstream marker calls and P1 scoring.
- COT input must be batch-corrected; uncorrected data invalidate the marker lists being scored.
- COT complements rather than replaces existing methods, so P1 benchmarking against OVR t-test/limma/edgeR is comparative, not a substitution.

## Evidence

- [supplementary] Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490.: "Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490."
- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [readme] Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation.: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
- [abstract] Nevertheless, COT complements not replaces existing methods.: "Nevertheless, COT complements not replaces existing methods."
