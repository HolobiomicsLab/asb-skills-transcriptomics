---
name: differential-pathway-activity-testing-between-condition-groups
description: Use when a normalized RNA-seq expression matrix with two condition groups
  (e.g., CF vs non-CF, or treated vs control) needs a benchmarked differential pathway
  activity comparison across rROMA and sample-wise scoring methods.
when_to_use_negative:
- The expression matrix is raw counts or otherwise not normalized — rRoma explicitly
  requires normalized data (e.g., cpm or vst) and will not give valid module statistics
  on raw input
- The analysis goal is single-method pathway scoring rather than a cross-method benchmark
  against expected biology
- Gene sets outside MSigDB Hallmark-scale references are required without a GMT file
  — the workflow depends on a GMT module list (SelectFromMSIGdb or ReadGMTFile)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: rROMA
  role: 'Primary method: computes module activity via PC1 with shift/overdispersion
    statistics on the Hallmark module list'
  repo: https://github.com/sysbio-curie/rROMA
- name: GSVA
  role: Comparison method used to compute sample-wise pathway scores by four algorithms
    (GSVA, PLAGE, ssGSEA, z-score) on the same normalized matrix
- name: edgeR
  role: Provides cpm normalization of the RNA-seq count matrix before pathway activity
    quantification
- name: R
  role: Implementation environment; rRoma is an R package run via rRoma.R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:2c54cf833ccf447a2d6822ba10b55d9a2cd810e4f53402ae70e0b7aec77481c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - GSVA
  - R
  - edgeR
  - rROMA
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# differential-pathway-activity-testing-between-condition-groups

## Summary

Benchmark workflow that tests differential pathway activity between condition groups by comparing rROMA module statistics against sample-wise quantification methods (GSVA, PLAGE, ssGSEA, z-score) on the GSE176121 cystic fibrosis airway epithelial dataset, checking whether each method recovers expected biology (TNFα signaling as top shifted pathway in the IL17+TNFα scenario).

## When to use

Use when a normalized RNA-seq expression matrix with two condition groups (e.g., CF vs non-CF, or treated vs control) needs a benchmarked differential pathway activity comparison across rROMA and sample-wise scoring methods.

## When NOT to use

- The expression matrix is raw counts or otherwise not normalized — rRoma explicitly requires normalized data (e.g., cpm or vst) and will not give valid module statistics on raw input
- The analysis goal is single-method pathway scoring rather than a cross-method benchmark against expected biology
- Gene sets outside MSigDB Hallmark-scale references are required without a GMT file — the workflow depends on a GMT module list (SelectFromMSIGdb or ReadGMTFile)

## Inputs

- Normalized gene expression matrix (edgeR cpm), genes as rows, samples as columns, from GSE176121 (e.g., Rehman_CF_NCF_RNA_exp_matrix_norm.rda, Rehman_IL17_TNFa_RNA_exp_matrix_norm.rda)
- Sample labels defining group membership (e.g., Rehman_CF_NCF_sample_labels.rda, Rehman_IL17_TNFa_sample_labels.rda)
- GMT module file of MSigDB Hallmark gene sets (h.all.v2023.1.Hs.symbols.gmt or Msigdb.v6.2 via SelectFromMSIGdb)

## Outputs

- rRoma ModuleMatrix with L1 and Median Exp p/q values per module and sample scores
- Sample-wise pathway score matrices from GSVA, PLAGE, ssGSEA and z-score
- Comparison table: dysregulated pathway counts per method (scenario 1) and TNFα pathway rank per method (scenario 2)

## How to apply

Normalize the expression matrix as edgeR cpm (genes as rows, samples as columns), load sample labels defining the scenario groups, and retrieve the 50 MSigDB Hallmark gene sets with SelectFromMSIGdb('HALLMARK') (MSigDB v6.2), stripping the 'HALLMARK_' prefix. Run rRoma.R with a fixed random seed and recommended settings (PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, GeneOutDetection='L1OutExpOut', nSamples>=100), calling modules shifted at 'ppv Median Exp' <= 0.05. In parallel, compute sample-wise scores for the same gene sets with the GSVA package using the four methods (GSVA, PLAGE, ssGSEA, z-score) on the identical normalized matrix. For each method, test differential activity between groups per scenario and assemble the comparison: in scenario 1 (CF vs non-CF), count dysregulated pathways; in scenario 2 (IL17+TNFα vs control), rank TNFα signaling by each method's significance/score. The rationale is that a method should reproduce the expected biology — rROMA flags TNFA_SIGNALING_VIA_NFKB as the top significantly shifted pathway while GSVA/ssGSEA/z-score find 0 dysregulated pathways and PLAGE reports an implausible 45/50 in scenario 1.

## Related tools

- **rROMA** (Primary method: computes module activity via PC1 with shift/overdispersion statistics on the Hallmark module list) — https://github.com/sysbio-curie/rROMA
- **GSVA** (Comparison method used to compute sample-wise pathway scores by four algorithms (GSVA, PLAGE, ssGSEA, z-score) on the same normalized matrix)
- **edgeR** (Provides cpm normalization of the RNA-seq count matrix before pathway activity quantification)
- **R** (Implementation environment; rRoma is an R package run via rRoma.R)

## Examples

```
library(rRoma); AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Scenario 1 (CF vs non-CF) expected counts: GSVA/ssGSEA/z-score = 0 dysregulated pathways; PLAGE = 45/50; rROMA = 3 (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION)
- Scenario 2 (IL17+TNFα vs control): TNFα signaling ranked 5/24 by GSVA, 5/27 by PLAGE, 7/17 by ssGSEA, 18/45 by z-score, versus top rank for rROMA
- rRoma ModuleMatrix contains L1 and Median Exp p/q value columns; shifted modules are called at 'ppv Median Exp' <= 0.05
- All five methods must be run on the identical normalized matrix and identical 50 gene sets for the benchmark to be fair
- The rRoma run uses a fixed random seed so results are reproducible

## Limitations

- rRoma requires normalized input data; running on raw counts is invalid
- Missing values are imputed with mice via low-rank PCA approximation, but the authors recommend using datasets with missing values already imputed
- The CF case study was stated by the authors to lack statistical power and would require additional transcriptional datasets and testing of several gene set reference databases
- Hallmark was chosen for explanatory purposes; for a complete view of biological systems the authors recommend using rRoma with multiple databases combined

## Evidence

- [readme] rRoma works with normalized data: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [readme] SelectFromMSIGdb usage: "AllHall <- SelectFromMSIGdb("HALLMARK")"
- [methods] Run rRoma with fixed seed: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
- [discussion] Three significantly active Hallmark pathways in CF: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] Dataset provenance: "This is a RNA sequencing dataset of human airway epithelial cultures from cystic fibrosis (CF) and non-CF donors from GSE176121, PMID: 34166230."
- [discussion] p-value computation against null distribution: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value"
