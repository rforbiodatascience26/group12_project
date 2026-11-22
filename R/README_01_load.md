# README --- GSE7904 data loading (`01_load.qmd`)

## 1. Purpose

This component loads the GEO Series Matrix dataset **GSE7904**, extracts
its sample metadata and gene-expression measurements, performs initial
checks, classifies samples into biological tissue groups, and saves two
tab-separated output files.

The script preserves the expression values as supplied in the GEO Series
Matrix file. It does not perform additional normalization, logarithmic
transformation, probe-to-gene annotation, or differential-expression
analysis.

## 2. Dataset and input

-   **GEO accession:** GSE7904
-   **Organism:** *Homo sapiens*
-   **Platform:** GPL570 --- Affymetrix Human Genome U133 Plus 2.0 Array
-   **Measurement values:** dChip signal values in the GEO Series Matrix
    file
-   **Expected samples:** 62
-   **Expected probe identifiers:** 54,675

The original compressed input file is:

`data/_raw/GSE7904_series_matrix.txt.gz`

The script downloads it from the NCBI GEO FTP server if it is not
already present. If the file already exists, it reuses the existing
file.

## 3. Output files

  -----------------------------------------------------------------------
  Output file                         Contents
  ----------------------------------- -----------------------------------
  `data/01_dat_load.tsv`              Expression matrix: one row per
                                      probe set and one column per
                                      sample, in addition to the `ID_REF`
                                      probe identifier column

  `data/01_sample_metadata.tsv`       One row per sample, containing
                                      sample identifiers, titles, source
                                      descriptions, GEO characteristics,
                                      and the assigned `tissue_group`
  -----------------------------------------------------------------------

The `data/` directory is ignored by Git in this project. The script
therefore creates `data/` and `data/_raw/` automatically when needed.

## 4. Workflow overview

1.  Load the required R packages and define the project and data
    directories.
2.  Construct the GEO download URL and download the Series Matrix file
    if it is missing.
3.  Check that the input file exists and is not empty; calculate its MD5
    checksum.
4.  Read the compressed file into R as lines and inspect its beginning.
5.  Extract sample identifiers, titles, source descriptions, and sample
    characteristics.
6.  Locate and import the expression matrix between the GEO table
    boundary markers.
7.  Inspect dimensions, sample categories, and expression-value
    statistics.
8.  Assign each sample to a tissue group.
9.  Check sample correspondence, duplicate probe identifiers, and
    missing expression values.
10. Write the expression matrix and metadata to TSV files.
11. Read both files back and validate the saved outputs.

## 5. Understanding the R code

### Packages

-   `library(tidyverse)` loads a collection of packages for data
    manipulation and related tasks. In this script, functions such as
    `tibble()`, `count()`, `select()`, `mutate()`, `case_when()`, and
    the pipe operator `%>%` are used.
-   `library(here)` provides `here::here()`, which builds file paths
    relative to the project root rather than relying on a particular
    working directory.

### Project paths and directory creation

`project_root <- here::here()` identifies the project root.

`here::here("data", "_raw")` and `here::here("data")` construct the
raw-data and output-directory paths.

`dir.create(..., recursive = TRUE, showWarnings = FALSE)` creates a
directory if it does not exist. `recursive = TRUE` also allows parent
directories to be created. `showWarnings = FALSE` suppresses the warning
that would otherwise appear if the directory already exists.

### Downloading and checking the GEO file

`paste0()` joins text fragments without adding separators. It is used to
construct the download URL and local filename.

`file.path()` joins path components using the operating system's path
separator.

`file.exists()` checks whether a file exists. The
`if (!file.exists(...))` condition means that the download is performed
only when the local file is absent.

`download.file()` downloads the GEO file. The `mode = "wb"` argument
requests binary mode, which is appropriate for a compressed `.gz` file.

`file.info(... )$size` obtains the file size in bytes. The script
divides this by `1024^2` to display the size in megabytes.

`stop()` terminates execution with an error message if the file is
missing or empty.

`tools::md5sum()` calculates an MD5 checksum for the downloaded file.
The `package::function` notation calls a function from a package without
attaching that package with `library()`.

### Reading the compressed GEO file

`gzfile(..., open = "rt")` opens a gzip-compressed file as a text
connection.

`readLines()` reads the connection into a character vector, with one
file line per vector element. `warn = FALSE` suppresses a warning about
a final line that may not end with a newline.

`close()` closes the connection after reading.

`head(geo_lines, 35)` selects the first 35 lines. `cat(..., sep = "\n")`
prints them with line breaks.

Reading the file into `geo_lines` makes it possible to locate and parse
the metadata and expression-table sections separately.

### Extracting sample metadata

The function `extract_sample_field()` is defined once and reused for
different GEO metadata fields.

-   `function(field_name) { ... }` defines a function with one input
    argument.
-   `startsWith(geo_lines, field_name)` returns `TRUE` for lines that
    begin with the supplied field name.
-   `which()` returns the positions of the matching lines.
-   `length(line_index) == 0` checks whether no matching line was found.
    In that case, `return(NULL)` exits the function and returns `NULL`.
-   `read.delim()` reads tab-separated text into a data frame. Here,
    `text =` supplies the selected line directly as text,
    `header = FALSE` says that the line does not contain a separate
    column-name row, and `sep = "\t"` specifies tab separation.
-   `quote = "\""` tells the parser that double quotation marks delimit
    quoted text.
-   `check.names = FALSE` preserves the supplied column names instead of
    automatically modifying them.
-   `stringsAsFactors = FALSE` keeps text values as character data
    rather than converting them to factors.
-   `parsed[1, -1]` selects the first row and all columns except the
    first. The first field contains the GEO metadata label; the
    remaining fields contain values for the samples.
-   `unlist(..., use.names = FALSE)` converts the selected one-row
    result into a vector and removes element names.
-   `as.character()` ensures that the returned sample values are
    character strings.

The function is used to extract the GEO sample accession, sample title,
and sample source fields.

### Extracting all sample-characteristic rows

`which(startsWith(...))` finds every line beginning with
`!Sample_characteristics_ch1`.

`lapply(characteristic_indices, function(i) { ... })` applies the same
parsing operation to every matching line. The result is a list: each
list element contains the characteristic values for all samples from one
GEO characteristics row.

`tibble()` creates the initial metadata table with one row per sample.

The `for` loop adds each characteristics vector as a separate column.
`seq_along(characteristics)` generates the sequence of valid list
positions. `paste0("characteristic_", i)` creates column names such as
`characteristic_1` and `characteristic_2`.

For example, `characteristic_1` means the first characteristics row in
the GEO file. It does not mean that a sample has only one
characteristic.

`rm()` removes temporary objects that are no longer needed. It does not
remove the metadata table.

### Importing the expression matrix

`grep("^!series_matrix_table_begin", geo_lines)` and
`grep("^!series_matrix_table_end", geo_lines)` locate the beginning and
end markers of the expression table. The `^` in each regular expression
means that the marker must occur at the start of a line.

The `if` condition checks that each marker occurs exactly once and that
the beginning marker appears before the ending marker. If this structure
is not found, `stop()` halts execution.

The expression rows are selected using:

-   `table_begin + 1`: the line immediately after the beginning marker;
-   `table_end - 1`: the line immediately before the ending marker.

`paste(expression_lines, collapse = "\n")` joins the selected lines into
one text string, inserting a newline between them.

`read.delim()` imports this text as a tab-separated data frame.
`header = TRUE` indicates that the first expression-table row contains
column names. `na.strings = c("", "NA")` tells R to interpret empty
fields and the text `NA` as missing values.

The resulting `expression_data` object contains the `ID_REF` probe
identifier column and the sample measurement columns.

### Initial inspection and summary statistics

`dim()` reports the number of rows and columns in an object.

`head()` displays the first rows. The expression import chunk selects
the first six columns for inspection.

`str()` displays the structure and data types of an object.

`count(characteristic_1, sort = TRUE)` counts samples in each value of
the first GEO characteristics field and sorts the counts.

`select()` chooses the columns displayed in the sample-classification
inspection.

`n = Inf` in the `print()` call requests all rows rather than the
default limited number.

For expression statistics, `expression_data[-1]` selects every
expression-data column except the first probe identifier column.
`unlist(..., use.names = FALSE)` combines the values into one vector.

`summary()` gives a statistical summary. The script also calculates:

-   `min()`: minimum value;
-   `max()`: maximum value;
-   `mean()`: arithmetic mean;
-   `sd()`: standard deviation.

These statistics describe the imported values; they do not transform
them.

### Assigning biological tissue groups

`mutate()` adds or modifies a column in a data frame.

`case_when()` evaluates conditions in order and assigns the
corresponding value:

-   `Basal`, `non-BLC`, and `BRCA1` are assigned to `Tumour`;
-   `NB` is assigned to `Normal breast`;
-   `NO` is assigned to `Normal organelle`;
-   any other value is assigned `NA_character_`, a missing character
    value.

`%in%` tests whether a value belongs to a specified set.

`is.na()` identifies missing values, and `any()` checks whether at least
one such value is present. If any sample remains unclassified, the
script stops with an error.

The script retains `Normal breast` and `Normal organelle` as separate
groups.

### Validation checks

The validation section compares the number of sample rows in the
metadata with the number of expression columns, excluding the probe
identifier column.

`colnames(expression_data)[-1]` obtains the expression sample
identifiers. The two `%in%` checks test that every expression sample has
a metadata row and every metadata sample has an expression column.

`duplicated(expression_data$ID_REF)` identifies probe identifiers that
have appeared earlier in the column. `sum()` counts the duplicated
entries.

`is.na(expression_data[-1])` identifies missing expression measurements,
and `sum()` counts them.

These checks are printed as diagnostic results; the code does not stop
solely because a duplicate probe ID or missing expression value is
reported.

### Saving and final validation

`readr::write_tsv()` writes a data frame as a tab-separated file.

`file.exists()` checks that both output files were created. If either is
absent, `stop()` terminates execution.

`readr::read_tsv()` reads the saved TSV files back into R.
`show_col_types = FALSE` suppresses the column-type message.

`stopifnot()` checks that each supplied condition is `TRUE`. It stops
execution if a condition fails.

The final checks confirm that:

-   the saved expression matrix has the same dimensions as the in-memory
    expression matrix;
-   the saved metadata table has the same dimensions as the in-memory
    metadata table;
-   the saved expression sample-column names are identical, in order, to
    the saved metadata sample identifiers.

Finally, `count(tissue_group)` displays the number of samples in each
assigned tissue group.

## 6. Biological groups represented in the metadata

The script maps the first GEO sample-characteristics field to the
following groups:

  Original GEO characteristic   Assigned `tissue_group`
  ----------------------------- -------------------------
  `Basal`                       Tumour
  `non-BLC`                     Tumour
  `BRCA1`                       Tumour
  `NB`                          Normal breast
  `NO`                          Normal organelle

The dataset composition recorded in the QMD is 43 tumour samples, 7
normal breast samples, and 12 normal organelle samples, for 62 samples
in total. The two normal categories remain separate in the output
metadata.

## 7. Scope of this component

This component is responsible for loading and organizing the source
data. In particular:

-   It preserves the GEO dChip signal values as supplied.
-   It retains the original Affymetrix probe-set identifiers in
    `ID_REF`.
-   It extracts the GEO sample metadata and assigns the stated tissue
    groups.
-   It saves the expression matrix and sample metadata separately.

It does not carry out additional normalization or log transformation,
map probe sets to gene symbols, aggregate multiple probes into genes, or
perform downstream statistical analysis.
