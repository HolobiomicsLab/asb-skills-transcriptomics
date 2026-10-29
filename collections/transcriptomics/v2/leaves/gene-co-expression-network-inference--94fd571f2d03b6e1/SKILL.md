---
name: gene-co-expression-network-inference
description: Use when you have a raw UMI count matrix from a defined cell population
  (e.g. dysfunctional CD8+ T cells from MARS-seq melanoma, GSE123139) and want gene-level
  co-expression edges that recapitulate known functional annotations.
when_to_use_negative:
- When the goal is causal regulatory network reconstruction (v-structures, regularized
  regression) or Granger causality with longitudinal data — Normalisr explicitly does
  not do this
- When testing arbitrary nonlinear associations between genes — Normalisr is a linear-model
  association framework
- When input is already normalized or imputed full-length (non-UMI) scRNA-seq — nonlinear
  covariates were validated only on UMI-based datasets and may need revalidation on
  full-length platforms
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Normalisr
  role: Normalization (lcpm, scaling_factor, normcov, compute_var, normvar), QC (qc_reads,
    qc_outlier), co-expression Q-value computation (coex), and GO program covariate
    removal (gotop)
  repo: https://github.com/lingfeiwang/normalisr
- name: GOATOOLS
  role: Testing over-abundance of GO pathway gene pairs among co-expression network
    edges with Bonferroni correction
- name: BioServices
  role: Fetching KEGG pathway annotations for edge-enrichment testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:d7bfce878e4d91cb9156f6f314358a02458f958b71a683f4d80df95ef24a735b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-co-expression-network-inference

## Summary

Infer single-cell gene co-expression networks from UMI scRNA-seq by first removing nonlinear library-size confounders with Normalisr normalization (lcpm, normcov, compute_var, normvar) and then computing all pairwise gene co-expression Q-values with linear-model association testing, retaining significant edges at a stringent FDR cutoff.

## When to use

Use when you have a raw UMI count matrix from a defined cell population (e.g. dysfunctional CD8+ T cells from MARS-seq melanoma, GSE123139) and want gene-level co-expression edges that recapitulate known functional annotations.

## When NOT to use

- When the goal is causal regulatory network reconstruction (v-structures, regularized regression) or Granger causality with longitudinal data — Normalisr explicitly does not do this
- When testing arbitrary nonlinear associations between genes — Normalisr is a linear-model association framework
- When input is already normalized or imputed full-length (non-UMI) scRNA-seq — nonlinear covariates were validated only on UMI-based datasets and may need revalidation on full-length platforms

## Inputs

- Raw UMI count matrix from scRNA-seq (tsv or sparse Cell Ranger mtx format, e.g. GSE123139 MARS-seq human melanoma)
- Cell and gene name lists (one per line, text format)
- GO and KEGG pathway annotations (via GOATOOLS and BioServices) for post-hoc enrichment validation

## Outputs

- Pairwise gene co-expression Q-value matrix from Normalisr's coex function
- Filtered co-expression network of significant edges (Q ≤ 1e-15)
- Major connected component of the co-expression network (major_component_coex_network.tsv)
- GO/KEGG pathway enrichment results over network edges (pathway_enrichment.tsv)

## How to apply

Load the raw UMI matrix (e.g. GSE123139 from GEO) and apply QC thresholds: restrict to cells with ≥500 reads and ≥100 detected genes, and genes expressed in ≥500 cells (MARS-seq; ≥50 cells and ≥2% of cells for 10x data). Normalize raw counts with Normalisr's pipeline (lcpm, scaling_factor, normcov, compute_var, normvar) to remove nonlinear library-size effects at both mean and variance levels, removing outlier cells with qc_outlier using a prior outlier proportion bound of 2% and Bonferroni P ≤ 1e-10. Compute all pairwise gene co-expression Q-values with Normalisr's coex function and retain edges at FDR Q ≤ 1e-15 — this strong cutoff prioritizes more direct gene interactions. Optionally iteratively remove top principal components of dominant GO programs (e.g. 'cytosolic part' and 'chromosome condensation') as covariates using the gotop function, recomputing co-expression at each iteration, then extract the major connected component. Validate by testing over-abundance of GO/KEGG pathway gene pairs among edges (GOATOOLS, BioServices, Bonferroni P ≤ 0.05): significant enrichment indicates the network recovered cell-type-specific and generic functional similarities.

## Related tools

- **Normalisr** (Normalization (lcpm, scaling_factor, normcov, compute_var, normvar), QC (qc_reads, qc_outlier), co-expression Q-value computation (coex), and GO program covariate removal (gotop)) — https://github.com/lingfeiwang/normalisr
- **GOATOOLS** (Testing over-abundance of GO pathway gene pairs among co-expression network edges with Bonferroni correction)
- **BioServices** (Fetching KEGG pathway annotations for edge-enrichment testing)

## Examples

```
import normalisr.normalisr as norm; ?norm.de  # python functional interface; CLI: normalisr coex (submodule help via normalisr -h)
```

## Evaluation signals

- All retained edges have co-expression FDR Q ≤ 1e-15; recomputing the coex run with the same QC and cutoff should reproduce the edge set (non-parametric, so no adjustable parameters besides QC and final cutoffs)
- Outlier cells removed only with prior outlier proportion bound 2% and Bonferroni P ≤ 1e-10; number of removed cells should be within the 2% bound
- GO/KEGG pathway enrichment on the final edge set recovers significant over-abundance (Bonferroni P ≤ 0.05) — the paper found 33 GO and 8 KEGG pathways significant, indicating the network recapitulates known gene functions
- Input format check: matrices are tsv (or gzipped tsv, auto-recognized) or sparse mtx; row/column name files one per line
- Iterative gotop removal of 'cytosolic part' and 'chromosome condensation' principal components should change the edge set at each iteration; the final network's major connected component is extracted after convergence

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce different read count distributions and require revalidating the nonlinear cellular summary covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Normalisr does not reconstruct causal regulatory networks via v-structures or regularized regression, nor Granger causality with longitudinal data
- Batch effects in multi-dataset integrative studies are not addressed and may require additional consideration
- The strong Q ≤ 1e-15 cutoff prioritizes direct interactions but trades off sensitivity; the GO pathway removal process is iterative and depends on the co-expression Q-value cutoff

## Evidence

- [abstract] Normalization and co-expression computation: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [abstract] Co-expression recapitulates known functions: "On conventional scRNA-seq, Normalisr recovers gene-level co-expression networks that recapitulated known gene functions."
- [discussion] Cell QC threshold: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [discussion] Gene QC threshold by platform: "genes with non-zero reads both in at least 50 cells and in at least 2% of all cells for 10× technologies, or in at least 500 cells"
- [discussion] Outlier removal parameters: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [discussion] Co-expression cutoff rationale: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [discussion] Iterative GO removal: "The GO pathway removal took an iterative process given the significance cutoff for co-expression Q-value"
- [discussion] Enrichment tooling: "We used GOATOOLS63 and BioServices64 for GO and KEGG pathways, respectively"
- [methods] Normalization function suite: "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [readme] Input formats accepted: [evidence span withheld — anchor status: external_doc]
- [readme] Non-parametric, few parameters: [evidence span withheld — anchor status: external_doc]
- [other] Purpose of strong cutoff: [evidence span withheld — anchor status: not_found]
