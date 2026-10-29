---
name: cross-species-promoter-element-profiling
description: Use when you have high-confidence TSS coordinates (e.g. from csRNA-seq
  or other nascent-transcription data) for any species and need to detect core promoter
  elements and assess motif positioning around the TSS.
when_to_use_negative:
- Input is already a set of raw sequences without TSS positions — ElemeNT 2023 expects
  sequences centered on known TSSs; element detection without a TSS anchor cannot
  support positional-enrichment validation
- Goal is de novo motif discovery rather than detection of known core promoter elements
  — ElemeNT 2023 searches defined core promoter elements (and user-defined motifs),
  not unmotivated motif discovery
- The task is only differential expression or peak-calling analysis with no promoter-element
  interpretation
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_3176
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Command-line (Perl) detector of core promoter elements run on the extracted
    ±100 bp TSS-centered FASTA sequences; also available as a web-based tool
  repo: https://github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Original Elements Navigation Tool whose CORE database of predicted core promoter
    elements around annotated TSSs covers 10 species, underpinning cross-species application
  repo: https://github.com/OritAdato/ElemeNT
- name: HOMER
  role: findPeaks used to call high-confidence TSS peaks from csRNA-seq data prior
    to sequence extraction
- name: Perl
  role: Implementation language of the ElemeNT command-line version; required to install
    and execute ElemeNT 2023 locally
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:c321f4cfadb74fabadfdfd0fe0dd47c1dd63559bfefcf88b7bd0ed8fe8cc5152
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT
  - ElemeNT 2023
  - HOMER
  - Perl
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_3176
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-species-promoter-element-profiling

## Summary

Detect and profile core promoter elements around transcription start sites in any sequenced species using the ElemeNT 2023 command-line (Perl) tool, demonstrating that element detection is not limited to D. melanogaster and can be applied to human and mouse TSS datasets such as csRNA-seq.

## When to use

Use when you have high-confidence TSS coordinates (e.g. from csRNA-seq or other nascent-transcription data) for any species and need to detect core promoter elements and assess motif positioning around the TSS.

## When NOT to use

- Input is already a set of raw sequences without TSS positions — ElemeNT 2023 expects sequences centered on known TSSs; element detection without a TSS anchor cannot support positional-enrichment validation
- Goal is de novo motif discovery rather than detection of known core promoter elements — ElemeNT 2023 searches defined core promoter elements (and user-defined motifs), not unmotivated motif discovery
- The task is only differential expression or peak-calling analysis with no promoter-element interpretation

## Inputs

- csRNA-seq or other TSS-mapping sequencing data (e.g. GEO accession GSE135498 for human K562 cells and mouse bone marrow-derived macrophages)
- High-confidence TSS peak coordinates (from HOMER findPeaks)
- Reference genome FASTA for sequence extraction
- ±100 bp core promoter sequences centered on each called TSS, in FASTA format

## Outputs

- Per-sequence core promoter element detections from ElemeNT 2023 (motif position and score)
- Per-species position, fraction, and median-score distributions for each core promoter element (element_distributions.csv)

## How to apply

Call TSS peaks from the raw sequencing data (e.g. csRNA-seq from GEO accession GSE135498, downloaded via wget/sra-tools) using HOMER findPeaks to obtain high-confidence TSS coordinates for each sample. Extract ±100 bp core promoter sequences centered on each called TSS into FASTA files using bedtools getfasta. Run the ElemeNT 2023 command-line implementation (Perl, from github.com/OritAdato/ElemeNT) on each FASTA file to detect core promoter elements. Aggregate element detections into position, fraction, and median-score distributions per species (e.g. human K562 and mouse bone marrow-derived macrophages) and tabulate them as element_distributions.csv. Validate the result by confirming that detected elements show preferential positional enrichment around the TSS, which ElemeNT 2023 uses to evaluate the quality of high-throughput TSS datasets; cross-species transfer is supported by the tool's CORE database expansion covering 10 species, ranging from worms to human.

## Related tools

- **ElemeNT 2023** (Command-line (Perl) detector of core promoter elements run on the extracted ±100 bp TSS-centered FASTA sequences; also available as a web-based tool) — https://github.com/OritAdato/ElemeNT
- **ElemeNT** (Original Elements Navigation Tool whose CORE database of predicted core promoter elements around annotated TSSs covers 10 species, underpinning cross-species application) — https://github.com/OritAdato/ElemeNT
- **HOMER** (findPeaks used to call high-confidence TSS peaks from csRNA-seq data prior to sequence extraction)
- **Perl** (Implementation language of the ElemeNT command-line version; required to install and execute ElemeNT 2023 locally) — https://github.com/OritAdato/ElemeNT

## Examples

```
perl ElemeNT2023/src/ElemeNT/element.pl --input K562_tss_promoters.fa
```

## Evaluation signals

- Detected core promoter elements show preferential positional enrichment around the TSS, consistent with ElemeNT 2023's intended use for evaluating high-throughput TSS dataset quality
- Per-species distributions (position, fraction, median score) are tabulated as element_distributions.csv with one set of rows per species (human K562 and mouse BMDM)
- Element detection succeeds on both human and mouse TSS-derived sequences, demonstrating applicability beyond D. melanogaster
- All analyzed sequences are ±100 bp around the called TSS, matching the core promoter window used for extraction
- Running the same sequences through the ElemeNT 2023 web interface reproduces the command-line element calls

## Limitations

- ElemeNT 2023 is focused on core promoter elements; searching sequence-specific DNA binding sites is possible but outside the primary scope
- Quality of element detection depends on the upstream TSS peak calls — poorly called TSSs will degrade the positional enrichment signal
- Demonstrated on csRNA-seq data from human K562 and mouse BMDM; transferability to species outside the CORE database coverage (10 species) was not assessed

## Evidence

- [results] Applying ElemeNT 2023 to available csRNA-seq data (GSE135498) of human K562 cells and mouse bone marrow-derived macrophages demonstrates that the use of ElemeNT 2023 is not limited to D. melanogaster.: [evidence span withheld — anchor status: not_found]
- [abstract] novel capabilities for evaluating high-throughput TSS dataset quality via motif positioning: "which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential"
- [abstract] CORE database expanded to 10 species: "is now expanded to cover 10 species, ranging from worms to human"
- [intro] ElemeNT is implemented in Perl and can search any user-defined motif: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [other] command-line version available at the GitHub repo: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [readme] README describes ElemeNT as a Perl tool for core promoter element detection with a web interface and command-line execution: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [readme] README folder layout includes install package and manuscript analysis sources: "The contents of the folders located under ElemeNT2023 include:"
