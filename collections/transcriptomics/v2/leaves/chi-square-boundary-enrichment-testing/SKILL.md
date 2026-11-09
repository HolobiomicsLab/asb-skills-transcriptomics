---
name: chi-square-boundary-enrichment-testing
description: Use when spot-level positive/negative pathway labels from GSDensity (e.g.,
  BIOCARTA granulocyte or mesenchyme morphogenesis) must be tested for enrichment
  at anatomical boundaries in Visium ST data.
when_to_use_negative:
- The input is targeted-panel spatial data with limited numbers of genes, which GSDensity
  is explicitly not suitable for
- The gene set failed the compute.kld coordination pre-filter (n.grids=100, n.times=100),
  since binarization is only recommended on coordinated gene sets
- No anatomical boundary annotation exists for the section, making inside/outside
  boundary counts undefined
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Generates the spot-level pathway activity scores (run.rwr/run.rwr.list) and
    positive/negative labels (compute.cell.label) that feed the Chi-square contingency
    table
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: Loads and normalizes the 10X Visium cancer ST count matrices and supplies
    spot spatial coordinates
- name: msigdbr
  role: Supplies the BIOCARTA gene sets (granulocyte and mesenchyme morphogenesis)
    tested for boundary enrichment
- name: GSDensity_manuscript_code
  role: Manuscript code containing panc.preprocess.R / panc.process.R for the cancer
    Visium analysis supporting the pan-cancer ST map
  repo: https://github.com/qingnanl/GSDensity_manuscript_code
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:7f7493634028ce3276333f80dbd3f2ff352840b9801a60ba327a99302c10f9ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - GSDensity_manuscript_code
  - Seurat
  - future.apply
  - igraph
  - msigdbr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# chi-square-boundary-enrichment-testing

## Summary

A boundary-enrichment statistical test that determines whether a pathway's GSDensity-derived positive-label spots are over-represented inside versus outside anatomical region boundaries in 10X Visium spatial transcriptomics data, used to reproduce high-order organizational patterns in the pan-cancer ST map.

## When to use

Use when spot-level positive/negative pathway labels from GSDensity (e.g., BIOCARTA granulocyte or mesenchyme morphogenesis) must be tested for enrichment at anatomical boundaries in Visium ST data.

## When NOT to use

- The input is targeted-panel spatial data with limited numbers of genes, which GSDensity is explicitly not suitable for
- The gene set failed the compute.kld coordination pre-filter (n.grids=100, n.times=100), since binarization is only recommended on coordinated gene sets
- No anatomical boundary annotation exists for the section, making inside/outside boundary counts undefined

## Inputs

- 10X Visium spatial transcriptomics count matrices for cancer datasets (BC, CC, IC, OC, PACC, PC) normalized with Seurat
- GSDensity MCA co-embedding (compute.mca) and nearest-neighbor edge list (compute.nn.edges, nn.use=300)
- BIOCARTA gene sets (e.g., granulocyte and mesenchyme morphogenesis pathways) from msigdbr
- Spot spatial coordinates and anatomical boundary annotations per tumor section

## Outputs

- Per-pathway positive/negative spot labels (compute.cell.label output)
- 2x2 contingency tables of positive-label spot counts inside vs outside anatomical boundaries
- Chi-square test statistics and p-values per pathway-boundary pair, called significant at adjusted p < 0.05

## How to apply

First generate pathway activity weights per spot by running run.rwr.list on the cell-gene nearest-neighbor graph (compute.nn.edges with nn.use=300, restart=0.75) built from the MCA co-embedding, then binarize spots with compute.cell.label to obtain positive/negative pathway labels. Convert the spatial coordinates to a gridded format and define anatomical boundary regions for the tumor type under study. Build a 2x2 contingency table of positive-label spot counts inside versus outside each boundary and apply a Chi-square test to assess boundary enrichment. Call the pathway boundary-enriched when the test is significant (adjusted p < 0.05), and validate against the published pan-cancer ST map where the BIOCARTA granulocyte and mesenchyme morphogenesis pathways show significant boundary enrichment.

## Related tools

- **GSDensity** (Generates the spot-level pathway activity scores (run.rwr/run.rwr.list) and positive/negative labels (compute.cell.label) that feed the Chi-square contingency table) — https://github.com/qingnanl/gsdensity
- **Seurat** (Loads and normalizes the 10X Visium cancer ST count matrices and supplies spot spatial coordinates)
- **msigdbr** (Supplies the BIOCARTA gene sets (granulocyte and mesenchyme morphogenesis) tested for boundary enrichment)
- **GSDensity_manuscript_code** (Manuscript code containing panc.preprocess.R / panc.process.R for the cancer Visium analysis supporting the pan-cancer ST map) — https://github.com/qingnanl/GSDensity_manuscript_code

## Examples

```
cv <- run.rwr(el = el, gene_set = gene.set.list[["BIOCARTA_MESENCHYME_MORPHOGENESIS"]], cells = cells); cl <- compute.cell.label(cv); chisq.test(table(cl, spot_in_boundary))
```

## Evaluation signals

- The Chi-square test for the BIOCARTA granulocyte pathway shows significant boundary enrichment consistent with the published pan-cancer ST map
- The Chi-square test for the BIOCARTA mesenchyme morphogenesis pathway shows significant boundary enrichment (adjusted p < 0.05)
- Positive/negative labels derive from compute.cell.label applied to run.rwr probabilities with restart=0.75 on the nn.use=300 graph
- The 2x2 contingency table row and column sums reconcile with the total number of spots in the Visium section
- The pathway being tested passed the compute.kld coordination pre-filter (adjusted p < 0.05) before binarization

## Limitations

- Boundary enrichment depends on prior anatomical boundary annotation of the tumor section; mis-specified boundaries invalidate the contingency table
- PAL-based binarization can yield pathways with more than two modalities, so manual visual inspection of PAL distributions is recommended to validate automatic binarization
- GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes

## Evidence

- [readme] get a 'positive' or 'negative' label for each cell of this gene set: "get a 'positive' or 'negative' label for each cell of this gene set"
- [intro] weighted spatial density basis: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [abstract] pan-cancer ST map scope: "we create a pan-cancer ST map revealing spatially relevant and recurrently active pathways across six different tumor types"
- [readme] label propagation seeds: "fetch the most relevant cells for a gene set using the label propagation algorithm since the genes and cells can be placed in the same graph"
