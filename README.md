# meep

Pipeline for human data depletion and bacterial whole-genome sequencing (WGS) based on Oxford Nanopore Technology (ONT) long-read sequencing.

## Pipeline overview

| Step | Rule | Tool | Container |
|------|------|------|-----------|
| 1 | `nanostat_raw` | NanoStat | `longread-env` |
| 2 | `human_depletion_minimap2` | minimap2 + samtools | `longread-env` |
| 3 | `porechop` | Porechop | `longread-env` |
| 4 | `nanostat_clean` | NanoStat | `longread-env` |
| 5 | `filtlong` | Filtlong | `longread-env` |
| 6 | `flye` | Flye | `longread-env` |
| 7 | `quast` | QUAST | `assembly-tools` |
| 8 | `medaka` | Medaka | `medaka` |
| 9 | `checkm2` | CheckM2 | `checkm2` |
| 10 | `multiqc` | MultiQC | `assembly-tools` |

## Container images

All pipeline steps run inside Docker/Apptainer (Singularity) containers hosted on the GitHub Container Registry:

| Image | Tools |
|-------|-------|
| `ghcr.io/tts577/meep/longread-env:latest` | minimap2, samtools, porechop, filtlong, flye, nanostat |
| `ghcr.io/tts577/meep/assembly-tools:latest` | QUAST, MultiQC |
| `ghcr.io/tts577/meep/medaka:latest` | Medaka 1.11.3 |
| `ghcr.io/tts577/meep/checkm2:latest` | CheckM2 |

Images are built automatically via GitHub Actions (`.github/workflows/build-containers.yml`) whenever the `envs/` or `containers/` directories change.

## Requirements

- [Snakemake](https://snakemake.readthedocs.io) ≥ 8
- [Apptainer / Singularity](https://apptainer.org) (for container execution)

## Setup

### 1. Edit the sample sheet

Fill in `config/samples.tsv` with your sample names and paths to raw FASTQ files:

```tsv
sample	long_reads
sample1	/path/to/sample1_reads.fastq.gz
sample2	/path/to/sample2_reads.fastq.gz
```

### 2. Edit the configuration

Adjust `config/config.yaml` to set output paths, filtlong thresholds, Flye genome size, Medaka model, and CheckM2 database path.

### 3. Provide reference resources

Place the following files at the paths configured in `config/config.yaml` (defaults shown):

- `meep_pipeline/resources/GRCh38.mmi` — minimap2 index of the human reference genome
- `meep_pipeline/resources/checkm2_db/uniref100.KO.1.dmnd` — CheckM2 diamond database

## Running the pipeline

```bash
# Dry-run to verify the workflow
snakemake --use-apptainer --cores <N> -n

# Full run
snakemake --use-apptainer --cores <N>
```

> **Note:** On older Snakemake 7.x installations use `--use-singularity` instead of `--use-apptainer`.

## Building containers locally

All Dockerfiles use the repository root as the build context so that the `envs/` YAML files are accessible:

```bash
docker build -f containers/longread_env/Dockerfile   -t meep-longread-env  .
docker build -f containers/assembly_tools/Dockerfile -t meep-assembly-tools .
docker build -f containers/medaka/Dockerfile         -t meep-medaka        .
docker build -f containers/checkm2/Dockerfile        -t meep-checkm2       .
```
