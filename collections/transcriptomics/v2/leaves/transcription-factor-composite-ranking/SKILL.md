---
name: transcription-factor-composite-ranking
description: Use when DC3 has assigned scRNA-seq and scATAC-seq cells to subpopulations
  and you need to rank transcription factors driving each subpopulation by joint expression
  and motif enrichment evidence.
when_to_use_negative:
- When only bulk RNA-seq or bulk ATAC-seq is available without single-cell data, since
  subpopulation-specific FPKM and subpopulation-specific peak calling require DC3
  cell-to-subpopulation assignments
- When the goal is deconvolution or clustering itself rather than downstream regulator
  ranking — use the core DC3 workflow instead
- When candidate regulators are not transcription factors or lack motif models, since
  the importance score requires motif enrichment evidence
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DC3
  role: Clusters single cells into subpopulations and deconvolves bulk profiles to
    provide the subpopulation assignments used to merge cells for expression and accessibility
    computation
  repo: github.com/SUwonglab/DC3
- name: MACS2
  role: Calls open chromatin peaks on scATAC-seq reads merged per DC3 subpopulation
- name: Homer
  role: Computes motif enrichment scores on subpopulation-specific peaks; score =
    geometric mean of -log10(p-value) and fold change
- name: scipy
  role: Performs t-tests for differential expression with adjusted p-value < 0.01
    used in the key regulator filter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:767bd0efa6915417e5657811f91cb44dadec828c3e3af46c493b96d7a7bbeee2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Homer
  - MACS2
  - scipy
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# transcription-factor-composite-ranking

## Summary

Identifies and ranks subpopulation-specific key transcription factors by combining DC3-derived subpopulation gene expression (FPKM) with scATAC-seq motif enrichment, using a composite importance score to surface regulators specific to each subpopulation.

## When to use

Use when DC3 has assigned scRNA-seq and scATAC-seq cells to subpopulations and you need to rank transcription factors driving each subpopulation by joint expression and motif enrichment evidence.

## When NOT to use

- When only bulk RNA-seq or bulk ATAC-seq is available without single-cell data, since subpopulation-specific FPKM and subpopulation-specific peak calling require DC3 cell-to-subpopulation assignments
- When the goal is deconvolution or clustering itself rather than downstream regulator ranking — use the core DC3 workflow instead
- When candidate regulators are not transcription factors or lack motif models, since the importance score requires motif enrichment evidence

## Inputs

- DC3 subpopulation assignments for scRNA-seq cells (e.g. scRNA-result.txt clustering output)
- DC3 subpopulation assignments for scATAC-seq cells (e.g. scATAC-result.txt clustering output)
- Per-cell scRNA-seq expression matrix (genes × cells)
- Per-cell scATAC-seq .bam files for peak calling
- Transcription factor list with associated position-weight matrices for Homer motif analysis

## Outputs

- Subpopulation-specific gene expression table in FPKM
- Subpopulation-specific open peaks called by MACS2 with Homer motif enrichment scores
- Key regulator lists per subpopulation (e.g. 58, 42, and 71 TFs for subpopulations 1, 2, and 3 of the RA-day 4 mixture)
- Top 30 ranked transcription factors per subpopulation by composite importance score

## How to apply

After DC3 clusters single cells and deconvolves profiles, merge scRNA-seq reads per subpopulation to compute subpopulation-specific FPKM, and merge scATAC-seq cells per subpopulation to call open peaks with MACS2. Score motifs on the subpopulation peaks with Homer, defining the motif enrichment score as the geometric mean of -log10(p-value) and fold change. Filter candidate key regulators to transcription factors with FPKM > 10, motif enrichment score > 2, and significant differential expression versus at least one other subpopulation (t-test, adjusted p-value < 0.01 via scipy). Rank TFs within each subpopulation by importance = log2(FPKM) × expression specificity (maximum expression fold change versus the other two subpopulations) × motif enrichment score, and output the Top 30 per subpopulation. The rationale is that a true subpopulation-specific regulator should be highly expressed, preferentially expressed, and have its motif enriched in that subpopulation's accessible chromatin.

## Related tools

- **DC3** (Clusters single cells into subpopulations and deconvolves bulk profiles to provide the subpopulation assignments used to merge cells for expression and accessibility computation) — github.com/SUwonglab/DC3
- **MACS2** (Calls open chromatin peaks on scATAC-seq reads merged per DC3 subpopulation)
- **Homer** (Computes motif enrichment scores on subpopulation-specific peaks; score = geometric mean of -log10(p-value) and fold change)
- **scipy** (Performs t-tests for differential expression with adjusted p-value < 0.01 used in the key regulator filter)

## Examples

```
python dc3.py -k 3 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt
```

## Evaluation signals

- Key regulator counts for the RA-day 4 dataset should equal 58, 42, and 71 for subpopulations 1, 2, and 3 respectively
- Lhx1 and Neurod1 should rank specifically in subpopulation 1; Gata4, Sox17, and Foxa2 in subpopulation 2; Rfx4 and Sox3 in subpopulation 3
- Every retained key regulator must satisfy all three filters: FPKM > 10, motif enrichment score > 2, and adjusted p-value < 0.01 from the t-test
- Motif enrichment scores should match the geometric mean of -log10(p-value) and fold change computed by Homer on MACS2-called subpopulation peaks
- Each subpopulation output should contain exactly the Top 30 ranked TFs ordered by the composite importance score

## Limitations

- Requires matched scRNA-seq and scATAC-seq samples from the same heterogeneous population plus DC3 clustering; cannot be applied to bulk-only or unmatched data
- Differential expression uses a t-test with adjusted p-value < 0.01 versus at least one other subpopulation, which may miss regulators with modest but coordinated differences across both comparator subpopulations
- Motif enrichment depends on the quantity of merged scATAC-seq reads per subpopulation; small subpopulations may yield too few peaks for reliable Homer scores

## Evidence

- [full_text] Key regulator criteria and thresholds: [evidence span withheld — anchor status: not_found]
- [results] Subpopulation-specific TF assignments: [evidence span withheld — anchor status: not_found]
- [methods] Motif enrichment scoring definition: [evidence span withheld — anchor status: not_found]
- [abstract] DC3 joint analysis capability: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [readme] README: DC3 outputs clustering results: "scATAC-result.txt                       the clustering results for scATAC-seq"
