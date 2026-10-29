---
name: chimeric-contig-classification-and-count-tabulation
description: Use when you have a de novo assembled transcriptome (e.g., Trinity contigs)
  plus a reference transcript set and need to quantify how many chimeric vs. correct
  vs. unidentified contigs a filter removed.
when_to_use_negative:
- Input is a reference-guided or genome-based assembly where chimeras are not the
  dominant failure mode — the skill targets de novo transcriptome assemblies with
  no reference genome available for quality assessment
- No reference transcript set or adequate annotation exists to define gene-level BLAST
  hits, so chimeric/nonchimeric/unidentified labels cannot be assigned
- You only need a single quality score for the assembly (e.g., a TransRate score)
  rather than per-contig chimera classification and removal statistics
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Bellerophon
  role: Pipeline whose filtered output contigs are classified and tabulated; runs
    TPM filter (cut-off 1), CD-HIT-EST (95% identity), and TransRate filtering in
    default order
  repo: github.com/JesseKerkvliet/Bellerophon
- name: BLASTn
  role: Aligns filtered contigs against the reference transcripts (E-value <= 1e-3,
    identity >= 90%, alignment length >= 300 nt) to assign chimeric/nonchimeric/unidentified
    labels
- name: Trinity
  role: Produces the de novo transcriptome assembly from SRA reads (SRR8735410–12)
    that is then classified and filtered
- name: TransRate
  role: Establishes the ground quality score of the unfiltered assembly and applies
    the final TransRate filtering step
- name: CD-HIT-EST
  role: Removes highly identical redundant contigs at the default 95% identity cutoff
    prior to classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:2699406090fcb63e2a8406c3cd05bb6dd5533cc89f09a6fdc01b1cfd34c56026
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chimeric-contig-classification-and-count-tabulation

## Summary

Classify each contig of a de novo transcriptome assembly as chimeric, nonchimeric, or unidentified by BLASTing against a reference transcript set, then tabulate how many contigs of each class a filtering pipeline (e.g., Bellerophon) removed. This quantifies whether filtering preferentially removes chimeras over correct sequences.

## When to use

Use when you have a de novo assembled transcriptome (e.g., Trinity contigs) plus a reference transcript set and need to quantify how many chimeric vs. correct vs. unidentified contigs a filter removed.

## When NOT to use

- Input is a reference-guided or genome-based assembly where chimeras are not the dominant failure mode — the skill targets de novo transcriptome assemblies with no reference genome available for quality assessment
- No reference transcript set or adequate annotation exists to define gene-level BLAST hits, so chimeric/nonchimeric/unidentified labels cannot be assigned
- You only need a single quality score for the assembly (e.g., a TransRate score) rather than per-contig chimera classification and removal statistics

## Inputs

- De novo assembled transcriptome FASTA (e.g., Trinity contigs, with contigs matching fewer than 5 reads removed)
- Unfiltered (pre-Bellerophon) assembly FASTA for the before-filtering baseline
- Reference transcript set (e.g., D. melanogaster transcripts from NCBI RefSeq GCF_000001215.4)
- RNA-Seq reads (FASTQ, paired-end) used to re-run or verify the assembly

## Outputs

- Per-contig class labels: chimeric (multiple non-overlapping gene matches), nonchimeric (exactly one gene match), unidentified (no gene matches)
- Counts and percentages of each contig class removed by the Bellerophon filter
- Unique-gene count in the original vs. filtered assembly

## How to apply

First run the filtering pipeline in its default order (TransRate/Busco ground quality assessment, TPM filter cut-off 1, CD-HIT-EST at 95% identity, TransRate filtering) on the de novo assembly built from reads such as SRA experiments SRR8735410–12 (BioProject PRJNA527373). Then BLAST the Bellerophon-filtered contigs against the reference transcript set with E-value <= 1e-3, percentage identity >= 90, and alignment length >= 300 nucleotides. Label a contig chimeric when it contains two non-overlapping mapping-region matches to different genes, nonchimeric when it matches exactly one gene, and unidentified when it matches no genes. Compare class counts between the original and filtered assemblies to compute removal percentages (e.g., 1619/2,673 chimeras = 60.6%, 14,325/26,528 correct = 54%, 3,005/5,183 unidentified = 58% on a 33,484-contig D. melanogaster assembly), and track the unique-gene count change (10,527 to 9,148).

## Related tools

- **Bellerophon** (Pipeline whose filtered output contigs are classified and tabulated; runs TPM filter (cut-off 1), CD-HIT-EST (95% identity), and TransRate filtering in default order) — github.com/JesseKerkvliet/Bellerophon
- **BLASTn** (Aligns filtered contigs against the reference transcripts (E-value <= 1e-3, identity >= 90%, alignment length >= 300 nt) to assign chimeric/nonchimeric/unidentified labels)
- **Trinity** (Produces the de novo transcriptome assembly from SRA reads (SRR8735410–12) that is then classified and filtered)
- **TransRate** (Establishes the ground quality score of the unfiltered assembly and applies the final TransRate filtering step)
- **CD-HIT-EST** (Removes highly identical redundant contigs at the default 95% identity cutoff prior to classification)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Reproduced removal percentages match the published benchmark: chimeras ~60.6% (1619/2,673), correct sequences ~54% (14,325/26,528), unidentified ~58% (3,005/5,183)
- Unique-gene count drops from 10,527 to 9,148 after filtering on the 33,484-contig D. melanogaster assembly
- Every contig receives exactly one of the three labels; chimeric contigs show two non-overlapping BLAST mapping regions with hits to different genes
- BLAST parameters are honored: E-value <= 1e-3, percentage identity >= 90, alignment length >= 300 nucleotides
- The filter removes proportionally more chimeric than nonchimeric contigs, consistent with the paper's stated design goal

## Limitations

- Bellerophon may erroneously remove contigs from alternative splicing that are misclassified as chimeras when expression is uneven along the sequence
- When there is no reference genome available, the quality of a de novo assembled transcriptome is difficult to assess, so classification quality depends on the completeness of the reference transcript set
- Custom order of filtering steps is disabled in the current Bellerophon version, so classification must be performed on the default TPM -> CD-HIT-EST -> TransRate-C output

## Evidence

- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [results] the TPM filtering step first removed 227 (8.5%) of the chimeras: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
- [readme] Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies.: "Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies."
- [results] The final Transrate‐C filtering step removed 792 (42.9%) chimeras: "The final Transrate‐C filtering step removed 792 (42.9%) chimeras"
