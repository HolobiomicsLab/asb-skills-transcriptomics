---
name: sense-antisense-expression-combination-annotation
description: Use when you have CCIVR Table.csv output containing cis-NAT pairs with
  expression profiling columns (fold_change, padj) and need to tabulate pair counts
  per expression combination to find inversely correlated (down-up) pairs.
when_to_use_negative:
- Input lacks RNA-seq expression profiling columns (fold_change, padj) — CCIVR can
  extract cis-NATs structurally, but expression-combination classification is impossible
  without expression data
- Analysis requires de novo transcriptome assembly or strand-specific error-rate modeling
  rather than annotation-based cis-NAT extraction (use NATpipe or NASTI-seq instead)
- Gene annotation (structure and strand direction) accuracy in the source database
  is questionable, since CCIVR prediction quality depends entirely on it
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CCIVR
  role: Extracts cis-NAT pairs (EB, FO, HH, TT types) with structural characteristics
    and expression data from the annotated CSV input, producing Table.csv that is
    then classified by expression combination
  repo: github.com/CCIVR/ccivr
- name: DESeq2
  role: Upstream statistical analysis producing the fold_change and padj (q-value)
    values used to classify each strand of a pair as down or up
- name: RSEM
  role: Upstream expression profiling that generates the TPM/FPKM values accompanying
    the input gene dataset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:a74be9ae84ab76431855f446be1fd96efe460b9a6eae14c3620653c818da8fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# sense-antisense-expression-combination-annotation

## Summary

Classifies cis-NAT pairs extracted by CCIVR into expression-combination categories (down-down, down-up, up-down, up-up) per structural type (EB, FO, HH, TT) using the fold_change and padj columns of the CCIVR output table, enabling identification of inversely correlated sense-antisense pairs such as BCAR3-AS1/BCAR3 and GPX2/CHURC1.

## When to use

Use when you have CCIVR Table.csv output containing cis-NAT pairs with expression profiling columns (fold_change, padj) and need to tabulate pair counts per expression combination to find inversely correlated (down-up) pairs.

## When NOT to use

- Input lacks RNA-seq expression profiling columns (fold_change, padj) — CCIVR can extract cis-NATs structurally, but expression-combination classification is impossible without expression data
- Analysis requires de novo transcriptome assembly or strand-specific error-rate modeling rather than annotation-based cis-NAT extraction (use NATpipe or NASTI-seq instead)
- Gene annotation (structure and strand direction) accuracy in the source database is questionable, since CCIVR prediction quality depends entirely on it

## Inputs

- ccivr_output/Table.csv listing all extracted cis-NATs with structural characteristics (EB, FO, HH, TT) and expression profiling columns (TPM, FPKM, fold_change, padj)
- ccivr_output/Summary.csv with absolute number and percentage of each cis-NAT type
- Input CSV with gene annotations (id, Chr, Strand, Start, End) and optional expression columns from RNA-seq (e.g. Huh-7 cells with TGFβ stimulation)

## Outputs

- Expression-combination table (e.g. fig4b_cisnat_expression_combination_table.csv) counting cis-NAT pairs per down-down/down-up/up-down/up-up combination for each of EB, FO, HH, TT
- Subset of down-up (inversely correlated) cis-NAT pairs, optionally rendered as a heatmap (e.g. reproducing Figure 4D)
- Candidate pairs for functional validation (e.g. SETD2 knockdown expression dynamics and H3K36me3 promoter modification assays)

## How to apply

Run CCIVR on a CSV input containing gene annotations (id, Chr, Strand, Start, End) plus expression columns (TPM, FPKM, fold_change, padj), which produces ccivr_output/Table.csv and ccivr_output/Summary.csv. Parse Table.csv and classify each extracted sense-antisense pair by the expression combination of the two strands (down or up for each strand), using the fold_change column to determine direction and padj for significance. Tabulate pair counts per combination (down-down, down-up, up-down, up-up) separately for the four structural types (EB embedded, FO fully-overlapped, HH head-to-head, TT tail-to-tail), e.g. writing fig4b_cisnat_expression_combination_table.csv. Focus on the down-up (inversely correlated) pairs — e.g. fully-overlapped (FO) pairs — for downstream functional validation, as the paper did with BCAR3-AS1/BCAR3 and GPX2/CHURC1 upon TGFβ stimulation. Validate that Summary.csv reproduces the expected composition (for the Huh-7 TGFβ test file: 65,065 total genes, 27,351 [42.04%] with cis-NATs; EB 12.47%, FO 18.74%, HH 9.21%, TT 9.37%) and that combination-table totals match the cis-NAT counts in Table.csv.

## Related tools

- **CCIVR** (Extracts cis-NAT pairs (EB, FO, HH, TT types) with structural characteristics and expression data from the annotated CSV input, producing Table.csv that is then classified by expression combination) — github.com/CCIVR/ccivr
- **DESeq2** (Upstream statistical analysis producing the fold_change and padj (q-value) values used to classify each strand of a pair as down or up)
- **RSEM** (Upstream expression profiling that generates the TPM/FPKM values accompanying the input gene dataset)

## Examples

```
ccivr data/Test_file.csv --output ccivr_output && python classify_pairs.py ccivr_output/Table.csv --by fold_change,padj --combinations down-up,up-down,up-up,down-down
```

## Evaluation signals

- ccivr_output/Summary.csv for the Huh-7 TGFβ test file reports total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]
- Combination-table totals match the cis-NAT pair counts in Table.csv (sum of down-down + down-up + up-down + up-up per structural type equals type count)
- All extracted pairs fall into one of the four structural types (EB, FO, HH, TT), each obtained via plus-to-minus and minus-to-plus extracting passes
- Known inversely correlated pairs (BCAR3-AS1/BCAR3, GPX2/CHURC1) appear among the down-up fully-overlapped (FO) pairs from the TGFβ-stimulated Huh-7 analysis

## Limitations

- Prediction of cis-NAT composition by CCIVR depends on the accuracy of gene annotation, including structure and strand direction, deposited in the databases (e.g. Ensembl)
- CCIVR scripts do not cover the full analysis pipeline: it uses processed RNA-seq data produced by third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis)
- Completing the whole CCIVR process with the test file may take from a few to dozens of minutes on a standard-powered computer

## Evidence

- [readme] Input preparation and expression columns: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as "TPM", "FPKM", "fold_change""
- [readme] Output files: "Two CSV formatted output files "Table.csv" and "Summary.csv" will be generated and saved in a new directory named "ccivr_output""
- [readme] Expected test-file result: "total genes : 65065
genes with cisNats : 27351 [42.04%]"
- [abstract] TGFβ finding: "CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation"
- [intro] Structural characteristics rationale: "to elucidate the function of cis-NATs, it is critical to investigate both structural characteristics and expression profiles of cis-NAT pairs simultaneously"
- [discussion] Annotation dependence limitation: "it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases"
- [discussion] Pipeline dependence limitation: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
