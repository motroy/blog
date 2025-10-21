## AllTheBacteria.org: A Curated Resource for Bacterial & Archaeal Genomics
Date: 21 Oct 2025

## Introduction
Public sequencing projects have generated an avalanche of raw data—much of it unassembled, inconsistently processed, and difficult to reuse. AllTheBacteria.org solves this problem by delivering a uniformly assembled, annotated, and quality‑controlled collection of ≈2 million bacterial and archaeal genomes. Whether you need to find the closest relative of a new isolate, build a phylogenetic tree, or trace plasmid‑borne antimicrobial‑resistance (AMR) genes, this resource gives you a solid foundation for large‑scale comparative genomics.

## 1. What Is AllTheBacteria.org?

| Feature                     | Why It Matters |
|-----------------------------|----------------|
| **Uniformly processed genomes** | Eliminates batch effects; every assembly follows the same pipeline. |
| **Standardized annotations**    | Consistent gene calls, GTDB‑based taxonomy, and AMR detection (AMRFinderPlus). |
| **Searchable indexes**          | Pre‑built sketch libraries (`sketchlib`, `sourmash`) let you locate similar genomes instantly. |

## 2. How to Use the Data
### a. As a skani‑style reference database
you can use sketchlib (https://github.com/bacpop/sketchlib.rust) to search against the ATB, AllTheBacteria.org supplies ready‑made indexes for microbial genomes (https://ftp.ebi.ac.uk/pub/databases/AllTheBacteria/Releases/0.2/indexes/sketchlib/).
```
#The distributed index is sketch size 1024 with k=17. You can create different ranges using the sketch subcommand.

# 1. To calculate distances of a subset of the data, use a command such as:
sketchlib dist -v -k 17 --subset Haemophilus_influenzae.txt --ani atb_sketchlib_v020 --threads 4 > dists.txt
# Where the --subset file contains the list of samples you want to include.
# Removing --ani will calculate Jaccard distances.

# 2. To query new samples against the index, first sketch it:

sketchlib sketch -v -o query -k 17 -f queries.tsv -s 1000
#(where queries.tsv contains query samples with name and file location, tab separated)

# 3. Then query it
sketchlib dist -v -k 17 atb_sketchlib_v020 query
```
Result: Rapidly retrieve the closest matches among millions of genomes.

### b. Extract specific genes for Multiple‑Sequence Alignments (MSA)
Because every genome carries the same annotation schema (see details of how to retrieve Bakta annotations for ATB genomes of interest: https://github.com/AllTheBacteria/AllTheBacteria/issues/40), pulling out a gene of interest (e.g., gyrB, recA, or an AMR determinant) is straightforward:
```
# Pseudocode – adapt to your preferred language/tool
for gff in all_the_bacteria/*.gff; do
    grep -w "gene_name_of_interest" "$gff" >> gene_collection.faa
done
mafft --auto gene_collection.faa > gene_alignment.faa
You can then perform phylogenetics, allele‑typing, or functional comparisons across thousands of strains.
```
### c. Build phylogenetic trees to discover clusters
Leverage the GTDB taxonomy embedded in each record, or construct your own tree from concatenated core genes. The dataset’s breadth lets you place a novel isolate within a global evolutionary context and spot emerging clades.

### d. Plasmid and mobile element investigations
AllTheBacteria.org includes plasmid contigs alongside chromosomal assemblies, all processed with identical QC standards. You can:
- Track the distribution of a particular plasmid backbone across taxa.
- Map the acquisition of AMR genes onto plasmid phylogenies.
- Identify “super‑plasmids” that have acted as hubs for resistance gene exchange.

## 3. Downloading the Dataset
You do not need an OSF account—the data are publicly available through several mirrors.

| Method                           | Command / Link                             | Tips |
|----------------------------------|--------------------------------------------|------|
| **AWS S3 (most flexible)** | `aws s3 sync --no-sign-request s3://allthebacteria-assemblies/ ./ATB/` | Works without AWS credentials; use `--exclude/--include` to fetch only what you need (e.g., `*/genomes/`). |
| **Open Science Framework (OSF)** | Browse the project UI and click “Download”. | Handy for occasional, small‑scale pulls. |
| **ENA FTP** | `https://ftp.ebi.ac.uk/pub/databases/AllTheBacteria/Releases/0.2/` | Mirrors the S3 bucket; use any FTP client. |

Tip: If storage is a concern, sync only the annotation files (*.gff, *.faa) or the specific taxonomic groups you plan to analyse.

## 4. Real‑World Example: Tracing Plasmid‑Mediated AMR
- A recent Science paper (Cazares et al., 2025, DOI: 10.1126/science.adr1522) leveraged a massive plasmid collection—very similar to AllTheBacteria.org—to reconstruct the evolution of multidrug‑resistant (MDR) plasmids.

### Workflow Overview
- Screen > 40 k plasmids for AMR genes using AMRFinderPlus.
- Cluster plasmids by backbone similarity (Mash distance < 0.05).
- Build a plasmid phylogeny and overlay AMR presence/absence.
- Identify fusion events that generated “super‑plasmids” driving global MDR spread.

### How AllTheBacteria.org Helps
* Uniform assemblies ensure plasmid backbones are comparable across decades.
* Consistent AMR annotation removes the need to rerun detection pipelines on each download.
* Massive sample size (millions of genomes, tens of thousands of plasmids) provides the statistical power to detect rare evolutionary events.
* By reproducing this pipeline with AllTheBacteria.org, you can explore similar questions in any taxonomic group—Enterobacteriaceae plasmids, Streptococcus phage‑derived elements, or environmental Archaea mobilome dynamics.

## 5. Quick‑Start: Running Bactopia on AllTheBacteria Assemblies
If you prefer an end‑to‑end workflow, the Bactopia suite integrates smoothly with the ATB data. Below is a minimal example using Legionella pneumophila assemblies.
```
# 1️⃣ Download a subset (e.g., Legionella pneumophila)
aws s3 cp --no-sign-request \
    s3://allthebacteria-assemblies/Legionella_pneumophila/ ./legionella_raw/ \
    --recursive

# 2️⃣ Convert to Bactopia layout (creates symlinks, saves space)
bactopia atb-formatter \
    -i ./legionella_raw/ \
    -o ./legionella_bactopia/

# 3️⃣ Run a Bactopia tool (e.g., legsta typing)
bactopia \
    -i ./legionella_bactopia/ \
    --wf legsta \
    -o ./legionella_results/
```
The legsta report can be merged across all isolates to produce a population‑wide serogroup table—ideal for epidemiological surveillance.

## 6. Key Take‑aways
* AllTheBacteria.org turns a chaotic mass of raw sequencing data into a clean, searchable library of millions of microbial genomes.
* It supports sketch‑based searches, gene‑level MSAs, large‑scale phylogenetics, and plasmid/AMR investigations.
* Downloading is straightforward via AWS S3, OSF, or ENA FTP—no registration required.
* The dataset has already powered high‑impact research (e.g., the Science plasmid‑AMR study) and integrates seamlessly with pipelines like Bactopia.
* Whether you’re a bioinformatician building a new comparative pipeline or a microbiologist hunting for the nearest neighbor of a clinical isolate, AllTheBacteria.org offers a ready‑made, high‑quality foundation for your next discovery.

Happy analysing!

# References

1. Cazares, A., Figueroa, W., Cazares, D., et al. (2025). Pre and Post antibiotic epoch: the historical spread of antimicrobial resistance. Science. DOI: https://doi.org/10.1126/science.adr1522
2. AllTheBacteria.org – Official Repository: https://github.com/AllTheBacteria/AllTheBacteria
3. Sketch indexes README: https://ftp.ebi.ac.uk/pub/databases/AllTheBacteria/Releases/0.2/indexes/README.md
4. Sourmash issue discussing ATB integration: https://github.com/sourmash-bio/sourmash/issues/3247
5. Bactopia ATB formatter documentation: https://github.com/Pathogen-Genomics/bactopia/wiki/ATB-Formatter
(All links are live as of 21 Oct 2025.)
