---
name: strand-specific-cis-nat-pair-extraction
description: Use when you have a CSV of gene annotations (id, Chr, Strand, Start,
  End) from Ensembl, optionally with TPM/FPKM/fold_change/padj columns, and need to
  enumerate sense-antisense overlapping pairs with structural classification.
when_to_use_negative:
- Input is raw RNA-seq reads — CCIVR expects processed gene-level annotation and expression
  values; mapping/profiling must be done upstream with STAR/RSEM/DESeq2.
- You need strand-specific read-level assembly of antisense transcripts de novo (e.g.
  NATpipe-style) rather than annotation-based overlap extraction from known gene loci.
- Gene annotations in the source database are unreliable for strand direction or transcript
  structure — CCIVR's output accuracy depends entirely on deposited annotation accuracy.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CCIVR
  role: 'Core tool: extracts all cis-NAT pairs from annotation CSV, classifies EB/FO/HH/TT
    types, writes Table.csv and Summary.csv'
  repo: https://github.com/CCIVR/ccivr
- name: STAR
  role: Upstream mapping of RNA-seq reads to produce the processed expression data
    CCIVR consumes
- name: RSEM
  role: Upstream expression profiling (TPM/FPKM) supplying the expression columns
    attached to the CCIVR input CSV
- name: DESeq2
  role: Upstream statistical analysis producing fold_change and padj columns used
    to classify pairs as up/down per strand
- name: BEDTools
  role: Alternative prior approach for identifying overlapped cis-NAT pairs from genome
    annotation; CCIVR generalizes this with structural typing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:975b7b5f90350c52f14b8595e2ab3c5f51be7775bff0865468f01774ad98492d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - BEDTools
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# strand-specific-cis-nat-pair-extraction

## Summary

Use CCIVR to comprehensively extract all cis-NAT (cis-natural antisense transcript) pairs from a genome annotation CSV, classifying each pair into four structural types (EB, FO, HH, TT) via plus-to-minus and minus-to-minus passes, and optionally fold in RNA-seq expression profiling to find inversely correlated (down-up) pairs.

## When to use

Use when you have a CSV of gene annotations (id, Chr, Strand, Start, End) from Ensembl, optionally with TPM/FPKM/fold_change/padj columns, and need to enumerate sense-antisense overlapping pairs with structural classification.

## When NOT to use

- Input is raw RNA-seq reads — CCIVR expects processed gene-level annotation and expression values; mapping/profiling must be done upstream with STAR/RSEM/DESeq2.
- You need strand-specific read-level assembly of antisense transcripts de novo (e.g. NATpipe-style) rather than annotation-based overlap extraction from known gene loci.
- Gene annotations in the source database are unreliable for strand direction or transcript structure — CCIVR's output accuracy depends entirely on deposited annotation accuracy.

## Inputs

- CSV-formatted gene annotation file with required columns: id, Chr, Strand, Start, End (e.g. from Ensembl GRCm39/GRCh38)
- Optional expression profiling columns from processed RNA-seq: TPM, FPKM, fold_change, padj (e.g. Huh-7 TGFβ-stimulated data preprocessed with STAR/RSEM/DESeq2)

## Outputs

- ccivr_output/Table.csv — list of all extracted cis-NAT pairs with structural type (EB/FO/HH/TT) and attached expression data
- ccivr_output/Summary.csv — absolute number and percentage of each cis-NAT type relative to total genes

## How to apply

Install CCIVR with `pip install git+https://github.com/CCIVR/ccivr.git` and verify via `ccivr --help`. Build the input CSV with the five required annotation columns (id, Chr, Strand, Start, End; column order is flexible since CCIVR matches column names), optionally adding expression columns (TPM, FPKM, fold_change, padj) and extra annotations (GeneSymbol, gene_biotype). Run `ccivr [input CSV] --output [dir]`; the tool processes all four structural types in sequence (EB, FO, HH, TT), each via plus-to-minus and minus-to-minus extraction passes, writing Table.csv (list of all extracted cis-NATs with structural characteristics) and Summary.csv (counts and percentages per type) to a ccivr_output directory. With expression columns present, parse Table.csv and classify each pair by expression combination (down-down, down-up, up-down, up-up) using fold_change and padj to find inversely correlated pairs for downstream functional validation. Allow several minutes to tens of minutes runtime on a standard machine (~6.5 min for the 65,065-gene test file on an iMac 2017).

## Related tools

- **CCIVR** (Core tool: extracts all cis-NAT pairs from annotation CSV, classifies EB/FO/HH/TT types, writes Table.csv and Summary.csv) — https://github.com/CCIVR/ccivr
- **STAR** (Upstream mapping of RNA-seq reads to produce the processed expression data CCIVR consumes)
- **RSEM** (Upstream expression profiling (TPM/FPKM) supplying the expression columns attached to the CCIVR input CSV)
- **DESeq2** (Upstream statistical analysis producing fold_change and padj columns used to classify pairs as up/down per strand)
- **BEDTools** (Alternative prior approach for identifying overlapped cis-NAT pairs from genome annotation; CCIVR generalizes this with structural typing)

## Examples

```
pip install git+https://github.com/CCIVR/ccivr.git && ccivr data/Test_file.csv --output ccivr_output
```

## Evaluation signals

- Summary.csv on the human Huh-7 test file should report: total genes 65065; genes with cisNats 27351 [42.04%]; EB 8112 [12.47%]; FO 12194 [18.74%]; HH 5992 [9.21%]; TT 6097 [9.37%].
- Pair counts tabulated per expression combination (down-down, down-up, up-down, up-up) across EB/FO/HH/TT must sum to the total cis-NAT rows in Table.csv.
- Table.csv rows must be exhaustively partitioned into exactly four structural types (EB, FO, HH, TT) with each pair appearing in both plus-to-minus and minus-to-minus extraction passes.
- Console output must show all eight extraction passes (EB/FO/HH/TT × plus-to-minus/minus-to-minus) completing without error before Summary.csv is written.
- Known biological sanity check: the imprinted cis-NAT pair KCNQ1/KCNQ1OT1 should be recoverable from appropriate RNA-seq input, and BCAR3-AS1/BCAR3 and GPX2/CHURC1 should appear as fully-overlapped (FO) down-up pairs in the Huh-7 TGFβ test data.

## Limitations

- Accuracy depends on the accuracy of gene annotation (structure and strand direction) deposited in databases such as Ensembl/NCBI.
- CCIVR does not cover the full pipeline — it requires pre-processed RNA-seq data from third-party programs (STAR for mapping, RSEM for expression, DESeq2 for statistics).
- Processing the ~65k-gene test file takes several to dozens of minutes on a standard computer (e.g. 6 min 34 s on an iMac 2017 with 3.4 GHz Intel Core i5).
- pip installation from GitHub requires git; otherwise the ZIP must be downloaded and installed locally.

## Evidence

- [readme] Input file preparation: "The input file must contain five different gene annotations listed in their independent columns"
- [readme] Output structure: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**."
- [readme] Structural types extracted: "a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics, including embedded (EB), fully-overlapped (FO), head-to-head (HH), and tail-to-tail (TT)"
- [readme] Expected test-file results: "total genes : 65065
genes with cisNats : 27351 [42.04%]
     EB : 8112 [12.47%]
     FO : 12194 [18.74%]
     HH : 5992 [9.21%]
     TT : 6097 [9.37%]"
- [methods] Expression column attachment: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**,"
- [discussion] Annotation dependency limitation: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
