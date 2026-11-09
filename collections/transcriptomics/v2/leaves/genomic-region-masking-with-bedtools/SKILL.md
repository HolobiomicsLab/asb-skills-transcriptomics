---
name: genomic-region-masking-with-bedtools
description: Use when building a reference for tncRNA (tRF/tRNA half) detection where
  reads mapping to non-tRNA loci would otherwise be misattributed, and tRNA regions
  plus 50 nt flanks must be masked before adding artificial tRNA chromosomes.
when_to_use_negative:
- Input is already an artificial genome or Bowtie index built with the tncRNA-Toolkit
  — re-masking would corrupt the existing index.
- Genome FASTA headers do not start with '>chr[Num]' (e.g. >chr1, >chrMt, >chrPt),
  so tncRNA-Toolkit automation cannot separate nuclear and organellar regions.
- The goal is generic repeat or gene masking (e.g. RepeatMasker-style soft-masking)
  rather than building tRNA-aware artificial chromosomes for tncRNA read classification.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0099
tools:
- name: bedtools
  role: maskfasta script is used to mask the filtered tRNA regions plus 50 bp 3' and
    5' flanks in the reference genome
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Bowtie (v1.3)
  role: bowtie-build creates the Bowtie index for the artificial genome (masked genome
    + tRNA-derived artificial chromosomes)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE (v2.0.6)
  role: predicts the nuclear (eukaryotes model) and organellar ('-O' option) tRNA
    genes whose regions define the mask
- name: tncRNA Toolkit
  role: workflow wrapper that orchestrates tRNA prediction, masking, index creation
    (python3 tncRNAs.py -g <genome fasta> -s <species name>) and downstream tncRNA
    calling
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:de7fff10b57af2373bcbad45ab6564333e48e3c968040e590b0387381b7cfdac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie (v1.3)
  - bedtools
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-region-masking-with-bedtools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Mask genuine tRNA gene regions plus 50 bp of 3' and 5' flanking sequence in a reference genome FASTA using bedtools maskfasta, then concatenate mature/leader/trailer tRNA FASTAs as artificial chromosomes to build a Bowtie index that resolves ambiguous mapping of tRNA-derived small RNA reads.

## When to use

Use when building a reference for tncRNA (tRF/tRNA half) detection where reads mapping to non-tRNA loci would otherwise be misattributed, and tRNA regions plus 50 nt flanks must be masked before adding artificial tRNA chromosomes.

## When NOT to use

- Input is already an artificial genome or Bowtie index built with the tncRNA-Toolkit — re-masking would corrupt the existing index.
- Genome FASTA headers do not start with '>chr[Num]' (e.g. >chr1, >chrMt, >chrPt), so tncRNA-Toolkit automation cannot separate nuclear and organellar regions.
- The goal is generic repeat or gene masking (e.g. RepeatMasker-style soft-masking) rather than building tRNA-aware artificial chromosomes for tncRNA read classification.

## Inputs

- Reference genome FASTA (e.g. TAIR10.1 for Arabidopsis thaliana; SL4.0 nuclear and SL3.0 organellar for tomato)
- Filtered tRNAscan-SE prediction set (pseudogenes removed, Covels score >= 50; eukaryote model, '-O' option for organellar tRNAs)
- BED file of tRNA gene regions plus 50 bp upstream and downstream flanks
- Three tRNA FASTA files: mature tRNA (CCA-added, intron-removed), 50 bp 5' leader flanks, 50 bp 3' trailer flanks

## Outputs

- Masked reference genome FASTA (tRNA regions + 50 nt flanks masked)
- Artificial genome FASTA (masked chromosomes plus additional tRNA-derived chromosomes with 'chr'-prefixed headers, incl. chrMt/chrPt)
- Bowtie1 index files (artificial_genome.*.ebwt) under lib/indexes/<species name>

## How to apply

First filter the tRNAscan-SE (v2.0.6) predictions by removing pseudogenes and any tRNA with a Covels score below 50 (eukaryote model for nuclear tRNAs, '-O' for organellar tRNAs). Prepare three FASTA files: the mature tRNA (intron removed, CCA added at the 3' terminus), a 50 bp 5' leader flank, and a 50 bp 3' trailer flank. Run bedtools (v2.29) maskfasta to replace the filtered tRNA regions plus 50 bp on both sides with masked sequence in the reference genome. Concatenate the three tRNA FASTAs onto the masked genome as additional chromosomes — headers must begin with 'chr' (e.g. >chr1, with >chrMt and >chrPt for mitochondrial/plastid sequences) per the tncRNA-Toolkit requirements — and build the artificial genome index with bowtie-build (Bowtie v1.3), writing to lib/indexes/<species name>. Subsequent classification is valid only because a read is called a tncRNA when it maps exclusively to these artificial chromosomes, so the masking must be complete for every filtered tRNA.

## Related tools

- **bedtools** (maskfasta script is used to mask the filtered tRNA regions plus 50 bp 3' and 5' flanks in the reference genome) — https://github.com/skbinfo/tncRNA-Toolkit
- **Bowtie (v1.3)** (bowtie-build creates the Bowtie index for the artificial genome (masked genome + tRNA-derived artificial chromosomes)) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE (v2.0.6)** (predicts the nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes whose regions define the mask)
- **tncRNA Toolkit** (workflow wrapper that orchestrates tRNA prediction, masking, index creation (python3 tncRNAs.py -g <genome fasta> -s <species name>) and downstream tncRNA calling) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g <genome fasta> -s <species name>
```

## Evaluation signals

- Bowtie index files for the artificial genome (artificial_genome.*.ebwt) exist under lib/indexes/<species name>.
- The artificial genome FASTA contains the masked reference chromosomes plus the additional tRNA-derived chromosomes (headers beginning 'chr', with chrMt/chrPt for organellar sequences).
- Every filtered tRNA (pseudogenes removed, Covels score >= 50) is represented as a masked region plus a mature/leader/trailer artificial chromosome.
- Downstream mapping with bowtie '-v2 --best' shows reads classified as tncRNAs only when they map exclusively to the artificial chromosomes, with multi-mappers (>50 alignments, '-m 50') discarded.
- Mature tRNA FASTA entries have introns removed and CCA added at the 3' terminus.

## Limitations

- Reads are aligned with bowtie '-v2 --best'; permitting even a single mismatch to tolerate sequencing errors can cause base misidentification and inflate false negatives.
- Multi-mapped reads aligned more than 50 times are discarded, so highly repetitive tncRNAs may be lost.
- The pipeline requires genome FASTA headers to start with '>chr[Num]' (and >chrMt/>chrPt for organellar); other header conventions break the automated scripts.
- Experimental validation and characterization of predicted tncRNAs are still needed to distinguish bona fide tncRNAs; conventional RNA-seq suffers modification biases that tRNA-seq would mitigate.

## Evidence

- [methods] Masking of tRNA regions plus 50 bp flanks in reference genome: [evidence span withheld — anchor status: not_found]
