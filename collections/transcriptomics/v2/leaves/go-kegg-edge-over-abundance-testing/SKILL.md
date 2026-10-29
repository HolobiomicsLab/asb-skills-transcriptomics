---
name: go-kegg-edge-over-abundance-testing
description: Use when you have a filtered gene co-expression network (e.g. Normalisr
  coex edges at Q ≤ 1e-15) and want to test whether genes annotated to the same GO
  or KEGG pathway are connected more often than expected by chance.
when_to_use_negative:
- You want to test arbitrary nonlinear gene–gene associations rather than co-expression
  captured by Normalisr's linear framework — the article states Normalisr is not designed
  for hypothesis testing of arbitrary nonlinear associations.
- The network was built from full-length (non-UMI) scRNA-seq — the study only focused
  on UMI-based datasets and did not validate the pipeline on full-length platforms.
- You need causal regulatory network reconstruction (v-structures, regularized regression,
  Granger causality) rather than functional enrichment of undirected co-expression
  edges.
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: Normalisr
  role: Upstream tool that produced the co-expression network (lcpm, scaling_factor,
    normcov, compute_var, normvar normalization followed by coex) whose edges are
    tested for pathway over-abundance
  repo: https://github.com/lingfeiwang/normalisr
- name: GOATOOLS
  role: Provides Gene Ontology annotations and enrichment machinery used to count
    gene pairs sharing GO terms among network edges
- name: BioServices
  role: Python interface used to fetch KEGG pathway annotations for gene pairs sharing
    KEGG pathways
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:940ae25449556e96b9ab4ee064572d46b6b8247bc1f614ed33587362383560a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# go-kegg-edge-over-abundance-testing

## Summary

Tests whether gene pairs sharing GO or KEGG pathway annotations are over-represented among edges of a single-cell co-expression network, using GOATOOLS (GO) and BioServices (KEGG) with Bonferroni correction. Applied to the Normalisr-derived dysfunctional CD8+ T cell network, it recovered significant edge over-abundance for 33 GO and 8 KEGG pathways.

## When to use

Use when you have a filtered gene co-expression network (e.g. Normalisr coex edges at Q ≤ 1e-15) and want to test whether genes annotated to the same GO or KEGG pathway are connected more often than expected by chance.

## When NOT to use

- You want to test arbitrary nonlinear gene–gene associations rather than co-expression captured by Normalisr's linear framework — the article states Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations.
- The network was built from full-length (non-UMI) scRNA-seq — the study only focused on UMI-based datasets and did not validate the pipeline on full-length platforms.
- You need causal regulatory network reconstruction (v-structures, regularized regression, Granger causality) rather than functional enrichment of undirected co-expression edges.

## Inputs

- Major connected component of a Normalisr gene co-expression network (significant edges at FDR Q ≤ 1e-15), e.g. major_component_coex_network.tsv
- Gene Ontology annotation database queried with GOATOOLS
- KEGG pathway database accessed through BioServices

## Outputs

- Pathway enrichment table of over-abundant GO/KEGG gene pairs among network edges (Bonferroni-corrected P ≤ 0.05), saved as pathway_enrichment.tsv
- Major-component co-expression network TSV (major_component_coex_network.tsv) written prior to enrichment testing

## How to apply

First restrict the co-expression network to its major connected component before testing. For every network edge (gene pair), determine whether the two genes share a GO term (via GOATOOLS) or a KEGG pathway (via BioServices). Under a null in which pathway annotations are randomly permuted across genes (preserving pathway sizes), count the expected number of edges connecting gene pairs with shared annotations; compare observed counts per pathway with a one-sided test. Apply Bonferroni correction over all tested pathways and call pathways significant at P ≤ 0.05. The rationale is that a biologically meaningful co-expression network should connect functionally related genes more often than random annotation assignment. Save the major-component network to major_component_coex_network.tsv and enrichment results to pathway_enrichment.tsv.

## Related tools

- **Normalisr** (Upstream tool that produced the co-expression network (lcpm, scaling_factor, normcov, compute_var, normvar normalization followed by coex) whose edges are tested for pathway over-abundance) — https://github.com/lingfeiwang/normalisr
- **GOATOOLS** (Provides Gene Ontology annotations and enrichment machinery used to count gene pairs sharing GO terms among network edges)
- **BioServices** (Python interface used to fetch KEGG pathway annotations for gene pairs sharing KEGG pathways)

## Evaluation signals

- At least one GO or KEGG pathway shows significant edge over-abundance after Bonferroni correction at P ≤ 0.05; the reference dysfunctional T cell analysis recovered 33 GO and 8 KEGG pathways.
- Observed edge counts for annotated pathway pairs exceed the random-annotation null expectation for every pathway reported as significant.
- The enrichment is computed on the major connected component of the network only, not the full edge list.
- Output file pathway_enrichment.tsv contains one row per tested pathway with a Bonferroni-corrected P value; major_component_coex_network.tsv contains the filtered edge list.
- The co-expression input must already be thresholded at the strong dysfunctional-T-cell cutoff (FDR Q ≤ 1e-15) before testing.

## Limitations

- Only UMI-based datasets were analyzed in the source work; full-length platforms may need revalidation of the nonlinear cellular summary covariates.
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations, so edge evidence is limited to linear co-expression relationships.
- The pipeline does not reconstruct causal regulatory networks or address batch effects in multi-dataset integrative studies.
- Statistical significance depends on the network Q-value cutoff; the GO pathway removal procedure in the source work was iterative given the co-expression Q-value cutoff chosen.

## Evidence

- [discussion] GOATOOLS and BioServices for GO and KEGG enrichment: "We used GOATOOLS63 and BioServices64 for GO and KEGG pathways, respectively"
- [discussion] Strong co-expression cutoff used for the dysfunctional T cell network: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [discussion] Iterative GO pathway procedure tied to co-expression cutoff: "The GO pathway removal took an iterative process given the significance cutoff for co-expression Q-value"
- [abstract] Normalisr co-expression networks recapitulate known gene functions: "On conventional scRNA-seq, Normalisr recovers gene-level co-expression networks that recapitulated known gene functions."
