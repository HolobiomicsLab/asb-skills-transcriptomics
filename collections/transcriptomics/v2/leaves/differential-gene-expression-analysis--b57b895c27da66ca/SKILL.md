---
name: differential-gene-expression-analysis
description: Use when raw scRNA-seq expression matrices for two or more FACS-sorted
  or annotated cell subsets are available and signature genes distinguishing terminal
  cell types are needed for fate prediction or marker discovery.
when_to_use_negative:
- Input has no defined subset/group labels or end-stage contrasts, since Mpath depends
  on signature genes of differentiated cell types for fate prediction
- The goal is de novo clustering or marker discovery in heterogeneous data where specific
  subset information is not known, rather than testing pre-specified subset contrasts
- Temporal ordering of developmental stages is assumed in advance — this method explicitly
  uses genes only from end-stage contrasts without a priori temporal ordering knowledge
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Mpath
  role: Consumes the derived signature genes to predict individual cell fate by travelling
    back in time from the differentiated cDC1 and cDC2 subsets
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment in which the ANOVA, post-hoc Tukey's test, and Benjamini-Hochberg
    adjustment are performed and in which the Mpath R package runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:d51c09b04aa836e827a2c58f1aa7f4f1a39492a874e5f06e2c055bb3ff7ebee3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# differential-gene-expression-analysis

## Summary

Identifies differentially expressed genes between defined cell subsets (e.g., mature cDC1 versus cDC2 in GSE60782) using ANOVA followed by post-hoc Tukey's test, with Benjamini-Hochberg P-value adjustment. These end-stage contrast genes serve as signature genes for downstream trajectory/fate analyses such as Mpath cell-fate prediction.

## When to use

Use when raw scRNA-seq expression matrices for two or more FACS-sorted or annotated cell subsets are available and signature genes distinguishing terminal cell types are needed for fate prediction or marker discovery.

## When NOT to use

- Input has no defined subset/group labels or end-stage contrasts, since Mpath depends on signature genes of differentiated cell types for fate prediction
- The goal is de novo clustering or marker discovery in heterogeneous data where specific subset information is not known, rather than testing pre-specified subset contrasts
- Temporal ordering of developmental stages is assumed in advance — this method explicitly uses genes only from end-stage contrasts without a priori temporal ordering knowledge

## Inputs

- scRNA-seq expression matrix for annotated end-stage cell subsets (e.g., GEO GSE60782 mature cDC1 vs cDC2)
- Vector of sample/cell group labels defining the subset contrasts

## Outputs

- List of differentially expressed genes per contrast, with ANOVA/Tukey statistics and Benjamini-Hochberg-adjusted P values
- Signature gene sets for differentiated cell types (cDC1- and cDC2-specific genes) usable for trajectory fate prediction

## How to apply

Load the per-cell or per-subset expression matrix (e.g., mature cDC1 and cDC2 subsets from GSE60782) into R. Run a gene-wise ANOVA across the end-stage subset contrasts, then apply post-hoc Tukey's test to identify genes differentially expressed between specific subset pairs. Adjust P values for multi-test correction using Benjamini-Hochberg to control the false discovery rate. The resulting DEG lists (e.g., Supplementary Data 1–3 style outputs: DEGs between preDC_a/preDC_c and along CDP-to-cDC1/cDC2 paths) are then used as signature genes for Mpath's fate-prediction step, which travels back in time from differentiated subsets without a priori temporal ordering knowledge.

## Related tools

- **Mpath** (Consumes the derived signature genes to predict individual cell fate by travelling back in time from the differentiated cDC1 and cDC2 subsets) — JinmiaoChenLab/Mpath
- **R** (Runtime environment in which the ANOVA, post-hoc Tukey's test, and Benjamini-Hochberg adjustment are performed and in which the Mpath R package runs)

## Examples

```
fit <- aov(t(expr) ~ group); tukey <- TukeyHSD(fit); padj <- p.adjust(pv, method='BH')
```

## Evaluation signals

- Every gene's reported P value carries a corresponding Benjamini-Hochberg-adjusted value (multi-test correction applied)
- Per-gene contrast was tested with ANOVA followed by post-hoc Tukey's test rather than a plain pairwise test
- Derived signature genes recapitulate expected subset biology (known cDC1/cDC2 subset-specific genes recovered; Mpath identified early transcriptional markers comprising known subset-specific genes and putative novel regulators)
- DEG counts and identity are reproducible against the article's supplementary data tables (e.g., Supplementary Data 1–3 DEG lists)
- When fed into Mpath fate prediction, the signature genes yield the expected result: branches at the preDC stage exclusively committed to cDC1 or cDC2

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, so DEGs from poorly sorted or impure subsets propagate errors downstream
- Genes are selected only from end-stage contrasts; early transient regulators may be missed without additional ordering information
- Demonstrated utility was limited to two data sets (mouse bone marrow-derived DC progenitors and primary human myoblasts), so generalization to other tissues is untested

## Evidence

- [supplementary] DEG identification using ANOVA followed by post-hoc Tukey's test: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [supplementary] Benjamini-Hochberg multiple-testing correction: "P values were adjusted for multi-test correction using Benjamini-Hochberg."
- [discussion] Signature genes from end-stage contrasts for fate prediction: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [readme] Mpath R package availability: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
- [discussion] Fate prediction from differentiated subsets: "The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors."
