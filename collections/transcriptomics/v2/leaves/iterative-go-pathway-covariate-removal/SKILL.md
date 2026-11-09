---
name: iterative-go-pathway-covariate-removal
description: Use when a Normalisr co-expression network from UMI scRNA-seq (e.g. dysfunctional
  CD8+ T cells from MARS-seq GSE123139) is dominated by generic GO programs and you
  want to regress them out before pathway over-abundance testing.
when_to_use_negative:
- Input is a full-length (non-UMI) scRNA-seq platform, where the nonlinear cellular
  summary covariates and normalization have not been validated
- The goal is causal regulatory network reconstruction (v-structures, regularized
  regression, Granger causality) rather than functional co-expression association
- An integrative multi-dataset analysis where batch effects between datasets require
  additional handling Normalisr does not provide
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the coex, gotop, qc_outlier and normalization functions (lcpm, scaling_factor,
    normcov, compute_var, normvar) used to compute, prune, and iteratively recompute
    the co-expression network with GO-program covariates
  repo: https://github.com/lingfeiwang/normalisr
- name: GOATOOLS
  role: Used to retrieve GO pathway annotations and test for over-abundance of pathway
    gene pairs among co-expression network edges
- name: BioServices
  role: Used to retrieve KEGG pathway annotations for the edge over-abundance test
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:bec6531c7fe47040c497034c879c972d588f0b51d5d92de33d0b128c8a3f7a56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# iterative-go-pathway-covariate-removal

## Summary

Iteratively removes top principal components of dominant GO programs (e.g. 'cytosolic part' and 'chromosome condensation') as covariates between rounds of Normalisr co-expression testing, so that the final single-cell co-expression network reflects cell-type-specific rather than generic functional structure.

## When to use

Use when a Normalisr co-expression network from UMI scRNA-seq (e.g. dysfunctional CD8+ T cells from MARS-seq GSE123139) is dominated by generic GO programs and you want to regress them out before pathway over-abundance testing.

## When NOT to use

- Input is a full-length (non-UMI) scRNA-seq platform, where the nonlinear cellular summary covariates and normalization have not been validated
- The goal is causal regulatory network reconstruction (v-structures, regularized regression, Granger causality) rather than functional co-expression association
- An integrative multi-dataset analysis where batch effects between datasets require additional handling Normalisr does not provide

## Inputs

- Raw UMI count matrix from MARS-seq human melanoma dataset GSE123139 (GEO), restricted to dysfunctional CD8+ T cells
- QC-filtered cell/gene sets (cells with ≥500 reads and ≥100 detected genes; genes expressed in ≥500 cells)
- Normalized, variance-adjusted expression matrix from Normalisr functions (lcpm, scaling_factor, normcov, compute_var, normvar)
- GO and KEGG pathway annotations retrieved via GOATOOLS and BioServices

## Outputs

- Iteratively refined gene co-expression network of dysfunctional CD8+ T cells at FDR Q ≤ 1e-15
- Major connected component of the final co-expression network (major_component_coex_network.tsv)
- GO/KEGG pathway over-abundance results with Bonferroni-corrected P values (pathway_enrichment.tsv)

## How to apply

After Normalisr normalization (lcpm, scaling_factor, normcov, compute_var, normvar) and outlier cell removal (qc_outlier with a 2% prior outlier proportion bound and Bonferroni P ≤ 1e-10), compute all pairwise gene co-expression Q-values with Normalisr's coex function and retain significant edges at FDR Q ≤ 1e-15, a strong cutoff chosen to prioritize direct gene interactions. Apply Normalisr's gotop function to remove top principal components of the 'cytosolic part' and 'chromosome condensation' GO programs as covariates, then recompute co-expression — the GO pathway removal is inherently iterative because the set of dominant programs depends on the co-expression Q-value significance cutoff. Extract the major connected component from the final network, then test for over-abundance of GO and KEGG pathway gene pairs among edges using GOATOOLS and BioServices with Bonferroni correction at P ≤ 0.05. Correct application is confirmed when significant GO/KEGG pathways (33 GO and 8 KEGG in the paper) show more edges than expected under randomly assigned annotations.

## Related tools

- **Normalisr** (Provides the coex, gotop, qc_outlier and normalization functions (lcpm, scaling_factor, normcov, compute_var, normvar) used to compute, prune, and iteratively recompute the co-expression network with GO-program covariates) — https://github.com/lingfeiwang/normalisr
- **GOATOOLS** (Used to retrieve GO pathway annotations and test for over-abundance of pathway gene pairs among co-expression network edges)
- **BioServices** (Used to retrieve KEGG pathway annotations for the edge over-abundance test)

## Examples

```
python3 -m normalisr -h  # CLI entry point; python functional interface: import normalisr.normalisr as norm; ?norm.gotop
```

## Evaluation signals

- Co-expression edges pass a strong FDR cutoff Q ≤ 1e-15, prioritizing more direct gene interactions
- Over-abundance testing at Bonferroni P ≤ 0.05 yields significant pathways comparable to the paper's 33 GO and 8 KEGG pathways with more edges than expected under randomly assigned annotations
- Each iteration removes top principal components of the dominant GO programs and recomputes co-expression until the 'cytosolic part' and 'chromosome condensation' programs no longer dominate
- The final network reduces to a single major connected component, saved as major_component_coex_network.tsv
- Outlier cells removed via qc_outlier respect a 2% prior outlier proportion bound with Bonferroni P ≤ 1e-10

## Limitations

- The iterative GO pathway removal is conditional on the chosen co-expression Q-value significance cutoff, so results depend on that threshold (Q ≤ 1e-15 here)
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations and does not reconstruct causal regulatory networks
- Only UMI-based datasets were analyzed; full-length platforms may need revalidation of the nonlinear cellular summary covariates
- Batch effects in multi-dataset integrative studies are not addressed and may need extra consideration

## Evidence

- [discussion] Iterative GO pathway removal in co-expression networks: "The GO pathway removal took an iterative process given the significance cutoff for co-expression Q-value"
- [discussion] Co-expression cutoff for dysfunctional T cells: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [discussion] Pathway annotation and testing tools: "We used GOATOOLS63 and BioServices64 for GO and KEGG pathways, respectively"
- [abstract] Confounder removal rationale: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels"
- [readme] Normalisr interface: "Normalisr is in python and provides a command-line and a python functional interface."
