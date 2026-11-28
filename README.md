# Project Contributors

| Name                    | Student ID | GitHub username |
|-------------------------|------------|-----------------|
| Ricsi (Member 1)        | <s263490>  | <s263490>      |
| Daniel (Member 2)       | <sXXXXXX>  | <username>      |
| Konstantinos (Member 3) | s263498    | DizzyForest     |
| Sasa (Member 4)         | s263479    | b-sasa          |
| Marie (Member 5)        | s260367    | mariekvl        |

## Presentation

[Direct link to the presentation](https://raw.githack.com/%3Cgithub-org%3E/%3Crepo-name%3E/main/doc/presentation.html)

## Project Overview

**Group 12: Gene expression differences between healthy and cancerous breast tissue (GSE7904)**

Research question: which expression probes differ between breast tumour samples and normal breast tissue in GSE7904, and do overall expression profiles separate the two groups?

Short summary: \<2-3 sentences on data, methods and main finding. Fill in at the end.\>

## Data Retrieval

Data are **not** stored on GitHub. To reproduce the project:

1. Dataset: GEO accession [GSE7904](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE7904).
2. The `01_load.qmd` script automatically downloads the GEO Series Matrix file to `data/_raw/` if it is not already present.
3. The original downloaded file is preserved and should not be manually modified.

## Data Loading (`01_load.qmd`)

The first step of the analysis workflow is implemented in `R/01_load.qmd`, which retrieves the GSE7904 dataset from NCBI GEO, extracts the expression measurements and sample metadata, performs initial validation, and exports the structured data for downstream analysis.

### Input data

- **GEO accession:** GSE7904
- **Organism:** *Homo sapiens*
- **Platform:** GPL570 — Affymetrix Human Genome U133 Plus 2.0 Array
- **Original samples:** 62
- **Probe sets:** 54,675
- **Input file:** `data/_raw/GSE7904_series_matrix.txt.gz`

The script automatically downloads the compressed GEO Series Matrix file if it is not already present. It preserves the original file and creates the required data directories automatically.

### Data extraction and processing

The loading workflow:

1. Reads the compressed Series Matrix file using `gzfile()` and `readLines()`.
2. Extracts sample identifiers, titles, source descriptions, and sample characteristics from the GEO metadata.
3. Uses a reusable extraction function and `lapply()` to process multiple metadata fields.
4. Locates the expression table using the Series Matrix boundary markers and imports it with `read.delim()`.
5. Assigns biological tissue groups using `dplyr::mutate()` and `case_when()`.
6. Inspects dataset dimensions, sample composition, expression-value statistics, missing values, and duplicated probe identifiers.

### Biological sample groups

| Original GEO category | Assigned group | Number of samples |
|---|---|---:|
| Basal | Tumour | 18 |
| non-BLC | Tumour | 21 |
| BRCA1 | Tumour | 4 |
| NB | Normal breast | 7 |
| NO | Normal organelle | 12 |
| **Total** | | **62** |

The three tumour-related categories are assigned to one `Tumour` group. `Normal breast` and `Normal organelle` remain separate in the metadata. The primary downstream comparison focuses on tumour versus normal breast samples.

### Output files

| File | Description |
|---|---|
| `data/01_dat_load.tsv` | Expression measurements, with one row per probe set and one column per sample, plus the `ID_REF` identifier column. |
| `data/01_sample_metadata.tsv` | Sample-level metadata, including GEO characteristics and assigned tissue groups. |

### Validation and reproducibility

The script performs checks to verify:

- Successful download and non-empty input file.
- Expected expression matrix and metadata dimensions.
- Correspondence between expression sample identifiers and metadata identifiers.
- Duplicate probe identifiers.
- Missing expression measurements.
- Successful export and preservation of output dimensions and sample ordering.

An MD5 checksum is also calculated for the downloaded input file.

### Scope and limitations

This component focuses exclusively on data retrieval, extraction, organization, and initial validation.

The expression values are preserved as supplied in the GEO Series Matrix file (dChip signal values). No additional normalization, logarithmic transformation, probe-to-gene annotation, probe aggregation, or differential-expression analysis is performed at this stage.

The `data/` directory is ignored by Git, so the generated data files are not stored in the repository. They are recreated when the workflow is executed.

## How to Run the Project

1.  Clone the repository and open `group12_project.Rproj` in RStudio.

2.  Install the required packages:

    ``` r
    install.packages(c("tidyverse", "here", "broom", "quarto", "ragg", "tidyr", "ggplot2", "ggrepel", "dplyr", "vsn", "gridExtra"))
    # <add Bioconductor or other packages as they are used>
    ```

3. Run the workflow. Required data directories are created automatically, and the GEO dataset is downloaded if necessary.

4.  Retrieve the raw data as described above.

5.  Render the full workflow by rendering `R/00_all.qmd` (or in the console: `quarto::quarto_render("R/00_all.qmd")`).

6.  Output HTML files appear in `results/`.

Each `.qmd` file in `R/` can also be rendered on its own from a fresh R session.

## Repository Structure

```         
data/
  _raw/               raw data (never edited, not on GitHub)
  01_dat_load.tsv     loaded data
  02_dat_clean.tsv    cleaned data
  03_dat_aug.tsv      analysis-ready data
R/
  00_all.qmd          master document, runs the whole workflow
  01_load.qmd         load data
  02_clean.qmd        clean data and annotate probes
  03_augment.qmd      add derived variables
  04_describe.qmd     descriptive statistics and plots
  05_analysis_1.qmd   differential expression (tumour vs normal)
  06_analysis_2.qmd   PCA / clustering
  99_proj_func.<R or qmd>   shared helper functions
results/              HTML output and key plots
doc/                  presentation.qmd and presentation.html
```

## Who Did What

| File                                              | Lead         |
|---------------------------------------------------|--------------|
| `01_load.qmd`                                     | Ricsi        |
| `02_clean.qmd`                                    | Daniel       |
| `03_augment.qmd`, `04_describe.qmd`               | Konstantinos |
| `05_analysis_1.qmd`                               | Sasa         |
| `06_analysis_2.qmd`, `00_all.qmd`, `99_proj_func` | Marie        |

All members are responsible for all parts of the project.

## Collaboration Conventions

-   Work on branches and merge through pull requests with at least one reviewer.
-   Make small commits with descriptive messages, e.g. "Add GSE7904 sample metadata import".
-   Use relative paths via `here::here()` only. No personal absolute paths.
-   Every `.qmd` must run from a fresh R session, reading its inputs from files and not from objects created in other documents.
-   Naming: group labels are `tumour` / `normal`; sample and probe IDs follow `<agree on a convention>`.
-   Do not re-clean or redefine the sample groups downstream of `02_clean.qmd`.

## Declaration of Generative AI Use

Generative AI tools were used as supporting tools during the development of this project. In particular, ChatGPT (OpenAI) was used to:

- Explain R programming concepts, functions, and methods relevant to the data-processing and analysis workflow.
- Support code review, debugging, and troubleshooting of R and Quarto scripts.
- Assist with understanding bioinformatics concepts and the methodological choices made during the analysis.
- Support the preparation and editing of project documentation, including README sections and presentation materials.

AI-generated suggestions and explanations were critically reviewed by the project members. Code was checked and tested within the project workflow, and scientific and methodological decisions were made and evaluated by the students.

The project members take full responsibility for the final code, analyses, interpretations, documentation, and conclusions. Generative AI was used as an assistance tool and not as a substitute for understanding or validating the work.

**AI tool reference:** OpenAI. ChatGPT. https://chatgpt.com/ (accessed during project preparation, November 2026).

## Data Dictionary and Notes

\<Link or short summary of the cleaned-data format from Member 2: column names, types, group definitions, excluded samples and why.\>
