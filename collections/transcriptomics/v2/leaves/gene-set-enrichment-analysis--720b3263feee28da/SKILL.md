---
name: gene-set-enrichment-analysis
description: Use when you have per-factor gene rankings from a trained multi-omics
  factor model (e.g., Mowgli on TEA-seq) and need to interpret latent factors biologically
  by testing their top genes for enrichment against curated gene set databases.
when_to_use_negative:
- Input is a ranked gene list of arbitrary length rather than the top 150 genes per
  factor derived from normalized H(rna) rows
- The factors have not yet been trained (e.g., no Mowgli embedding/loading matrices
  exist) — factor training must precede enrichment
- Enrichment is needed for peaks (chromatin accessibility) rather than genes — use
  Signac's FindMotifs on the top 100 peaks per factor with the JASPAR2022 database
  instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Provides the enrich interface through which the gProfiler API is called on
    the per-factor gene lists
- name: gProfiler
  role: Web API backend performing the gene set enrichment tests against GO, Azimuth,
    and Immune SigDB sources
- name: Mowgli
  role: Produces the factor loading matrix H(rna) whose top 150 genes per factor are
    the enrichment input
  repo: https://github.com/cantinilab/Mowgli
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:68fac94ad95c77de50a261cc60673a8f6bf4aa5dfa3cb360a096a2ab2397112b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/Mowgli
  tools:
  - Azimuth
  - Human Protein Atlas
  - Mowgli
  - Python
  - Scanpy
  - Signac
  - gProfiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gene-set-enrichment-analysis

## Summary

Functional annotation of Mowgli factor loadings: the top 150 genes per latent factor (rows of H(rna) normalized to 1) are submitted as unordered gene lists to the gProfiler API via Scanpy's enrich to identify enriched GO:CC/MF/BP, Azimuth, and Immune SigDB gene sets that give each factor a biological identity.

## When to use

Use when you have per-factor gene rankings from a trained multi-omics factor model (e.g., Mowgli on TEA-seq) and need to interpret latent factors biologically by testing their top genes for enrichment against curated gene set databases.

## When NOT to use

- Input is a ranked gene list of arbitrary length rather than the top 150 genes per factor derived from normalized H(rna) rows
- The factors have not yet been trained (e.g., no Mowgli embedding/loading matrices exist) — factor training must precede enrichment
- Enrichment is needed for peaks (chromatin accessibility) rather than genes — use Signac's FindMotifs on the top 100 peaks per factor with the JASPAR2022 database instead

## Inputs

- Trained Mowgli model output: RNA loading matrix H(rna) (genes x factors) from a Muon MuData/AnnData object
- List of the 150 top genes per factor (rows of H(rna) normalized to 1), submitted as unordered gene sets
- gProfiler gene set sources: GO:CC, GO:MF, GO:BP, Azimuth, and Immune SigDB

## Outputs

- Per-factor enrichment table of gene sets (GO:CC/MF/BP, Azimuth, Immune SigDB terms) with Bonferroni-adjusted p-values < 0.05
- Factor-level biological interpretation panels supporting factor–cell-type assignment (as in Fig. 5)

## How to apply

After training Mowgli (e.g., 50 latent dimensions on TEA-seq with rho_rna=rho_adt=0.01, rho_atac=0.1, mu=0.001), normalize each row of the RNA factor matrix H(rna) to sum to 1, take the 150 top genes per factor, and pass them as an unordered input to the gProfiler API through Scanpy's enrich function. Restrict the gene set sources to GO cellular component, molecular function, and biological process ontologies plus Azimuth and Immune SigDB. Apply a Bonferroni-adjusted p-value threshold of 0.05 to call enriched terms per factor. Interpret enriched terms jointly with top proteins (H(adt)) and TF/peak signals, star biologically interesting genes using the Human Protein Atlas and Regulatory Circuits TF-gene links, and use the results to demonstrate the one-to-one factor–cell-type associations that distinguish Mowgli from MOFA+ and integrative NMF (reproducing Fig. 5).

## Related tools

- **Scanpy** (Provides the enrich interface through which the gProfiler API is called on the per-factor gene lists)
- **gProfiler** (Web API backend performing the gene set enrichment tests against GO, Azimuth, and Immune SigDB sources)
- **Mowgli** (Produces the factor loading matrix H(rna) whose top 150 genes per factor are the enrichment input) — https://github.com/cantinilab/Mowgli

## Examples

```
sc.queries.enrich(top_150_genes_per_factor, gprofiler_kwargs=['GO:CC', 'GO:MF', 'GO:BP', 'Azimuth_2021', 'ImmuneSigDB'])
```

## Evaluation signals

- Each factor contributes exactly 150 genes, obtained from rows of H(rna) normalized to sum to 1
- All reported gene sets pass Bonferroni-adjusted p < 0.05 and come only from the GO:CC/MF/BP, Azimuth, and Immune SigDB sources
- Enriched terms are consistent with the cell type assigned to the factor by marker-based Leiden clustering (e.g., immune/cytotoxic signatures for the NK- or CD8 T-cell-specific factors)
- Reproduced factor annotations support one-to-one factor–cell-type associations with high specificity scores, contrasting with the multi-factor aggregation seen for MOFA+ and integrative NMF

## Limitations

- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); the factors entering enrichment may depend on k and several values should be tested
- The Regulatory Circuits website used for TF-gene links to interpret enriched factors is down; data had to be recovered from a mirror
- Gene lists are treated as unordered input by gProfiler, so ranking information within the top 150 genes is discarded

## Evidence

- [methods] GSEA via gProfiler through Scanpy: [evidence span withheld — anchor status: not_found]
- [methods] Top 150 genes per factor as input: [evidence span withheld — anchor status: not_found]
- [methods] Complementary motif enrichment on top peaks: "Signac85 was used to perform Motif Enrichment Analysis, using the JASPAR2022 motif database86."
- [readme] Mowgli/Scanpy scverse integration: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
