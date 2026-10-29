---
name: kshv-microrna-target-hybrid-identification
description: Use when analyzing hyb-format chimeric RNA data from a miRNA/target ligation
  experiment (e.g., qCLASH) to isolate hybrids containing a specific miRNA such as
  kshv-miR-k12-5 and enumerate its experimentally observed targets.
when_to_use_negative:
- Input data is not in the hyb file format (e.g., raw FASTQ reads or pre-computed
  interaction tables) — hybkit operates on hyb-format chimeric RNA records
- The goal is analyzing targets pooled across multiple experimental replicates — use
  Hybkit's Grouped Target Analysis instead of the single-replicate Target Analysis
- The question is predicted miRNA folding patterns rather than target identification
  — the Fold Analysis mode is the appropriate workflow
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: hybkit
  role: Core Python API and command-line toolkit used to load hyb records, QC, filter,
    assign types/miRNAs, run the Target Analysis, and plot/CSV outputs
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API-based workflow (Python3.8+)
  repo: https://pypi.org/project/hybkit/
- name: matplotlib
  role: Rendering target distribution plots via hybkit.plot (version >= 3.7.1)
  repo: https://matplotlib.org/
- name: BioPython
  role: Dependency of hybkit (>= 1.79) supporting its record/file infrastructure
  repo: https://biopython.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:dfd97d41dcc2eacef0169a4f14d22b1880f393bc6843f14f0ef6e1bc6164e9e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Python3
  - hybkit
  - matplotlib
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kshv-microrna-target-hybrid-identification

## Summary

A documented Hybkit example workflow for identifying hybrids containing the KSHV miRNA kshv-miR-k12-5 in public qCLASH data (GEO GSE101978, hyb format) and characterizing its targets via the Hybkit Target Analysis, exposed through both the Python API and command-line utilities.

## When to use

Use when analyzing hyb-format chimeric RNA data from a miRNA/target ligation experiment (e.g., qCLASH) to isolate hybrids containing a specific miRNA such as kshv-miR-k12-5 and enumerate its experimentally observed targets.

## When NOT to use

- Input data is not in the hyb file format (e.g., raw FASTQ reads or pre-computed interaction tables) — hybkit operates on hyb-format chimeric RNA records
- The goal is analyzing targets pooled across multiple experimental replicates — use Hybkit's Grouped Target Analysis instead of the single-replicate Target Analysis
- The question is predicted miRNA folding patterns rather than target identification — the Fold Analysis mode is the appropriate workflow

## Inputs

- hyb-format hybrid sequence file from qCLASH data (NCBI GEO accession GSE101978)
- Associated vienna (or CT) fold file containing predicted secondary structure for the hybrids
- hybkit type_finder criteria for assigning segment types and miRNA identities (e.g., MirBase-based miRNA identifiers)

## Outputs

- CSV count tables of kshv-miR-k12-5 target counts
- PNG target distribution plot rendered with hybkit.plot
- Filtered subset of HybRecord entries containing the kshv-miR-k12-5 miRNA

## How to apply

Load the qCLASH hyb-format data from GEO accession GSE101978 using hybkit's HybFile iterator, then run quality control on each hyb record (hyb_check-style integrity evaluation) and remove artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter. Evaluate each record with HybRecord.eval_types() and HybRecord.eval_mirna() so segment types and miRNA identities are assigned using hybkit's type_finder criteria. Select the subset of records containing the KSHV miRNA kshv-miR-k12-5 (optionally with hyb_filter --filter any_seg_contains kshv on the hyb and matching vienna file) from the single experimental replicate. Run the Hybkit Target Analysis (Analysis class or hyb_analyze target mode) to count and characterize the targets of kshv-miR-k12-5, exporting miRNA/target counts as CSV tables and rendering target distribution plots with hybkit.plot (requires matplotlib >= 3.7.1). The rationale is that hybkit assigns record segment types and miRNA identities consistently across its API and CLI, so the same filtering and target-counting logic is reproducible either way.

## Related tools

- **hybkit** (Core Python API and command-line toolkit used to load hyb records, QC, filter, assign types/miRNAs, run the Target Analysis, and plot/CSV outputs) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API-based workflow (Python3.8+)) — https://pypi.org/project/hybkit/
- **matplotlib** (Rendering target distribution plots via hybkit.plot (version >= 3.7.1)) — https://matplotlib.org/
- **BioPython** (Dependency of hybkit (>= 1.79) supporting its record/file infrastructure) — https://biopython.org/

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv && hyb_analyze target my_hyb_file.hyb
```

## Evaluation signals

- The CSV count tables and PNG target distribution plot exist and are parseable in tabular/graph form
- Per-miRNA target counts are internally consistent with the number of miRNA-containing records analyzed in the single replicate
- hyb_check-style QC completed without flagged data-integrity errors on the hyb records
- Ribosomal- and mitochondrial-RNA hybrids are absent from the filtered record set prior to target counting
- Every record entering the Target Analysis was assigned a miRNA identity via HybRecord.eval_mirna() and segment types via HybRecord.eval_types()

## Limitations

- The Target Analysis mode analyzes targets of a set of miRNAs from a single experimental replicate; pooled replicates require Grouped Target Analysis
- Artifactual ribosomal- and mitochondrial-RNA hybrids must be explicitly removed with hyb_filter before analysis, or counts will be contaminated
- Duplicate hybrids (by HybRecord.id) are not removed automatically; hyb_filter provides duplicate filtration options that must be applied if needed
- Python3.8+, matplotlib >= 3.7.1, BioPython >= 1.79, and typing_extensions >= 4.8.0 are required dependencies

## Evidence

- [readme] Hybkit provides several example analyses for hyb data using the utilities provided in the toolkit.: "Hybkit provides several example analyses for hyb data using the utilities provided in the toolkit."
- [methods] Analyze targets of a set of miRNAs from a single experimental replicate: "Analyze targets of a set of miRNAs from a single experimental replicate"
- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`): "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [readme] hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [methods] data acquired from the NCBI Gene Expression Omnibus (GEO) accession GSE101978: "data acquired from the NCBI Gene Expression Omnibus (GEO) accession `GSE101978 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE101978>`_"
