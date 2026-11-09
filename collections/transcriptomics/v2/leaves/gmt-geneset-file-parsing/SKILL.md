---
name: gmt-geneset-file-parsing
description: Use when a single-cell analysis needs per-cell gene set scores from a
  .gmt file (e.g. LM22 signatures on pbmc3k AnnData), and the gene sets must be extracted
  and direction-resolved before scoring.
when_to_use_negative:
- Gene sets are obtained via the Decoupler/Omnipath API rather than a .gmt file —
  use the decoupler model-fetching step instead of file parsing
- The 'file' is not in Broad GMT format (e.g. a single-column gene list or JSON signatures)
  — a different parser is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Consumes the parsed .gmt gene sets as gene_set_file input to score_cells.with_gene_sets
    for per-cell scoring on the nearest neighbor graph
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Provides the AnnData object (e.g. sc.datasets.pbmc3k_processed()) whose cells
    the parsed gene sets are scored against
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:fb8a39dbcb7e3287e7c5a044341c073f29e6d27f6e5e9c0f24fa4e4e3ff2b088
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gmt-geneset-file-parsing

## Summary

Parse GMT (Gene Matrix Transposed) gene set files, such as cibersort_lm22.gmt, into per-set gene lists following MSigDB nomenclature with UP/DN direction suffixes, so gene sets can be passed to gssnng score_cells.with_gene_sets for nearest-neighbor-smoothed single-cell gene set scoring.

## When to use

Use when a single-cell analysis needs per-cell gene set scores from a .gmt file (e.g. LM22 signatures on pbmc3k AnnData), and the gene sets must be extracted and direction-resolved before scoring.

## When NOT to use

- Gene sets are obtained via the Decoupler/Omnipath API rather than a .gmt file — use the decoupler model-fetching step instead of file parsing
- The 'file' is not in Broad GMT format (e.g. a single-column gene list or JSON signatures) — a different parser is needed

## Inputs

- .gmt gene set file (one gene set per line, tab-delimited name/description/genes), e.g. cibersort_lm22.gmt
- MSigDB-style gene set nomenclature suffixes (default-up, _UP, _DN)

## Outputs

- Dictionary or list of parsed gene sets: name -> member gene list, usable as gssnng score_cells.with_gene_sets input
- Optionally a per-set direction label (up/down) derived from _UP/_DN suffixes

## How to apply

Read the .gmt file line by line: each line contains a gene set name, an optional description, and a tab-delimited member gene list (the Broad Institute GMT format). Apply the MSigDB nomenclature conventions used by gssnng — gene sets default to up-regulated, and suffixes like _UP or _DN (e.g. cibersort_lm22 names like B.cell.naive) indicate direction, so split or tag sets accordingly when directionality matters. Return a dict (or list) mapping gene set name to its member genes, which is then supplied to gssnng's score_cells.with_gene_sets as gene_set_file or parsed equivalent. Validate by confirming every gene set line from the file is represented downstream and no member genes were dropped.

## Related tools

- **gssnng** (Consumes the parsed .gmt gene sets as gene_set_file input to score_cells.with_gene_sets for per-cell scoring on the nearest neighbor graph) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Provides the AnnData object (e.g. sc.datasets.pbmc3k_processed()) whose cells the parsed gene sets are scored against) — https://github.com/IlyaLab/gssnng

## Examples

```
from gssnng.score_cells import with_gene_sets; with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', score_method='geneset_overlap', groupby='louvain', smooth_mode='off', ranked=False)
```

## Evaluation signals

- Every gene set line in the .gmt file yields a parsed set — no sets silently dropped
- Member gene lists contain only the genes after the name/description fields of the GMT line (no off-by-one leading gene)
- Direction suffixes (default-up, _UP, _DN) are respected per MSigDB nomenclature when used by gssnng
- Downstream geneset_overlap output has one column per parsed gene set and fraction-mode scores within [0, 1]

## Limitations

- The parser inherits MSigDB naming conventions; gene set files that do not follow the default-UP / _UP / _DN suffix scheme cannot have directionality inferred automatically
- Genes in the .gmt that are absent from the AnnData var_names will be ignored during scoring, so overlap fractions depend on the expression matrix's gene universe

## Evidence

- [readme] GMT file path and format: "The gene set file with list of gene sets, gmt, one per line."
- [methods] MSigDB nomenclature: "We are following the MSigDB nomenclature, where gene sets default to up"
- [readme] gmt scoring entry point: "Score cells using .gmt gene set files"
- [methods] concrete gmt example file: "gene_set_file='cibersort_lm22.gmt', # File path of gene sets"
