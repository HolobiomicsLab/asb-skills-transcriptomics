---
name: pwm-log-likelihood-motif-scanning
description: Use when a user provides a nucleotide sequence (e.g. FASTA) and needs
  rapid detection and annotation of core promoter elements or user-defined motifs,
  including on the antisense strand, without prior TSS annotation.
when_to_use_negative:
- The task requires experimentally verified TSS positions rather than motif-based
  prediction of core promoter elements
- The input is already an annotated feature table rather than raw nucleotide sequence
  to be scanned
- The analysis needs TSS dataset quality assessment or evolutionary conservation evaluation
  rather than element detection within a single sequence
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: PWM-based detection and curation of core promoter elements within user-provided
    sequences, available as web tool and command line
  repo: github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of the ElemeNT interactive tool used to run the motif
    scan
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:1a69955a24f8842bec55124c55eeb2a212569c29ee3759e380fde222d2810072
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pwm-log-likelihood-motif-scanning

## Summary

Scan a nucleotide sequence with position-specific weight matrices (PWMs) for core promoter elements (TATA, Inr, DPE, MTE) in both orientations, retaining matches whose PWM log-likelihood score meets an element-specific cutoff, and report element/strand/coordinate/score combinations with biologically relevant spacing constraints (e.g. DPE depends on an Inr at the correct distance).

## When to use

Use when a user provides a nucleotide sequence (e.g. FASTA) and needs rapid detection and annotation of core promoter elements or user-defined motifs, including on the antisense strand, without prior TSS annotation.

## When NOT to use

- The task requires experimentally verified TSS positions rather than motif-based prediction of core promoter elements
- The input is already an annotated feature table rather than raw nucleotide sequence to be scanned
- The analysis needs TSS dataset quality assessment or evolutionary conservation evaluation rather than element detection within a single sequence

## Inputs

- User-provided nucleotide sequence in FASTA format
- Element-specific PWMs and score cutoff thresholds for core promoter motifs (TATA, Inr, DPE, MTE) or user-defined motifs

## Outputs

- elements_report.tsv with element name, strand, coordinates, and PWM log-likelihood score per detected element
- List of biologically relevant element combinations (e.g. DPE reported only in combination with an Inr hit at the expected spacing)

## How to apply

Load the user-provided FASTA nucleotide sequence (e.g. with Biopython). For each core promoter element motif (TATA, Inr, DPE, MTE, or a user-defined motif such as a DNA-binding site), compute PWM log-likelihood scores along the forward strand, then repeat the scan on the reverse complement to cover the antisense strand. Retain only matches whose PWM log score meets or exceeds the element-specific cutoff threshold; do not require prior determination of the TSS. Apply element-combination logic so spacing-dependent elements (e.g. DPE) are only reported when an Inr motif is present at the biologically relevant distance from it. Write reported element names, strand, coordinates, and scores to a tabular report (e.g. elements_report.tsv).

## Related tools

- **ElemeNT 2023** (PWM-based detection and curation of core promoter elements within user-provided sequences, available as web tool and command line) — github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of the ElemeNT interactive tool used to run the motif scan)

## Examples

```
perl ElemeNT2023/src/ElemeNT/elementnt.pl --input sample.fasta --output elements_report.tsv
```

## Evaluation signals

- elements_report.tsv exists and contains element/position/score fields
- All retained element matches have PWM log scores >= the element-specific cutoff
- DPE entries appear only in combination with an Inr hit at the expected spacing (qualitative acceptance check)
- Hits are reported on both sense and antisense strands (reverse-complement scan was run)

## Limitations

- Element detection is PWM-based prediction around no prior TSS determination; spacing-dependent elements are reported only when the dependency motif (e.g. Inr for DPE) is found at the biologically relevant distance
- Command-line version requires installation via the installPackage zip; running via the web interface requires no coding skills

## Evidence

- [other] ElemeNT 2023 reports the location and PWM log score of identified elements (scores ≥ the defined cutoff) on both sense and antisense strands, and displays their biologically relevant combinations: [evidence span withheld — anchor status: not_found]
- [intro] Tool description: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] Accessibility and user-defined motifs: "accessible both as a fast web-based tool and via command line (no coding skills are required to run the tool). While this tool is focused on core promoter elements, it can also be used for searching"
- [readme] README folder layout: "ElemeNT/ElemeNT2023/installPackage - zip of the installation package of ElemeNT2023 command line version."
