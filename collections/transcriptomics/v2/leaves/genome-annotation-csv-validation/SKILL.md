---
name: genome-annotation-csv-validation
description: Use when preparing a CSV of genome annotation for cis-NAT identification
  with CCIVR and you must confirm the required id/Chr/Strand/Start/End columns (and
  optional expression columns) exist before running.
when_to_use_negative:
- The input is raw sequencing data (FASTQ) or an unmapped BAM — CCIVR expects already
  processed RNA-seq data and gene annotations, not raw reads; mapping (STAR), expression
  profiling (RSEM), and statistics (DESeq2) are done by third-party programs upstream.
- The CSV lacks one of the five required gene annotation columns (id, Chr, Strand,
  Start, End) and cannot be restructured — CCIVR will not run correctly on it.
- The analysis goal is de novo transcriptome assembly or cis-NAT discovery without
  any genome annotation — tools like NATpipe (Perl, de novo assembled transcriptomes)
  or NASTI-seq (R) are the appropriate alternatives.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: CCIVR
  role: Tool whose CSV input schema is being validated; it recognizes the id, Chr,
    Strand, Start, End column characters and runs the cis-NAT extraction.
  repo: https://github.com/CCIVR/ccivr.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:09e4b8b78a5601c2b6e1a57d02b22060df98368c3a0ec140d1bf3dac1384d346
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr.git
  tools:
  - CCIVR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genome-annotation-csv-validation

## Summary

Validate and prepare the CSV-formatted gene annotation input required by CCIVR, ensuring the five mandatory columns (id, Chr, Strand, Start, End) are present in any order, optionally with RNA-seq expression columns (TPM, FPKM, fold_change, padj).

## When to use

Use when preparing a CSV of genome annotation for cis-NAT identification with CCIVR and you must confirm the required id/Chr/Strand/Start/End columns (and optional expression columns) exist before running.

## When NOT to use

- The input is raw sequencing data (FASTQ) or an unmapped BAM — CCIVR expects already processed RNA-seq data and gene annotations, not raw reads; mapping (STAR), expression profiling (RSEM), and statistics (DESeq2) are done by third-party programs upstream.
- The CSV lacks one of the five required gene annotation columns (id, Chr, Strand, Start, End) and cannot be restructured — CCIVR will not run correctly on it.
- The analysis goal is de novo transcriptome assembly or cis-NAT discovery without any genome annotation — tools like NATpipe (Perl, de novo assembled transcriptomes) or NASTI-seq (R) are the appropriate alternatives.

## Inputs

- CSV-formatted gene annotation file with required columns: id, Chr, Strand, Start, End
- Optional extra gene annotation columns (e.g. GeneSymbol, gene_source, gene_biotype, GO_analysis)
- Optional RNA-seq expression profiling columns (TPM, FPKM, fold_change, padj)
- Test_file.csv from the CCIVR repository (human gene datasets with Huh-7 hepatoma RNA-seq expression, TGFβ stimulation) for validation runs

## Outputs

- Validated input CSV accepted by the ccivr command (columns recognized regardless of order)
- Confirmation of correct installation via ccivr --help and pip show ccivr

## How to apply

Read the CSV header and confirm that the five required gene annotations — id (e.g. ENSMUSG00000085715), Chr, Strand, Start (minimum gene location), and End (maximum gene location) — each appear in independent columns; column order is flexible because CCIVR recognizes column characters rather than positions. Confirm that the annotation data can be sourced from the Ensembl database (e.g. Ensembl GRCm39). Optionally verify additional columns: extra gene annotations (GeneSymbol, gene_source, gene_biotype, GO_analysis) and RNA-seq expression profiling columns such as TPM, FPKM, fold_change, and padj. After installing CCIVR (Python 3.8+), check the installation with 'ccivr --help' and 'pip show ccivr' before running the full cis-NAT extraction. If pip fails during installation of the tool, fall back to pip3 or a local install of the unzipped package.

## Related tools

- **CCIVR** (Tool whose CSV input schema is being validated; it recognizes the id, Chr, Strand, Start, End column characters and runs the cis-NAT extraction.) — https://github.com/CCIVR/ccivr.git

## Examples

```
ccivr --help
```

## Evaluation signals

- Install check passes: 'ccivr --help' executes and 'pip show ccivr' reports an installation location ($PATH correct).
- Running the distributed Test_file.csv produces the console < Result > block with total genes : 65065 and genes with cisNats : 27351 [42.04%].
- Per-type counts match the reference run: EB : 8112 [12.47%], FO : 12194 [18.74%], HH : 5992 [9.21%], TT : 6097 [9.37%].
- Two CSV outputs (Table.csv and Summary.csv) are written into a new ccivr_output directory.
- Full test-file run completes in a plausible runtime (README reference: 6 minutes 34 seconds on an iMac 2017, 3.4 GHz Intel Core i5; a few to dozens of minutes on standard hardware).

## Limitations

- Accuracy of the annotation CSV depends on the quality of gene annotation (structure and strand direction) deposited in databases such as Ensembl; errors there propagate to cis-NAT predictions.
- Column order is flexible but CCIVR matches on the literal column characters (id, Chr, Strand, Start, End), so headers must be exactly these characters.
- Installation fragility: 'pip install git+https://github.com/CCIVR/ccivr.git' fails without git (use the ZIP local install) and pip may need to be replaced by pip3; a failed check usually indicates a $PATH problem.

## Evidence

- [methods] The input file must contain five different gene annotations listed in their independent columns, as follows: id, Chr, Strand, Start, End: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] These gene annotations can be obtained from the Ensembl database.: "These gene annotations can be obtained from the Ensembl database."
- [readme] The order of these five columns is changeable because CCIVR recognizes their characters (i.e., id, Chr, Strand, Start, End) but not their order.: "The order of these five columns is changeable because CCIVR recognizes their characters"
- [readme] Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as "TPM", "FPKM", "fold_change", "padj": "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process"
- [readme] To check whether CCIVR is correctly installed, please type the following command: ccivr --help. If this does not work, please check the setting of the $PATH environment.: "To check whether CCIVR is correctly installed, please type the following command:"
