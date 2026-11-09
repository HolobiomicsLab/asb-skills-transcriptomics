---
name: polya-site-usage-boxplot-visualization
description: Use when a gene's proximal vs distal poly(A) site usage (e.g., RUD scores
  from getAPAindexPACds) must be inspected as per-cell distributions across defined
  cell groups such as SC/RS/ES sperm stages.
when_to_use_negative:
- When genome-browser-like tracks of pA positions, read alignments, or per-cell read
  counts are needed — use vizTracks instead.
- When summarizing many APA markers across cell types at once (violin/heatmap/bubble/UMAP
  of marker lists) — use vizAPAMarkers.
- When the input is raw Sierra or scAPAtrap output rather than a PACdataset with a
  computed APA index; convert with readPACds and compute RUD first.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3517
tools:
- name: vizAPA
  role: Supplies vizStats for rendering box/dot/violin charts of per-gene poly(A)
    site usage across cell groups, and the demo scPACds PACdataset
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Wraps movAPA::movAPAindex via getAPAindexPACds to compute the per-gene RUD
    (proximal-distal) APA index fed into vizStats
  repo: https://github.com/BMILAB/movAPA
- name: R
  role: Runtime environment for vizAPA plotting (ggplot2-based figures, R >= 4.2.2
    recommended)
- name: scAPAtrap
  role: Alternative upstream source of poly(A) sites whose results can be read into
    a PACdataset and visualized with vizStats
  repo: github.com/BMILAB/scAPAtrap
- name: Sierra
  role: Alternative upstream tool for detecting poly(A) sites from scRNA-seq BAM files;
    its PAC data is importable into vizAPA
  repo: github.com/VCCRI/Sierra
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:cbbbc6b781dbcdcb375ad87f51653a5e32325622ba1a62b80a43321c49c0ff31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Sierra
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# polya-site-usage-boxplot-visualization

## Summary

Visualize the usage (RUD index) or expression of specific poly(A) site(s) of a gene across cell groups with vizAPA's vizStats module, which renders box, dot, violin, and bubble plots from a PACdataset-derived APA index; it complements heatmap-style averaged views by showing per-cell distributions.

## When to use

Use when a gene's proximal vs distal poly(A) site usage (e.g., RUD scores from getAPAindexPACds) must be inspected as per-cell distributions across defined cell groups such as SC/RS/ES sperm stages.

## When NOT to use

- When genome-browser-like tracks of pA positions, read alignments, or per-cell read counts are needed — use vizTracks instead.
- When summarizing many APA markers across cell types at once (violin/heatmap/bubble/UMAP of marker lists) — use vizAPAMarkers.
- When the input is raw Sierra or scAPAtrap output rather than a PACdataset with a computed APA index; convert with readPACds and compute RUD first.

## Inputs

- PACdataset of poly(A) site counts with cell-level colData (e.g., vizAPA demo scPACds: 974 pAs, 413 genes)
- Per-gene RUD PACdataset produced by getAPAindexPACds(choose2PA='PD') on 3'UTR-subset sites
- Cell-group annotation factor (e.g., celltype with levels SC, RS, ES)
- Gene identifier (e.g., Asrgl1, entrezid 66514, ENSMUSG00000024654)

## Outputs

- ggplot2 boxplot (or dot/violin/bubble chart) of the gene's RUD or pA usage distribution per cell group
- Exported figure file (e.g., asrgl1_rud_boxplot.png or figures.pptx via eoffice::topptx)

## How to apply

Load the PACdataset (e.g., data(scPACds, package='vizAPA'), the demo of mouse sperm cells with 974 pAs from 413 genes), set the cell-type factor order to match biology (SC < RS < ES), subset to anno$ftr=='3UTR', and compute the proximal–distal RUD index with getAPAindexPACds(choose2PA='PD'), which wraps movAPA::movAPAindex. Select the gene of interest (e.g., Asrgl1, entrezid 66514) and call vizStats(RUD, group='celltype', gene=gene, figType='box') so per-cell values are plotted unaveraged; use figType='dot' or 'violin' for alternative per-cell views. Save the returned ggplot2 object (e.g., as PNG or via eoffice::topptx). Correct application is judged by a successfully rendered figure whose group-wise pattern matches the expected APA dynamics, e.g., decreasing RUD from SC to ES reflecting 3'UTR shortening during sperm differentiation.

## Related tools

- **vizAPA** (Supplies vizStats for rendering box/dot/violin charts of per-gene poly(A) site usage across cell groups, and the demo scPACds PACdataset) — https://github.com/BMILAB/vizAPA
- **movAPA** (Wraps movAPA::movAPAindex via getAPAindexPACds to compute the per-gene RUD (proximal-distal) APA index fed into vizStats) — https://github.com/BMILAB/movAPA
- **R** (Runtime environment for vizAPA plotting (ggplot2-based figures, R >= 4.2.2 recommended))
- **scAPAtrap** (Alternative upstream source of poly(A) sites whose results can be read into a PACdataset and visualized with vizStats) — github.com/BMILAB/scAPAtrap
- **Sierra** (Alternative upstream tool for detecting poly(A) sites from scRNA-seq BAM files; its PAC data is importable into vizAPA) — github.com/VCCRI/Sierra

## Examples

```
vizStats(RUD, group='celltype', gene=gene, figType="box")
```

## Evaluation signals

- The ggplot2 boxplot renders successfully for the chosen gene and group factor
- Group-wise pattern matches expected biology: average RUD decreases SC -> RS -> ES (3'UTR shortening during sperm cell differentiation)
- For Asrgl1, the proximal site shows higher expression than the distal site across the three spermatogenesis stages
- Per-cell values are unaveraged in the plot (distribution shapes visible), distinguishing the boxplot from heatmap-style averaged summaries
- The RUD index was computed on 3'UTR-subset sites via getAPAindexPACds(choose2PA='PD'), as required for proximal–distal comparison

## Limitations

- Only genes with 3'UTR APA (multiple proximal/distal pAs) can yield RUD scores; single-site genes are not plottable this way
-  vizStats charts of individual genes show per-cell distributions, not averaged marker summaries — unsuitable for multi-gene overviews (use vizAPAMarkers)
- Results depend on correct cell-group factor ordering; misordered levels (SC/RS/ES) will obscure stage-progressive trends
- As with all APA analyses, internal priming artifacts should be removed (e.g., removePACdsIP) before trusting usage plots

## Evidence

- [readme] vizStats that generates various charts, including violin plot, box plot, bubble plot, and dot plo, to visualize usages of any given poly(A) site(s): [evidence span withheld — anchor status: external_doc]
- [methods] The RUD score of this gene is decreased from SC to ES, suggesting the APA dynamics during mouse spermatogenesis.: "The RUD score of this gene is decreased from SC to ES, suggesting the APA dynamics of of this gene during mouse spermatogenesis."
- [methods] The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES).: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES)."
