---
name: differential-expression-logfc-extraction
description: Use when a completed multi-method DEG results table (e.g. ExpHunterSuite
  output with vote-threshold calls and log2FC columns) needs a specific gene's logFC
  record extracted for reporting, validation, or downstream GSEA-style reuse.
when_to_use_negative:
- The input is raw counts or a count table — the DEG detection must already have been
  run via DEgenes Hunter before any logFC record exists to extract
- The gene of interest is not present in the prevalent DEG set (e.g. it fails the
  minimum vote threshold), so no validated logFC record can be recovered
- A full functional enrichment analysis of the DEG list is the goal, not extraction
  of a single-gene logFC record
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunterSuite
  role: R/Bioconductor package whose DEgenes Hunter pipeline produces the DEG results
    table from which the logFC record is extracted
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: Differential expression pipeline inside ExpHunterSuite that integrates multiple
    DE methods and vote-threshold calls into the DEG table
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R
  role: Environment used to load the DEG results table, subset the gene record, and
    write the logFC companion file
- name: Bioconductor
  role: Distribution channel for ExpHunterSuite used to install the package that generated
    the DEG input table
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:8b7b1a30f25880903a2c2ef19d5e9a029e858309ba3378a901e752e359cf9f18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - Bioconductor
  - DEgenes Hunter
  - ExpHunter Suite
  - ExpHunterSuite
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# differential-expression-logfc-extraction

## Summary

Extracts the per-gene log2 fold-change record (e.g. CACNA1H) from an ExpHunterSuite/DEgenes Hunter differentially expressed gene (DEG) results table built under a minimum vote threshold, writing it as a companion record alongside enrichment results.

## When to use

Use when a completed multi-method DEG results table (e.g. ExpHunterSuite output with vote-threshold calls and log2FC columns) needs a specific gene's logFC record extracted for reporting, validation, or downstream GSEA-style reuse.

## When NOT to use

- The input is raw counts or a count table — the DEG detection must already have been run via DEgenes Hunter before any logFC record exists to extract
- The gene of interest is not present in the prevalent DEG set (e.g. it fails the minimum vote threshold), so no validated logFC record can be recovered
- A full functional enrichment analysis of the DEG list is the goal, not extraction of a single-gene logFC record

## Inputs

- Prevalent PMM2-CDG DEG results table (TSV) from the upstream ExpHunterSuite differential expression task, containing gene identifiers, vote counts, and log2 fold-change columns
- Target gene identifier (e.g. CACNA1H)

## Outputs

- Companion record file (TSV) holding the gene's log2 fold-change value extracted from the DEG table
- In the same task context, functional_enrichment_results.tsv produced alongside the logFC record

## How to apply

Load the prevalent DEG results table produced by the upstream differential expression task (the DEgenes Hunter pipeline of ExpHunterSuite, which combines multiple DE methods using a minimum vote threshold cut-off, e.g. a threshold of 3 for the PMM2-CDG DEGs) into R. Subset the table to the gene of interest — here CACNA1H, the calcium channel gene implicated in PMM2-CDG — and extract its log2 fold-change field. Rationale: the paper stresses that incorrect fold-change estimation 'could lead to poor target selection' and affects downstream logFC-based analyses such as GSEA, so the record must carry the consensus-table logFC, not a single-method value. Write the single-gene record (e.g. CACNA1H_logFC.tsv) as a companion to the functional_enrichment_results.tsv. Validate by confirming the extracted logFC is negative, consistent with the underexpression finding for CACNA1H in PMM2-CDG.

## Related tools

- **ExpHunterSuite** (R/Bioconductor package whose DEgenes Hunter pipeline produces the DEG results table from which the logFC record is extracted) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (Differential expression pipeline inside ExpHunterSuite that integrates multiple DE methods and vote-threshold calls into the DEG table) — https://github.com/seoanezonjic/ExpHunterSuite
- **R** (Environment used to load the DEG results table, subset the gene record, and write the logFC companion file)
- **Bioconductor** (Distribution channel for ExpHunterSuite used to install the package that generated the DEG input table) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'deg <- read.delim("deg_results.tsv"); rec <- deg[deg$gene_id=="CACNA1H", "log2FC"]; write.table(rec, "CACNA1H_logFC.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- The extracted CACNA1H log2FC value is negative (value_in_range check), matching the underexpression finding
- The companion record contains exactly one row for the queried gene identifier, identical to the logFC value in the source DEG results table (diff against the parent table)
- The parent DEG table was produced with the stated minimum vote threshold (3 for the PMM2-CDG DEGs), so the record traces to a prevalent DEG
- The companion record is written in the same TSV tabular format as functional_enrichment_results.tsv
- Re-running the extraction on the same DEG table reproduces the identical logFC value (determinism check)

## Limitations

- The minimum vote threshold cut-off may not generalize: the authors note the optimum (2 for spike-in data by F-measure) 'may differ for other organisms, tissues and experimental designs' and are 'hesitant to extrapolate these findings to a more general rule'
- logFC estimates vary by DE method — for the PMM2-CDG experiment DESeq2 was 'by far the least conservative', so single-method logFC values can diverge from the consensus table record
- Incorrect fold-change estimation can propagate error: it 'could lead to poor target selection' and affects downstream logFC analyses such as GSEA

## Evidence

- [discussion] its incorrect estimation could lead to poor target selection. This also has important implications for downstream analyses that use logFC, such as the popular GSEA method: "its incorrect estimation could lead to poor target selection. This also has important implications for downstream analyses that use logFC, such as the popular GSEA method"
- [discussion] for the PMM2-CDG experiment DESeq2 was by far the least conservative: "for the PMM2-CDG experiment DESeq2 was by far the least conservative"
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [intro] Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge: "Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge"
- [readme] Execute file launch_DEgenesHunter.sh stored in specific folder.: [evidence span withheld — anchor status: external_doc]
