---
name: cell-barcode-extraction-scrna-seq
description: Use when processing raw 3'-end tag scRNA-seq FASTQ (e.g., MCA GSE108097
  or HCL GSE134355) where cell barcodes/UMIs must be extracted from merged inserts
  and corrected against a whitelist before isoform quantification.
when_to_use_negative:
- Input data are 10x Genomics reads intended for de novo cleavage-site mapping — most
  10x reads do not span mRNA 3' end cleavage sites, so this MWS-based extraction/atlas
  route is inappropriate
- Reads have not yet been merged (e.g., raw paired-end inserts whose barcode/UMI segments
  only reconstruct after PEAR overlap merging)
- The goal is bulk RNA-seq or full-length transcript analysis where single-cell barcode/UMI
  recovery does not apply
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PEAR
  role: Merges overlapping paired-end MWS reads into full insert sequences from which
    cell barcodes/UMIs are extracted
- name: kallisto
  role: Pseudoaligns barcode-bearing reads via `kallisto bus` with a `tech` argument
    indicating the scRNA-seq technology
- name: bustools
  role: Corrects cell barcodes against a `bx_whitelist` of valid barcodes via `bustools
    correct`
- name: scUTRquant
  role: Consumes the barcode-resolved reads for gene and 3'UTR isoform quantification;
    exposes `min_umis` and `exclude_unannotated_cells` parameters
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:cd41533dadea9f10ad27ce234f6ef35883b73a66407e9e7500880745953ff8f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - PEAR
  - bustools
  - kallisto
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cell-barcode-extraction-scrna-seq

## Summary

Recover and validate cell barcodes and UMIs from single-cell RNA-seq reads in the Microwell-seq (MWS) 3'UTR cleavage-site atlas workflow, so that downstream gene and 3'UTR isoform quantification with scUTRquant is attributable to annotated cell types from the Mouse Cell Atlas (MCA) and Human Cell Landscape (HCL).

## When to use

Use when processing raw 3'-end tag scRNA-seq FASTQ (e.g., MCA GSE108097 or HCL GSE134355) where cell barcodes/UMIs must be extracted from merged inserts and corrected against a whitelist before isoform quantification.

## When NOT to use

- Input data are 10x Genomics reads intended for de novo cleavage-site mapping — most 10x reads do not span mRNA 3' end cleavage sites, so this MWS-based extraction/atlas route is inappropriate
- Reads have not yet been merged (e.g., raw paired-end inserts whose barcode/UMI segments only reconstruct after PEAR overlap merging)
- The goal is bulk RNA-seq or full-length transcript analysis where single-cell barcode/UMI recovery does not apply

## Inputs

- Paired-end Microwell-seq FASTQ files (MCA, GEO:GSE108097; HCL, GEO:GSE134355)
- PEAR-merged full-insert sequences containing cell barcodes and UMIs
- Barcode whitelist file of valid barcodes for `bustools correct`
- MCA/HCL cell type annotations
- kallisto transcriptome index and `tech` specification for the library chemistry

## Outputs

- Barcode-corrected BUS-format count/reads matrices keyed by cell barcode and UMI
- Cell-level UMI counts after `min_umis` and `exclude_unannotated_cells` filtering
- Reads assigned to MCA/HCL cell types for downstream CS peak calling and scUTRquant quantification

## How to apply

Merge overlapping paired-end reads with PEAR to reconstruct the full insert sequences that carry the 3'-end information, then extract the cell barcodes and UMIs from the merged reads. Run kallisto bus pseudoalignment with the `tech` argument specifying the scRNA-seq technology, and use `bustools correct` with a `bx_whitelist` file of valid barcodes to resolve sequencing errors in cell barcodes. Assign barcode-resolved reads to their cell types using the MCA/HCL cell annotations. During quantification, enforce `min_umis` (minimum number of UMIs per cell) and optionally set `exclude_unannotated_cells` so that cells failing the UMI threshold or lacking annotations are removed from the final output.

## Related tools

- **PEAR** (Merges overlapping paired-end MWS reads into full insert sequences from which cell barcodes/UMIs are extracted)
- **kallisto** (Pseudoaligns barcode-bearing reads via `kallisto bus` with a `tech` argument indicating the scRNA-seq technology)
- **bustools** (Corrects cell barcodes against a `bx_whitelist` of valid barcodes via `bustools correct`)
- **scUTRquant** (Consumes the barcode-resolved reads for gene and 3'UTR isoform quantification; exposes `min_umis` and `exclude_unannotated_cells` parameters) — https://github.com/Mayrlab/scUTRquant

## Examples

```
kallisto bus -i txome.idx -o kb_out -x $tech R1.fastq.gz R2.fastq.gz; bustools correct -w bx_whitelist.txt -i kb_out/output.bus -o kb_out/corrected.bus
```

## Evaluation signals

- Cells below the `min_umis` UMI threshold and, if `exclude_unannotated_cells=true`, unannotated cells are absent from the final output
- All retained cell barcodes reconcile against the provided `bx_whitelist` after `bustools correct`
- Barcode-assigned reads map onto the expected MCA/HCL cell type annotations for each library
- Downstream CS peak calling yields the expected candidate counts (mouse N=170,617; human N=150,191) in the coordinate-annotated GTF records, implying barcode/UMI extraction did not lose or misattribute reads

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, which prevents exact de novo mapping of mRNA 3' ends — this barcode-extraction route was validated on Microwell-seq data, not 10x CS discovery
- MWS-derived cleavage sites required rigorous quality control and filtering for internal priming; barcode extraction errors upstream would propagate into peak calls
- Unannotated cells must be explicitly excluded via `exclude_unannotated_cells`, otherwise they remain in the final output and can dilute cell-type-specific CS usage estimates

## Evidence

- [methods] Extract cell barcodes/UMIs from the merged reads and assign reads to their cell types using the MCA/HCL cell annotations.: [evidence span withheld — anchor status: not_found]
- [methods] barcode correction with bustools correct: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [methods] kallisto bus technology specification: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] min_umis cell filtering: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [methods] unannotated cell exclusion parameter: "`exclude_unannotated_cells`: boolean indicating whether unannotated cells should be excluded from the final output"
- [methods] PEAR merge rationale from source workflow: [evidence span withheld — anchor status: not_found]
