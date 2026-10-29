---
name: genomic-interval-sequence-extraction
description: Use when converting tabix-indexed REDItools outTables into per-site 8×101
  feature matrices for TCN-based editing classification, e.g. filtering candidates
  (cov≥50, ≥3 Gs, A-to-G freq≥0.01) and pulling flanking sequence from a faidx-indexed
  genome.
when_to_use_negative:
- The input is already an encoded feature_vectors.tsv — this skill produces, not consumes,
  that format.
- Only raw RNAseq BAM files are available — REDItools (or REDItools3) must first generate
  and tabix-index the outTable of RNAseq pileups before window extraction.
- Candidate sites lack a fully covered 101-nt window and no imputation is acceptable
  — the light-version workflow discards incomplete intervals rather than encoding
  them.
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3316
tools:
- name: REDInet
  role: Implements the filtering, 101-nt window extraction, and 8×101 feature encoding
    (REDInet_Inference.py / REDInet_Inference_light_ver.py)
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: REDItools
  role: Produces the per-position RNAseq pileup outTable that supplies the candidate
    sites and nucleotide frequencies for window extraction
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: pysam
  role: Python library used to read the tabix-indexed outTable and query the faidx-indexed
    reference genome for interval sequences
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: faidx-indexes the GRCh37/GRCh38 reference FASTA so 101-nt windows can be retrieved
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:88aaa10cabc5e1c55fe9b10c8bd3d267b2a04369f421e98df318c7c8044c4f71
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet.git
  tools:
  - Python
  - REDInet
  - REDItools
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# genomic-interval-sequence-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts 101-nucleotide reference windows (±50 bases) centered on candidate A-to-I RNA editing sites from GRCh37/GRCh38 and encodes them as 8×101 feature matrices combining reference-base one-hot channels with log2-transformed normalized RNAseq nucleotide pileup frequencies, as implemented in the REDInet pipeline.

## When to use

Use when converting tabix-indexed REDItools outTables into per-site 8×101 feature matrices for TCN-based editing classification, e.g. filtering candidates (cov≥50, ≥3 Gs, A-to-G freq≥0.01) and pulling flanking sequence from a faidx-indexed genome.

## When NOT to use

- The input is already an encoded feature_vectors.tsv — this skill produces, not consumes, that format.
- Only raw RNAseq BAM files are available — REDItools (or REDItools3) must first generate and tabix-index the outTable of RNAseq pileups before window extraction.
- Candidate sites lack a fully covered 101-nt window and no imputation is acceptable — the light-version workflow discards incomplete intervals rather than encoding them.

## Inputs

- bgzip-compressed REDItools outTable (.gz) indexed with tabix -s 1 -b 2 -e 2 -c R
- faidx-indexed reference genome FASTA (GRCh37 or GRCh38 primary assembly, e.g. GRCh38.primary_assembly.genome.fa)
- Filter thresholds: minimum coverage (--C, e.g. 50), minimum A-to-G count (--M, e.g. 3), minimum A-to-G frequency (--F, e.g. 0.01)

## Outputs

- <output_files_prefix>.feature_vectors.tsv containing one 8×101 feature matrix per site with a complete 101-nt interval
- <prefix>_discarded.txt listing positions failing coverage/substitution filters
- <prefix>_incompleates.txt logging excluded positions as '0 *' (missing nucleotides) or '0 **' (strand-discordant nucleotides)

## How to apply

Load the bgzip-compressed, tabix-indexed REDItools outTable (indexed with 'tabix -s 1 -b 2 -e 2 -c R') using Python with pysam/pandas. Filter candidate sites with REDInet defaults — minimum RNAseq coverage of 50 (--C/-c), minimum of 3 guanosines substituting adenines (--M/-s), and minimum A-to-G substitution frequency of 0.01 (--F/-f) — recording excluded positions in <prefix>_discarded.txt. For each retained site, extract the 101-nucleotide window (50 upstream, 50 downstream, centered on the putative position) from the GRCh37 or GRCh38 reference genome (faidx-indexed via samtools), discarding intervals with missing nucleotides ('0 *') or strand-discordant bases ('0 **'), which are logged in <prefix>_incompleates.txt. Encode each window as an 8×101 matrix: 4 one-hot channels for the reference base (A, C, G, T) plus 4 channels of log2-transformed normalized RNAseq nucleotide pileup frequencies computed with an alpha=0.0001 pseudocount. Write encoded vectors for sites with complete intervals to <output_files_prefix>.feature_vectors.tsv.

## Related tools

- **REDInet** (Implements the filtering, 101-nt window extraction, and 8×101 feature encoding (REDInet_Inference.py / REDInet_Inference_light_ver.py)) — https://github.com/BioinfoUNIBA/REDInet.git
- **REDItools** (Produces the per-position RNAseq pileup outTable that supplies the candidate sites and nucleotide frequencies for window extraction) — https://github.com/BioinfoUNIBA/REDItools
- **pysam** (Python library used to read the tabix-indexed outTable and query the faidx-indexed reference genome for interval sequences) — https://github.com/pysam-developers/pysam
- **samtools** (faidx-indexes the GRCh37/GRCh38 reference FASTA so 101-nt windows can be retrieved)

## Examples

```
python3 ../REDInet/Package/Utilities/REDInet_Inference_light_ver.py -r outTable_SRR12492045.SRR12492046.gz -o ./results/REDInet -c 50 -s 3 -f 0.01
```

## Evaluation signals

- Output TSV exists and every feature vector corresponds to a complete 101-nt interval rendered as an 8×101 matrix.
- Only sites passing the coverage/substitution filters (cov≥50, ≥3 G substitutions, A-to-G frequency≥0.01) appear in the output.
- Excluded positions are fully accounted for: <prefix>_discarded.txt (filter failures) and <prefix>_incompleates.txt ('0 *' missing / '0 **' strand-discordant) plus retained sites equal the input site list.
- One-hot reference-base channels are valid 4-of-4 encodings per position, and the 4 RNAseq channels contain log2-transformed normalized frequencies consistent with an alpha=0.0001 pseudocount.
- The outTable was compressed with bgzip and indexed with 'tabix -s 1 -b 2 -e 2 -c R'; the reference FASTA has a matching .fai index.

## Limitations

- Intervals with missing or strand-discordant nucleotides are discarded; imputation of missing values is only available in the multiprocessing REDInet_Inference.py version and is explicitly a BETA feature.
- Requires prior bgzip + tabix ('-c R') indexing of the REDItools outTable and faidx-indexed reference genomes; unindexed inputs cannot be queried.
- Windows must lie within the reference assembly; positions near contig edges or with insufficient flanking coverage are excluded, biasing the site set toward well-covered regions.

## Evidence

- [other] gap-free intervals of 50 nucleotides flanking each putative (or non-putative) editing site supported by RNAseq reads were extracted and encoded as 8 x 101 feature matrices: [evidence span withheld — anchor status: not_found]
