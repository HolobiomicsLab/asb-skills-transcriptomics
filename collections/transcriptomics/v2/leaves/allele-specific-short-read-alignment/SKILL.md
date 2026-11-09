---
name: allele-specific-short-read-alignment
description: Use when scRNA-Seq reads from an F1 hybrid cross (e.g. CAST×B6) must
  be assigned to maternal/paternal alleles for allele-specific expression counting,
  and multi-mapping reads should be retained rather than discarded.
when_to_use_negative:
- Input reads are not from a cross with known parental variants — there is no SNP/indel
  basis to build a phase-known diploid transcriptome.
- UMI-based counting where only unambiguous allelic assignment is acceptable and multi-read
  resolution is not desired downstream.
- The goal is whole-genome or splice-aware alignment rather than transcriptome-level
  allele tagging.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3170
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38
    reference genome to reconstruct the CAST/EiJ genome sequence.
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: 'Index builder and aligner: constructs the diploid transcriptome index and
    aligns single-end reads with ''-v 3 --all --best --strata'' to produce allele-tagged
    alignments retaining multi-reads.'
- name: EMASE
  role: Consumes the allele-specific alignments downstream, resolving multi-reads
    by weighted allocation via an EM algorithm for allele-specific counting.
- name: scBASE
  role: Pipeline that encapsulates this alignment stage and performs subsequent ASE
    classification and estimation.
  repo: https://github.com/churchill-lab/scBASE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:cd9874f0207772fdc7a8abc0f5760ed564d7e9527e2c0dc56a5388744f5e81d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# allele-specific-short-read-alignment

## Summary

Constructs a phase-known diploid (CAST/B6) transcriptome and aligns scRNA-Seq reads against it with bowtie so every read is tagged with parental-origin labels, retaining genomic and allelic multi-reads for downstream allele-specific expression analysis.

## When to use

Use when scRNA-Seq reads from an F1 hybrid cross (e.g. CAST×B6) must be assigned to maternal/paternal alleles for allele-specific expression counting, and multi-mapping reads should be retained rather than discarded.

## When NOT to use

- Input reads are not from a cross with known parental variants — there is no SNP/indel basis to build a phase-known diploid transcriptome.
- UMI-based counting where only unambiguous allelic assignment is acceptable and multi-read resolution is not desired downstream.
- The goal is whole-genome or splice-aware alignment rather than transcriptome-level allele tagging.

## Inputs

- Single-end scRNA-Seq FASTQ reads (e.g. Deng et al. 286 CAST×B6 F1 pre-implantation embryo cells)
- GRCm38 reference mouse genome sequence
- Known SNP and short indel variant calls (Sanger REL-1505)
- Gene transcript annotations (Ensembl Release 78)

## Outputs

- Diploid (CAST+B6) transcriptome FASTA with maternal/paternal copy labels
- bowtie index over the diploid transcriptome
- Allele-specific alignment files with reads tagged by parental origin

## How to apply

First reconstruct the CAST/EiJ genome by incorporating known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference using g2gtools, then build a CAST-specific transcriptome and concatenate it with the B6 transcriptome (Ensembl Release 78) into a single diploid transcriptome FASTA carrying maternal/paternal copy labels. Build a bowtie (v1.0.0) index on this diploid FASTA. Align single-end reads (e.g. Deng et al. embryo cells) with bowtie allowing up to three mismatches ('-v 3') and reporting all best-scoring alignments ('--all --best --strata') so genomic and allelic multi-reads are preserved — this matters because discarding multi-mapping reads inflates variability in allelic proportions and produces spurious monoallelic calls. Emit the parental-origin-tagged alignments as alignment files for allele-specific counting (e.g. via the EM-based weighted allocation in EMASE).

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference genome to reconstruct the CAST/EiJ genome sequence.) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Index builder and aligner: constructs the diploid transcriptome index and aligns single-end reads with '-v 3 --all --best --strata' to produce allele-tagged alignments retaining multi-reads.)
- **EMASE** (Consumes the allele-specific alignments downstream, resolving multi-reads by weighted allocation via an EM algorithm for allele-specific counting.)
- **scBASE** (Pipeline that encapsulates this alignment stage and performs subsequent ASE classification and estimation.) — https://github.com/churchill-lab/scBASE

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome_index reads.fq > allele_specific_alignments.sam
```

## Evaluation signals

- The bowtie index builds successfully over the diploid transcriptome FASTA.
- Alignment output files exist for the Deng et al. cells with reads aligned under the '-v 3 --all --best --strata' parameterization.
- Alignments carry parental-origin tags (maternal/paternal labels) from the diploid transcriptome.
- Multi-mapping reads (genomic and allelic) are present in the output rather than filtered, consistent with the '--all' reporting mode.
- Variant incorporation can be verified by checking that known CAST SNPs/indels appear in the reconstructed transcriptome sequences.

## Limitations

- Discarding multi-mapping reads instead of retaining them leads to higher variability in allelic proportion estimates, more sampling zeros, and spurious findings of dynamic and monoallelic expression.
- The approach is specific to strains with dense, known variant calls (here CAST/EiJ vs B6 on GRCm38); phase-known diploid reconstruction depends on the Sanger REL-1505 variant set.
- bowtie alignment to a transcriptome ignores genomic intron structure and allows up to only three mismatches ('-v 3'), which may affect reads spanning exon junctions or with higher error rates.

## Evidence

- [discussion] We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome: "We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome"
- [discussion] We aligned reads using bowtie with parameters '–all', '–best', and '–strata', allowing for three mismatches ('-v 3'): [evidence span withheld — anchor status: not_found]
- [discussion] We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [abstract] discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros: "discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros"
- [discussion] multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
