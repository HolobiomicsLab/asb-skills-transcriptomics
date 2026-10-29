---
name: cross-method-pathway-rank-comparison
description: Use when a labeled RNA-seq expression matrix must be benchmarked across
  sample-wise pathway quantification methods to rank a target pathway (e.g., TNFa
  signaling in IL17+TNFa-treated vs control samples) by significance.
when_to_use_negative:
- The expression data are raw counts rather than normalized values — rRoma explicitly
  requires normalized data (edgeR cpm, DESeq2 vst, or equivalent).
- The analysis needs a complete biological systems view with multiple gene-set databases
  combined, not a Hallmark-only explanatory benchmark.
- The dataset contains missing values that have not been imputed — the rROMA README
  recommends using data whose missing values are already imputed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: rROMA
  role: Runs the PC1-based module-activity analysis producing the ModuleMatrix with
    L1/Median Exp p-values against the null distribution
  repo: github.com/sysbio-curie/rROMA
- name: GSVA
  role: Computes sample-wise pathway scores on the identical normalized matrix using
    the GSVA, PLAGE, ssGSEA and z-score methods for the benchmark comparison
- name: edgeR
  role: Provides the cpm normalization applied to the GSE176121 count matrix before
    rRoma and GSVA scoring
- name: R
  role: Environment in which rRoma.R and the GSVA-based comparison scripts (e.g. Rehman_TNF_IL17_other_methods.R)
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:0811d4640a1eaf52793c3c7fe3bf1878fd4a81de280fe77e52bff42017bc3229
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSVA
  - R
  - edgeR
  - rROMA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-method-pathway-rank-comparison

## Summary

Benchmarks rROMA's module-activity quantification against sample-wise pathway scoring methods (GSVA, PLAGE, ssGSEA, z-score) on the GSE176121 cystic-fibrosis airway-epithelial RNA-seq dataset, to show rROMA most consistently identifies expected TNFα signaling activation.

## When to use

Use when a labeled RNA-seq expression matrix must be benchmarked across sample-wise pathway quantification methods to rank a target pathway (e.g., TNFa signaling in IL17+TNFa-treated vs control samples) by significance.

## When NOT to use

- The expression data are raw counts rather than normalized values — rRoma explicitly requires normalized data (edgeR cpm, DESeq2 vst, or equivalent).
- The analysis needs a complete biological systems view with multiple gene-set databases combined, not a Hallmark-only explanatory benchmark.
- The dataset contains missing values that have not been imputed — the rROMA README recommends using data whose missing values are already imputed.

## Inputs

- GSE176121 RNA-seq expression matrix normalized as edgeR cpm (genes as rows, samples as columns; e.g. Rehman_CF_NCF_RNA_exp_matrix_norm.rda and Rehman_IL17_TNFa_RNA_exp_matrix_norm.rda)
- Sample labels defining group membership (e.g. Rehman_CF_NCF_sample_labels.rda, Rehman_IL17_TNFa_sample_labels.rda)
- GMT pathway database: h.all.v2023.1.Hs.symbols.gmt or the 50 HALLMARK gene sets retrieved from MSigDB v6.2 via SelectFromMSIGdb('HALLMARK')

## Outputs

- Benchmark comparison table of dysregulated pathway counts per method for scenario 1 (GSVA/ssGSEA/z-score: 0; PLAGE: 45/50; rROMA: 3)
- Per-method ranking of the TNFa signaling pathway among Hallmark sets for scenario 2 (GSVA 5/24, PLAGE 5/27, ssGSEA 7/17, z-score 18/45; rROMA top-ranked)
- rRoma ModuleMatrix with L1 and Median Exp p/q values and shifted/over-dispersed module calls at 'ppv Median Exp' <= 0.05

## How to apply

Normalize the raw GSE176121 counts as edgeR cpm (rRoma requires normalized data; genes as rows, samples as columns) and split into two scenarios: CF vs non-CF (scenario 1) and IL17+TNFa treated vs control (scenario 2). Retrieve the 50 MSigDB Hallmark sets with SelectFromMSIGdb('HALLMARK') (MSigDB v6.2), stripping the 'HALLMARK_' prefix, then run rRoma.R with a fixed seed and the recommended settings (PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, GeneOutDetection='L1OutExpOut', nSamples>=100) to obtain the ModuleMatrix with L1 and Median Exp p/q values; call a module shifted at 'ppv Median Exp' <= 0.05. In parallel, compute sample-wise scores for the same 50 gene sets with the GSVA package under its four methods on the identical matrix. For scenario 1, test differential pathway activity per method and count dysregulated pathways; for scenario 2, rank the TNFa signaling pathway among all tested Hallmark sets by each method's significance. The rationale is that rROMA's PC1-based shift/overdispersion testing avoids the loss of contrast seen with sample-wise scores, giving the most consistent result with expected biology.

## Related tools

- **rROMA** (Runs the PC1-based module-activity analysis producing the ModuleMatrix with L1/Median Exp p-values against the null distribution) — github.com/sysbio-curie/rROMA
- **GSVA** (Computes sample-wise pathway scores on the identical normalized matrix using the GSVA, PLAGE, ssGSEA and z-score methods for the benchmark comparison)
- **edgeR** (Provides the cpm normalization applied to the GSE176121 count matrix before rRoma and GSVA scoring)
- **R** (Environment in which rRoma.R and the GSVA-based comparison scripts (e.g. Rehman_TNF_IL17_other_methods.R) are executed)

## Examples

```
AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Scenario 1 benchmark invariant: 0 dysregulated pathways for GSVA, ssGSEA and z-score, 45/50 for PLAGE, and exactly 3 shifted Hallmark modules for rROMA (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION).
- Scenario 2 benchmark invariant: TNFa signaling ranked 5/24 by GSVA, 5/27 by PLAGE, 7/17 by ssGSEA and 18/45 by z-score, while rROMA places it top among significantly shifted modules.
- rRoma.R must be run under a fixed random seed; re-running with the same seed should reproduce the same ModuleMatrix p/q values.
- Module shifted/over-dispersed calls must satisfy 'ppv Median Exp' <= 0.05.
- The input matrix must be gene-by-sample with row names as gene names and column names as sample IDs, matching the sample labels table.

## Limitations

- The 50 HALLMARK gene sets were chosen for explanatory purposes; the authors recommend combining multiple databases (e.g. KEGG, MSigDB) for a more complete biological view.
- rRoma.R relies on random permutations, so results depend on the chosen random seed (fixed at # set.seed(69) in the vignette).
- Default hyperparameters (e.g. extreme-weight percentage, outlier z-score threshold, ApproxSample) are not reported in the article, complicating exact reproduction.
- The CF case study lacks statistical power; authors state additional transcriptional datasets would be needed to increase it.

## Evidence

- [methods] This is a RNA sequencing dataset of human airway epithelial cultures from cystic fibrosis (CF) and non-CF donors from GSE176121, PMID: 34166230.: "This is a RNA sequencing dataset of human airway epithelial cultures from cystic fibrosis (CF) and non-CF donors from GSE176121, PMID: 34166230."
- [readme] Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [readme] Here, we extract all the "HALLMARK" genesets from MSigDB (Liberzon A et al, Cell Syst 2015). We also remove "HALLMARK_" from the names to simplify graphical representations.: "Here, we extract all the "HALLMARK" genesets from MSigDB (Liberzon A et al, Cell Syst 2015). We also remove "HALLMARK_" from the names to simplify graphical representations."
- [readme] Repository contaning data and codes to reproduce the comparison of sample-wise methods in the rRoma article.: [evidence span withheld — anchor status: external_doc]
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
