---
name: percentile-based-maximum-estimation
description: Use when preprocessing a gene-expression matrix (rows = genes, columns
  = cell populations) for ICELLNET ligand-receptor scoring and highly expressed molecules
  could otherwise dominate the communication score.
when_to_use_negative:
- Input is already normalized/scaled for intercellular communication scoring (values
  already bounded, e.g., on a 0-10 reference scale)
- The downstream method is not ICELLNET-family communication scoring — the mean-of-top-5%
  maximum and 10-cap are calibrated for ICELLNET's communication-score quantification
- Non-human expression matrices, since the ICELLNET ligand-receptor database is curated
  exclusively from human studies
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Framework whose communication-score computation consumes the scaled expression
    matrix; provides the scRNAseq tutorial data layout this scaling follows
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Implementation environment for computing per-gene percentile-based maxima
    and writing scaled_expression.csv
  repo: github.com/soumelis-lab/ICELLNET
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:49c9996bfd6023ef83e04542419ac3f7848ea0d1bba98a67997db28f82e00852
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# percentile-based-maximum-estimation

## Summary

Estimates each ligand/receptor gene's maximum expression as the mean of its 5% highest values, then rescales expression so the reference maximum maps to 10 (values capped at 10). This is ICELLNET's gene-scaling scheme that keeps highly expressed molecules (e.g., chemokines) from dominating communication scores.

## When to use

Use when preprocessing a gene-expression matrix (rows = genes, columns = cell populations) for ICELLNET ligand-receptor scoring and highly expressed molecules could otherwise dominate the communication score.

## When NOT to use

- Input is already normalized/scaled for intercellular communication scoring (values already bounded, e.g., on a 0-10 reference scale)
- The downstream method is not ICELLNET-family communication scoring — the mean-of-top-5% maximum and 10-cap are calibrated for ICELLNET's communication-score quantification
- Non-human expression matrices, since the ICELLNET ligand-receptor database is curated exclusively from human studies

## Inputs

- Gene expression matrix (rows = ligand/receptor genes, columns = cell populations) from bulk RNA-seq, scRNA-seq, microarray, or spatial transcriptomics
- List of ligand/receptor genes to scale (e.g., genes in the ICELLNET curated ligand-receptor interaction database)

## Outputs

- scaled_expression.csv: expression matrix with each gene's values divided by its percentile-based maximum (mean of the 5% highest values), multiplied by 10, and capped at 10

## How to apply

Load the gene expression matrix in R using the ICELLNET scRNAseq tutorial data layout (rows = genes, columns = cell populations). For each ligand/receptor gene, compute the maximum as the mean of the 5% highest expression values for that gene — this percentile-based maximum is robust to single outlier cells or samples. Divide each gene's expression values by this computed maximum and multiply by 10 so the reference gene reaches a scaled value of 10. Cap any outlier values exceeding 10 at 10 to bound the scaled scores. The rationale, per the ICELLNET authors, is that scaling by this estimated maximum prevents the communication score from being dominated by a small number of highly expressed molecules (e.g., chemokines) and enables detection of important low-level molecules such as IL-12 and IL-4.

## Related tools

- **ICELLNET** (Framework whose communication-score computation consumes the scaled expression matrix; provides the scRNAseq tutorial data layout this scaling follows) — github.com/soumelis-lab/ICELLNET
- **R** (Implementation environment for computing per-gene percentile-based maxima and writing scaled_expression.csv) — github.com/soumelis-lab/ICELLNET

## Examples

```
max_ref <- apply(mat,1,function(x) mean(head(sort(x,decreasing=TRUE),ceiling(0.05*length(x))))); scaled <- pmin(t(t(mat)/max_ref)*10,10)
```

## Evaluation signals

- All values in scaled_expression.csv are non-negative
- No scaled value exceeds the cap of 10
- For each gene, the reference maximum (mean of its 5% highest values) maps to a scaled value of 10
- Low-expression but biologically important molecules (e.g., IL-12, IL-4) retain detectable scaled values rather than being crushed by highly expressed genes such as chemokines
- Scaled matrix dimensions match the input matrix (rows = genes, columns = cell populations)

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x, so the scaling code must target the matching version
- The ICELLNET database includes a relatively small number of interactions compared to other databases, and chemokine and checkpoint interactions were not yet included at publication time, constraining which genes benefit from this scaling
- Mean-of-top-5% maximum estimation assumes enough values per gene to estimate the tail; very sparse scRNA-seq genes may give unstable maxima

## Evidence

- [other] percentile maximum from source card: [evidence span withheld — anchor status: not_found]
- [other] scaling rationale from source card: [evidence span withheld — anchor status: not_found]
- [readme] applicability across transcriptomic technologies: "Versatile tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics)"
- [abstract] communication score quantification: "2) quantification of communication scores;"
- [methods] scRNAseq tutorial reference: "ICELLNET analysis on scRNAseq data: [Tutorial](https://github.com/soumelis-lab/ICELLNET/blob/master/exemples_tutorials/Exemple2_scRNAseq.md)"
