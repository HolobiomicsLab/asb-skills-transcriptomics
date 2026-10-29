---
name: genetic-meta-marker-ground-truth-annotation
description: 'Use when benchmarking higher-order genetic meta-marker miners: you need
  synthetic SNP data (n=3000 samples, per-gene SNP counts U[3,10]) where the true
  associated connected subgraph of the gene-interaction prior is known.'
when_to_use_negative:
- The data are real experimental GWAS samples (e.g. the athaliana avrRpm1 or mmusculus
  BMC.Mode datasets) — no ground-truth meta-marker exists there, so annotation would
  be fabricated
- The goal is running HOGImine for discovery rather than validating a miner against
  known truth; then only the HOGImine file-format skill is needed
- A FastCMH-style single-interval benchmark is required — that setting uses one significant
  interval plus a confounded interval, not a connected multi-gene meta-marker
tools:
- name: HOGImine
  role: Target algorithm and format definition; the six generated files must parse
    as valid HOGImine inputs and are consumed by HOGImine for validation
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: 'Smoke-test binary: run hogimine_additive on the synthetic data with -f 0.05
    to confirm the generated files are usable'
  repo: https://github.com/BorgwardtLab/HOGImine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:5d1a386c0901b675cdf0f3d1b7d803fcc97d7a8b893eb4a7f5ee9022087d4f1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# genetic-meta-marker-ground-truth-annotation

## Summary

Generates synthetic GWAS datasets in HOGImine format with a planted, fully recorded ground-truth meta-marker (a connected gene subgraph with tunable association strength rho), enabling power and FWER validation of meta-marker discovery algorithms such as HOGImine.

## When to use

Use when benchmarking higher-order genetic meta-marker miners: you need synthetic SNP data (n=3000 samples, per-gene SNP counts U[3,10]) where the true associated connected subgraph of the gene-interaction prior is known.

## When NOT to use

- The data are real experimental GWAS samples (e.g. the athaliana avrRpm1 or mmusculus BMC.Mode datasets) — no ground-truth meta-marker exists there, so annotation would be fabricated
- The goal is running HOGImine for discovery rather than validating a miner against known truth; then only the HOGImine file-format skill is needed
- A FastCMH-style single-interval benchmark is required — that setting uses one significant interval plus a confounded interval, not a connected multi-gene meta-marker

## Inputs

- Random seed / parameters: n=75 nodes, m=100 edges for the Erdos-Renyi gene-interaction graph
- Per-gene SNP count range U[3,10] and sample size n=3000
- Encoding choice: binary and/or additive, with per-SNP dominant/recessive annotation
- Tunable association strength rho for the planted causal subgraph
- Number of covariate classes C for the integer covariate column

## Outputs

- Marker matrix file for '-i': n.SNPs rows x 3000 sample columns, binary or additive values
- Binary labels file for '-l': n.samples x 1
- SNP names file for '-s'
- Covariates file for '-c': integer classes in {1,...,C}
- SNP map file for '-m': gene name followed by its mapped SNP names
- Gene interaction sets file for '-e': the 75-node/100-edge Erdos-Renyi prior
- Ground-truth file recording the genes, encodings, and SNP positions of the planted meta-marker

## How to apply

First sample an Erdos-Renyi G(n,m) graph with 75 nodes and exactly 100 edges (e.g. networkx.gnm_random_graph) as the gene-interaction prior written to the '-e' interactions file. Assign each gene a number of SNPs drawn from U[3,10], simulate genotypes for 3000 samples in binary and/or additive encoding (with per-SNP dominant/recessive annotation), plus a binary phenotype label column and an integer covariate column in {1,...,C}. Sample a random connected subgraph of the network as the causal module and inject phenotype association with tunable strength rho into the per-gene SNP intervals it spans. Serialize six HOGImine-format files — marker matrix '-i' (n.SNPs rows x 3000 sample columns), binary labels '-l' (n.samples x 1), SNP names '-s', covariates '-c', SNP map '-m' (gene name followed by its mapped SNPs), interactions '-e' — and additionally write a ground-truth file recording the genes, encodings, and SNP positions of the planted meta-marker, which is what distinguishes this skill from plain data formatting.

## Related tools

- **HOGImine** (Target algorithm and format definition; the six generated files must parse as valid HOGImine inputs and are consumed by HOGImine for validation) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Smoke-test binary: run hogimine_additive on the synthetic data with -f 0.05 to confirm the generated files are usable) — https://github.com/BorgwardtLab/HOGImine

## Examples

```
./src/hogimine_additive -i data/synthetic/synth_X.txt -l data/synthetic/synth_y_bin.txt -c data/synthetic/synth_cov.txt -m data/synthetic/synth_map.txt -e data/synthetic/synth_interactions.txt -f 0.05
```

## Evaluation signals

- All six generated files parse as valid HOGImine inputs
- Marker matrix '-i' has exactly 3000 sample columns with binary/additive values
- Labels file '-l' is binary and of size n.samples x 1
- Gene-interaction file '-e' contains a connected 75-node, 100-edge graph and the planted causal module forms a connected subgraph in it
- A smoke run of hogimine_additive on the synthetic data with -f 0.05 executes without error, and the ground-truth file reproduces the exact genes, encodings, and SNP positions of the injected meta-marker

## Limitations

- HOGImine accepts only binary and additive encodings, so the generator must restrict per-SNP annotations (dominant/recessive) to those two encodings
- Ground truth is only 'planted' truth: a small random connected subgraph holds the significant association, so any real (unplanted) signal in the synthetic data is noise by construction
- Synthetic parameters (3000 samples, 75 genes, 100 edges, U[3,10] SNPs/gene) are fixed by the Section 4.1.1 simulation design and may not transfer to other regimes

## Evidence

- [readme] HOGImine accepts both binary encodings and additive encodings for the markers.: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [readme] Marker file format: "```-i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$, i.e. with each column being a sample."
- [results] Synthetic ground truth design: [evidence span withheld — anchor status: not_found]
- [abstract] HOGImine scope for planted meta-markers: "We propose HOGImine, a novel algorithm that expands the class of discoverable genetic meta-markers by considering higher-order interactions of genes"
