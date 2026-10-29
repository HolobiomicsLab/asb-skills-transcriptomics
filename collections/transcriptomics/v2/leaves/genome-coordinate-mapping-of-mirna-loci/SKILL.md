---
name: genome-coordinate-mapping-of-mirna-loci
description: Use when merging miRNA annotations from multiple public databases and
  sRNA-Seq predictions for a species with a reference genome, and all loci must be
  expressed in a single uniform genome coordinate system before selection.
when_to_use_negative:
- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq
  data from species without genome sequences
- Most sequencing reads do not map to the reference genome — unmapped reads are ignored
  in the current version, so coordinate mapping would drop those candidates
- Inputs are already in a single uniform coordinate system on the same reference build
  — re-translation is unnecessary
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: iwa-miRNA
  role: Framework providing the miRNATranslate module that merges miRNARetrival and
    miRNAPredict outputs into a uniform genomic format
  repo: https://github.com/cma2015/iwa-miRNA
- name: genomePrepare
  role: Formats the species' genome sequences and annotation, establishing the reference
    coordinate system onto which all miRNA loci are mapped
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web platform in which the iwa-miRNA modules, including coordinate translation,
    are organized with user-friendly interfaces
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:67d4e10ffe0dc22e880747d6427ac433212ce184e900ed8412364f6328768c75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - Galaxy
  - PmiREN
  - PsRNA
  - RefSeq
  - SRA
  - genomePrepare
  - iwa-miRNA
  - miRBase
  - psRNAtarget
  - sRNAanno
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# genome-coordinate-mapping-of-mirna-loci

## Summary

Unifies already annotated miRNAs (from miRBase, PmiREN, sRNAanno, PsRNA) and newly predicted sRNA-Seq-derived miRNA precursors into one genomic coordinate system defined by the reference genome, via iwa-miRNA's miRNATranslate step, so that database annotations and de novo predictions can be merged and compared locus-by-locus.

## When to use

Use when merging miRNA annotations from multiple public databases and sRNA-Seq predictions for a species with a reference genome, and all loci must be expressed in a single uniform genome coordinate system before selection.

## When NOT to use

- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Most sequencing reads do not map to the reference genome — unmapped reads are ignored in the current version, so coordinate mapping would drop those candidates
- Inputs are already in a single uniform coordinate system on the same reference build — re-translation is unnecessary

## Inputs

- Output of miRNARetrival: already annotated miRNAs from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0
- Output of miRNAPredict: predicted miRNA precursors from sRNA-Seq data (SRA accession numbers or uploaded FASTQ files)
- Formatted genome sequences and annotation from genomePrepare (e.g., IWGSC RefSeq v1.0)

## Outputs

- miRNA and miRNA precursors with a uniform format (genomic coordinates on the reference genome)
- Aggregated precursor candidate list ready for miRNASelection (e.g., 2857 wheat precursors)

## How to apply

First run genomePrepare on the species' genome sequences and annotation (e.g., IWGSC RefSeq v1.0 for bread wheat) to produce formatted reference files that define the target coordinate system. Then take the output of miRNARetrival (already annotated miRNAs harvested from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0) and the output of miRNAPredict (precursors predicted from SRA sRNA-Seq datasets, e.g., 95 bread wheat datasets) and run miRNATranslate to convert both sets into the uniform genomic coordinate format. The rationale is that the four public databases report annotations inconsistently and against different references, so aggregation without re-mapping would double-count or misalign loci; translation onto one reference lets the merged list (e.g., 2857 wheat precursor candidates) feed cleanly into miRNASelection. Verify the output by checking that every entry carries coordinates on the chosen reference and that the merged count is reproducible across reruns.

## Related tools

- **iwa-miRNA** (Framework providing the miRNATranslate module that merges miRNARetrival and miRNAPredict outputs into a uniform genomic format) — https://github.com/cma2015/iwa-miRNA
- **genomePrepare** (Formats the species' genome sequences and annotation, establishing the reference coordinate system onto which all miRNA loci are mapped) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web platform in which the iwa-miRNA modules, including coordinate translation, are organized with user-friendly interfaces) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- Every entry in the miRNATranslate output carries genomic coordinates on the single reference genome prepared by genomePrepare
- Merged candidate counts are reproducible; for the wheat case study the aggregated set equals 2857 precursor candidates (2030 selected, 2163 loci)
- Both sources (database-annotated miRNAs and sRNA-Seq predictions) are represented in the merged output with a uniform format
- Downstream miRNASelection accepts the translated output without format errors, confirming coordinate consistency
- Coordinates of database-annotated miRNAs match their original loci after re-mapping (no systematic offsets or lost entries)

## Limitations

- Reads that do not map to the reference genome are ignored in the current version of iwa-miRNA, so candidates on unmapped contigs are lost
- The skill requires a reference genome; it cannot handle sRNA-Seq data from species without genome sequences
- The aggregated list after merging certainly contains false positive hits inherited from the databases and predictions

## Evidence

- [methods] miRNATranslate Output from miRNARetrival and miRNAPredict miRNA and miRNA precursors with a uniform format: [evidence span withheld — anchor status: not_found]
- [methods] genomePrepare formats genome sequences and annotation: [evidence span withheld — anchor status: not_found]
- [intro] Framework for plant species with reference genomes: "Here, we present iwa-miRNA, a web-based framework for interactive annotation of miRNAs from plant species with reference genomes."
- [discussion] Unmapped reads are ignored: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."
- [readme] Comprehensive collection of miRNA candidates: "iwa-miRNA allows users to generate a comprehensive collection of miRNA candidates, and to interrogate miRNA annotation in a straightforward way"
