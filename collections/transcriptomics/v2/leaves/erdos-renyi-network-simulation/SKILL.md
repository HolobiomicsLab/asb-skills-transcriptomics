---
name: erdos-renyi-network-simulation
description: Use when you need synthetic genotype-phenotype data with known planted
  higher-order (multi-gene) associations to benchmark network-guided interval miners
  such as HOGImine against their statistical power and runtime.
when_to_use_negative:
- The dataset is real (non-simulated) genotype data — this skill only creates synthetic
  benchmarks and cannot recover ground truth from real cohorts
- You need a synthetic benchmark for covariate-adjusted single-interval methods like
  FastCMH — its R generator makefastcmhdata has a different format and only supports
  K=2 covariate classes
- You only need to run HOGImine on an existing dataset (e.g. data/mmusculus/BMC.Mode
  or data/athaliana) rather than generate new synthetic data
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HOGImine
  role: Target file format the synthetic data must serialize into; HOGImine options
    (-i, -l, -s, -c, -m, -e) define the six output files
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: 'Smoke-run validator: executing hogimine_additive on the generated marker/label
    files with -f 0.05 confirms the synthetic data parse as valid HOGImine inputs'
  repo: https://github.com/BorgwardtLab/HOGImine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:295da9eb70aeed0d6d90dbb282fc23596a9b0fefd16fbbda89b28fd9759139e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# erdos-renyi-network-simulation

## Summary

Generates synthetic GWAS datasets with a planted, known ground-truth higher-order association: an Erdos-Renyi gene-interaction network, per-gene SNP intervals, and a causal connected subgraph injected with tunable strength rho, serialized in HOGImine input format.

## When to use

Use when you need synthetic genotype-phenotype data with known planted higher-order (multi-gene) associations to benchmark network-guided interval miners such as HOGImine against their statistical power and runtime.

## When NOT to use

- The dataset is real (non-simulated) genotype data — this skill only creates synthetic benchmarks and cannot recover ground truth from real cohorts
- You need a synthetic benchmark for covariate-adjusted single-interval methods like FastCMH — its R generator makefastcmhdata has a different format and only supports K=2 covariate classes
- You only need to run HOGImine on an existing dataset (e.g. data/mmusculus/BMC.Mode or data/athaliana) rather than generate new synthetic data

## Inputs

- Random seed / generator parameters (n=75 nodes, m=100 edges, SNP count U[3,10] per gene, 3000 samples, covariate classes C, signal strength rho)
- Choice of genotype encoding: binary (dominant/recessive per SNP) and/or additive

## Outputs

- Marker matrix file (-i): n.SNPs rows x 3000 sample columns with binary/additive values
- Binary labels file (-l): n.samples x 1
- SNP names file (-s)
- Covariate file (-c): integer class per sample in {1,...,C}
- SNP map file (-m): gene name followed by its mapped SNPs
- Gene interaction sets file (-e): the Erdos-Renyi network edges
- Ground-truth file listing genes, encodings, and SNP positions of the planted meta-marker

## How to apply

Sample an Erdos-Renyi G(n,m) random graph with 75 nodes and exactly 100 edges (e.g. networkx gnm_random_graph) as the gene-interaction prior. Assign each gene a number of SNPs drawn from U[3,10] and simulate genotypes for 3000 samples in binary and/or additive encoding, with per-SNP dominant/recessive annotation, a binary phenotype label column, and an integer covariate column in {1,...,C}. Pick a random connected subgraph of the network as the causal module and inject phenotype association with tunable strength rho into the per-gene SNP intervals it spans. Serialize six HOGImine-format files: marker matrix (-i, n.SNPs x 3000 samples), binary labels (-l), SNP names (-s), covariates (-c), SNP-to-gene map (-m), gene interaction sets (-e), plus a ground-truth file recording the genes, encodings, and SNP positions of the planted meta-marker. Validate by parsing all files as HOGImine inputs and running a smoke run of hogimine_additive with -f 0.05.

## Related tools

- **HOGImine** (Target file format the synthetic data must serialize into; HOGImine options (-i, -l, -s, -c, -m, -e) define the six output files) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Smoke-run validator: executing hogimine_additive on the generated marker/label files with -f 0.05 confirms the synthetic data parse as valid HOGImine inputs) — https://github.com/BorgwardtLab/HOGImine

## Examples

```
import networkx as nx; G = nx.gnm_random_graph(75, 100, seed=42)  # gene-interaction prior, then inject rho into a random connected subgraph and serialize as HOGImine -i/-l/-s/-c/-m/-e files
```

## Evaluation signals

- All six generated files (-i, -l, -s, -c, -m, -e) parse as valid HOGImine inputs
- Marker matrix has exactly 3000 sample columns with binary/additive values
- Labels file is binary with shape n.samples x 1
- Gene interaction file corresponds to a G(75,100) Erdos-Renyi graph with exactly 100 edges
- A smoke run of hogimine_additive on the synthetic data with -f 0.05 executes without error, and the ground-truth file records the planted meta-marker's genes, encodings, and SNP positions

## Limitations

- The planted signal exists only in a small random connected subgraph; the rest of the simulated genome carries no true association by construction
- Ground truth is only meaningful for the simulated setting — a discovery on real data has no planted-meta-marker file to check against
- Per-gene SNP counts are drawn from U[3,10], so the generator does not model genes with very large SNP intervals

## Evidence

- [results] In the generated synthetic data, a small random connected subgraph has a truly statistically significant association with the phenotype.: [evidence span withheld — anchor status: not_found]
- [intro] HOGImine accepts both binary encodings and additive encodings for the markers.: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [readme] marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$, i.e. with each column being a sample.: "- -i file: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$, i.e. with each column being a sample."
