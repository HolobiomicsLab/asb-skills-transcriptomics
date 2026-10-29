---
name: sankey-diagram-visualization
description: Use when you have per-cell predicted labels from a classifier (e.g.,
  SciBet Bet predictions) plus known true labels and need to visualize label-to-label
  correspondence, such as cross-species mapping results.
when_to_use_negative:
- There is no label-to-label correspondence to display (e.g., unsupervised clustering
  output without reference annotations) — a Sankey flow between predicted and true
  labels is undefined.
- The task requires a per-gene statistic visualization (e.g., ΔS entropy rankings
  from SelectGene) rather than per-cell label flows.
- Query cells are not annotated with known labels, so no reference side of the flow
  diagram exists; a simple prediction bar chart would be appropriate instead.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: SciBet
  role: Produces the per-cell predicted labels (via SelectGene, Learn, Bet) that populate
    the Sankey flows between predicted mouse-derived labels and known human pancreas
    cell types
  repo: github.com/PaulingLiu/scibet
- name: HomoloGene (Build 68)
  role: Lookup table used to convert Tabula Muris mouse gene labels into human gene
    space so predicted labels are comparable to human pancreas annotations in the
    diagram
- name: tidyverse
  role: R dependency package of the scibet installation used for data aggregation
    and plotting workflows behind the manuscript figures
- name: ggsci
  role: R dependency package of the scibet installation providing scientific color
    palettes for figure rendering
- name: viridis
  role: R dependency package of the scibet installation providing color scales used
    in figure visualization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:1de2db38b15f89f736462f1f6e1aefb71223d40c5c6074d51b8ded1b672825ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomoloGene
  - HomoloGene (Build 68)
  - SciBet
  - Tabula Muris
  - ggsci
  - tidyverse
  - viridis
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# sankey-diagram-visualization

## Summary

Render a Sankey (alluvial) diagram linking predicted cell-type labels to known reference labels, as used to display SciBet's cross-species mapping of human pancreas cells classified with the mouse Tabula Muris model (~92% correct mapping, Fig. 3b). It turns per-cell classification output into a flow diagram whose ribbon widths encode the number of cells assigned to each predicted-versus-true label pair.

## When to use

Use when you have per-cell predicted labels from a classifier (e.g., SciBet Bet predictions) plus known true labels and need to visualize label-to-label correspondence, such as cross-species mapping results.

## When NOT to use

- There is no label-to-label correspondence to display (e.g., unsupervised clustering output without reference annotations) — a Sankey flow between predicted and true labels is undefined.
- The task requires a per-gene statistic visualization (e.g., ΔS entropy rankings from SelectGene) rather than per-cell label flows.
- Query cells are not annotated with known labels, so no reference side of the flow diagram exists; a simple prediction bar chart would be appropriate instead.

## Inputs

- Per-cell classification output from SciBet Bet (predicted label per human pancreas query cell)
- Known/annotated cell-type labels for the human pancreas query cells
- Mouse Tabula Muris training label set restricted to one-to-one human-mouse orthologues (HomoloGene Build 68)
- Optionally, confidence scores from conf_score (cutoffs 0.001–0.5) to withhold low-confidence assignments before aggregation

## Outputs

- Sankey diagram (e.g., Fig. 3b) linking predicted mouse-derived labels to known human pancreas cell types, with ribbon widths proportional to cell counts
- Aggregated per-cell-type prediction counts (cross-tabulation of predicted vs. known labels)
- Overall mapping accuracy (fraction of correctly mapped cells, ~92%)

## How to apply

First run the SciBet classification: restrict the Tabula Muris training matrix to one-to-one human-mouse orthologues via the NCBI HomoloGene Build 68 lookup table, select informative genes with SelectGene (default k=500 by the ΔS entropy statistic), build the multinomial reference with Learn (Laplace/add-one smoothing), and classify each human pancreas query cell with Bet by multinomial MLE. Then aggregate the per-cell predictions, counting cells for each (predicted mouse-derived label, known human pancreas cell type) pair. Render a Sankey diagram linking the predicted mouse-derived labels on one side to the known human cell types on the other, so that correct mappings appear as thick ribbons connecting orthologous types. The plotting dependencies of the scibet R package (ggsci, viridis, tidyverse) support this visualization, and the scripts directory of the repository contains the scripts producing the manuscript's quantitative results. Validate against the reported Fig. 3b result: ~92% of human pancreas cells should flow to the correct orthologous cell type.

## Related tools

- **SciBet** (Produces the per-cell predicted labels (via SelectGene, Learn, Bet) that populate the Sankey flows between predicted mouse-derived labels and known human pancreas cell types) — github.com/PaulingLiu/scibet
- **HomoloGene (Build 68)** (Lookup table used to convert Tabula Muris mouse gene labels into human gene space so predicted labels are comparable to human pancreas annotations in the diagram)
- **tidyverse** (R dependency package of the scibet installation used for data aggregation and plotting workflows behind the manuscript figures)
- **ggsci** (R dependency package of the scibet installation providing scientific color palettes for figure rendering)
- **viridis** (R dependency package of the scibet installation providing color scales used in figure visualization)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- The fraction of human pancreas cells flowing to the correct orthologous cell type is approximately 92%, matching the reported Fig. 3b cross-species result.
- Sum of all Sankey ribbon widths equals the total number of classified query cells (no cells dropped except those withheld by a conf_score cutoff in the 0.001–0.5 range).
- Every label on the predicted side originates from the Tabula Muris reference label set; every label on the true side comes from the human pancreas annotations.
- Each predicted label connects to at least one known human cell type (no empty source nodes) and the diagram reproduces the dominant orthologous mappings seen in Fig. 3b.

## Limitations

- The reported ~92% mapping depends on the one-to-one orthologue restriction (HomoloGene Build 68); many-to-many homologues are excluded, so genes and labels are limited to the orthologous intersection.
- Cells with low confidence scores (cutoffs 0.001–0.5) are not assigned and therefore will not appear in the diagram, changing ribbon totals depending on the chosen cutoff.
- Version numbers for supporting software are not reported, and no seed/randomization parameters are given for the repeated splits underlying the results being visualized.

## Evidence

- [results] Cross-species classification Sankey supports the ~92% mapping claim: [evidence span withheld — anchor status: not_found]
- [discussion] HomoloGene orthologue conversion underpins the label correspondence visualized: "We used the HomoloGene databases provided by NCBI (Build 68) to identify homologous genes between human and mouse"
- [discussion] Informative-gene selection feeding the classifications being visualized: "we selected genes with top ΔS as informative genes (500 by default)"
- [readme] Repository scripts reproduce the manuscript figures including the Sankey: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
- [readme] Plotting dependency packages for visualization are installed with SciBet: "install.packages("Rcpp")
install.packages("RcppEigen")
install.packages("ggsci")
install.packages("viridis")
install.packages("tidyverse")"
