# bulk-rnaseq-differential-expression-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **trim_bulk_reads** — remove adapters and low-quality sequence from bulk RNA-seq reads  →  `rna-seq-adapter-quality-trimming`, `wgs-read-quality-trimming`, `bulk-rnaseq-read-qc-trimming`, `fastq-adapter-and-quality-trimming`
2. **quantify_raw_gene_counts** — pseudoalign reads and aggregate transcript estimates into raw gene counts  →  `kallisto-reference-assembly-pseudoalignment`, `kallisto-pseudoalignment-quantification`, `transcriptome-reference-index-preparation`, `kallisto-bus-pseudoalignment-to-utrome`
3. **test_differential_expression** — derive consensus differential-expression calls with per-gene logFC estimates  →  `rna-seq-differential-expression-analysis--864ee06faecb526b`, `differential-gene-expression-analysis--40640222da726826`, `deg-method-consensus-combination`, `multi-method-deg-consensus-voting`
4. **test_pathway_enrichment** — rank genes by their consensus logFC and identify pathways enriched at the extremes of that ranking  →  `gsea-prerank-pathway-enrichment-analysis`, `pathway-enrichment-estimation`, `gene-set-enrichment-analysis--1f9beeabb1328d9f`, `ssgsea-pathway-enrichment-scoring`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
