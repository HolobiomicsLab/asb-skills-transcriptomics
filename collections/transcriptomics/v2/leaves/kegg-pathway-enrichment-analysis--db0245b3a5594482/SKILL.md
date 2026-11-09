---
name: kegg-pathway-enrichment-analysis
description: Use when you have a filtered differential network edge list (e.g. top
  ~1000 TF-filtered BoostDiff edges) with Louvain module assignments and need to test
  whether module/network genes are enriched in disease-relevant KEGG pathways.
when_to_use_negative:
- The input is raw, unfiltered BoostDiff output (boostdiff_network_<keyword>.txt)
  — filter first with filter_network p=3 and select TF-filtered top edges, otherwise
  enrichment runs on the full unfiltered network
- You need differential-expression testing rather than pathway over-representation
  — BoostDiff is described as complementing, not replacing, standard differential
  expression analyses
- The gene set is not mappable to KEGG pathway gene identifiers (e.g. non-standardized
  IDs not mapped to HGNC symbols as done with biomaRt in the Crohn's workflow)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gseapy
  role: Runs KEGG over-representation analysis via the Enrichr interface on module
    genes and exports the enrichment table to CSV
- name: python-louvain
  role: Louvain community detection partitions the filtered differential network into
    modules whose gene sets are then tested for KEGG enrichment
  repo: https://github.com/taynaud/python-louvain
- name: BoostDiff
  role: Produces the filtered differential network edge list whose modules serve as
    the gene lists for enrichment analysis
  repo: https://github.com/scibiome/boostdiff_inference
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:59854f1faeba0ab121abe106b17d3c73a6717a7cabd78ce88e9a53e9230a27a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taynaud/python-louvain
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - conda
  - dplyr
  - gseapy
  - matplotlib
  - networkx
  - numpy
  - pandas
  - pip
  - pygraphviz
  - python-louvain
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# kegg-pathway-enrichment-analysis

## Summary

Run KEGG over-representation analysis (via gseapy Enrichr) on genes from modules of a filtered BoostDiff differential network to test whether the inferred disease-specific modules recover known disease-relevant signaling pathways.

## When to use

Use when you have a filtered differential network edge list (e.g. top ~1000 TF-filtered BoostDiff edges) with Louvain module assignments and need to test whether module/network genes are enriched in disease-relevant KEGG pathways.

## When NOT to use

- The input is raw, unfiltered BoostDiff output (boostdiff_network_<keyword>.txt) — filter first with filter_network p=3 and select TF-filtered top edges, otherwise enrichment runs on the full unfiltered network
- You need differential-expression testing rather than pathway over-representation — BoostDiff is described as complementing, not replacing, standard differential expression analyses
- The gene set is not mappable to KEGG pathway gene identifiers (e.g. non-standardized IDs not mapped to HGNC symbols as done with biomaRt in the Crohn's workflow)

## Inputs

- Filtered differential network edge list (tab-separated TSV, e.g. brca_diff_edges.tsv; top ~1000 TF-filtered edges from filter_network with p=3)
- Louvain module assignment table mapping each gene to its community (e.g. brca_modules.csv)
- Gene lists per network module (HGNC gene symbols, e.g. PPARG, PCK1, CD36, PLIN4)

## Outputs

- KEGG enrichment table exported as CSV (e.g. brca_kegg_enrichment.csv) with significant pathway terms per module
- Module assignment CSV (e.g. disease-network module memberships from Louvain partition)

## How to apply

First filter the raw BoostDiff network (boostdiff_network_<keyword>.txt) with filter_network at p=3 (3rd percentile of the largest mean differences in prediction error between conditions) and retain the top ~1000 transcription-factor-filtered edges per condition. Partition the filtered differential network into modules with the Louvain algorithm (python-louvain, imported as 'community'), exporting per-gene module assignments to a CSV. Then run KEGG over-representation analysis on the module/network genes using gseapy's Enrichr interface and write the enrichment table to a CSV (e.g. brca_kegg_enrichment.csv). Judge success by checking that the enrichment table is a non-empty tabular CSV and that it recovers the disease-relevant pathways reported for the dataset: 'Coronavirus disease', TNF signaling, Th17 cell differentiation, NF-κB/NOD-like receptor/Toll-like receptor signaling for COVID-19 (GSE156063); TNF, NOD-like receptor, Th17, IL-17, NF-κB and inflammatory bowel disease terms for Crohn's disease (GSE126124); and PPAR signaling, insulin resistance, AMPK signaling and adipocytokine signaling for TCGA breast cancer.

## Related tools

- **gseapy** (Runs KEGG over-representation analysis via the Enrichr interface on module genes and exports the enrichment table to CSV)
- **python-louvain** (Louvain community detection partitions the filtered differential network into modules whose gene sets are then tested for KEGG enrichment) — https://github.com/taynaud/python-louvain
- **BoostDiff** (Produces the filtered differential network edge list whose modules serve as the gene lists for enrichment analysis) — https://github.com/scibiome/boostdiff_inference

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=module_genes, gene_sets='KEGG_2021_Human', outdir='enrichment'); enr.results.to_csv('brca_kegg_enrichment.csv')
```

## Evaluation signals

- The enrichment CSV exists and is a valid non-empty table for each module analyzed
- For the COVID-19 dataset (GSE156063) the enrichment table contains 'Coronavirus disease', TNF signaling, Th17 differentiation, NF-kappa B, NOD-like receptor and Toll-like receptor signaling terms
- For Crohn's disease (GSE126124) the enrichment table recovers TNF signaling, NOD-like receptor signaling, Th17 cell differentiation, IL-17 signaling, NF-κB signaling and inflammatory bowel disease terms
- For TCGA BRCA the module enrichment reproduces 'PPAR signaling pathway', 'Insulin resistance', 'AMPK signaling pathway' and 'Adipocytokine signaling pathway', and the edge list contains PPARG–PLIN4, PPARG–PCK1 edges and the CD36 hub
- The upstream edge list TSV is a non-empty tab-separated table filtered at the p=3 percentile before modules are derived

## Limitations

- Enrichment results depend on the differential network being built from well-preprocessed expression matrices: datasets may need gene ID mapping (biomaRt), probe aggregation per gene (dplyr mean) and sample selection before BoostDiff is run
- BoostDiff's differential network complements but does not replace standard differential expression analysis; enrichment alone does not demonstrate differential expression
- The module partition (Louvain) and the p=3 filtering percentile shape which genes enter the enrichment test, so pathway recovery is sensitive to these thresholds

## Evidence

- [abstract] BoostDiff identifies context-specific networks enriched with known disease-relevant pathway genes: "BoostDiff identifies context-specific networks that are enriched with genes of known disease-relevant pathways and complements standard differential expression analyses."
- [methods] Raw networks must be filtered for target genes before downstream use: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [readme] python-louvain is pip-installed but imported as 'community' for the Louvain module detection feeding enrichment: [evidence span withheld — anchor status: external_doc]
- [readme] GEO datasets used for the COVID-19 and Crohn's enrichment analyses: "The COVID-19 and Crohn's disease datasets are available in the Gene Expression Omnibus (GEO) database with accession numbers GSE156063 and GSE126124, respectively."
