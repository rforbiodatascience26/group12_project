# Project Contributors

| Name | Student ID | GitHub username |
|---|---|---|
| Ricsi (Member 1) | <sXXXXXX> | <username> |
| Daniel (Member 2) | <sXXXXXX> | <username> |
| Kostas (Member 3) | <sXXXXXX> | <username> |
| Sasa (Member 4) | <sXXXXXX> | <username> |
| Marie (Member 5) | s260367 | mariekvl |

## Presentation

[Direct link to the presentation](https://raw.githack.com/<github-org>/<repo-name>/main/doc/presentation.html)

## Project Overview

**Group 12: Gene expression differences between healthy and cancerous breast tissue (GSE7904)**

Research question: which expression probes differ between breast tumour
samples and normal breast tissue in GSE7904, and do overall expression
profiles separate the two groups?

Short summary: <2-3 sentences on data, methods and main finding. Fill in at the end.>

## Data Retrieval

Data are **not** stored on GitHub. To reproduce the project:

1. Dataset: GEO accession [GSE7904](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE7904)
   (platform GPL570, Affymetrix Human Genome U133 Plus 2.0).
2. Download <exact file names or programmatic retrieval method> and place
   them in `data/_raw/` with these file names: `<expected file names>`.
3. Raw files must never be edited.

<Replace with the exact steps from `01_load.qmd` once Member 1 has settled them.>

## How to Run the Project

1. Clone the repository and open `group12_project.Rproj` in RStudio.
2. Install the required packages:

   ```r
   install.packages(c("tidyverse", "here", "broom", "quarto", "ragg"))
   # <add Bioconductor or other packages as they are used>
   ```

3. Create the data folders if they are missing: `data/_raw/` and `data/`.
4. Retrieve the raw data as described above.
5. Render the full workflow by rendering `R/00_all.qmd` (or in the console:
   `quarto::quarto_render("R/00_all.qmd")`).
6. Output HTML files appear in `results/`.

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

| File | Lead |
|---|---|
| `01_load.qmd` | Ricsi |
| `02_clean.qmd` | Daniel |
| `03_augment.qmd`, `04_describe.qmd` | Kostas |
| `05_analysis_1.qmd` | Sasa |
| `06_analysis_2.qmd`, `00_all.qmd`, `99_proj_func` | Marie |

All members are responsible for all parts of the project.

## Collaboration Conventions

- Work on branches and merge through pull requests with at least one reviewer.
- Make small commits with descriptive messages, e.g. "Add GSE7904 sample metadata import".
- Use relative paths via `here::here()` only. No personal absolute paths.
- Every `.qmd` must run from a fresh R session, reading its inputs from files and not from objects created in other documents.
- Naming: group labels are `tumour` / `normal`; sample and probe IDs follow `<agree on a convention>`.
- Do not re-clean or redefine the sample groups downstream of `02_clean.qmd`.

## Data Dictionary and Notes

<Link or short summary of the cleaned-data format from Member 2:
column names, types, group definitions, excluded samples and why.>
