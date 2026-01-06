# DiffexpWedgeR

DiffexpWedgeR is an R command-line pipeline that performs differential expression analysis from gene-level count tables using edgeR (QLF + TREAT), generates QC and visualization outputs, and runs GO enrichment using limma’s goana/topGO workflow. The pipeline is driven by a JSON configuration file that defines samples, factors, thresholds, organism settings, and output paths.

## Requirements

R (recommended >= 4.1)

Packages:

* edgeR
* org.Hs.eg.db
* GO.db
* topGO
* argparse
* dplyr
* tibble
* pheatmap
* EnhancedVolcano
* ggplot2
* jsonlite

## Input

### Count files

Each sample must provide a tab-separated count table with:

* Header row
* First column as gene identifiers (used as row names)
* One numeric count column

Example format:

GeneID\tCounts
TP53\t123
BRCA1\t45

### JSON configuration

The pipeline reads a JSON file passed via `--config`. It expects these fields:

* project.p_cutoff
* project.lfc_cutoff
* project.interaction
* project.data_source
* project.output_dir
* project.organism.species_code
* project.samples.sample_name
* project.samples.countFile
* project.samples.factors (per-sample list of {name, levels})

data_source must be one of:

* ncbi (expects gene identifiers as SYMBOL)
* ensembl (expects ENSEMBL IDs; version suffixes like .12 are removed)

interaction controls the model formula for multifactor designs:

* "True" uses full interaction (*)
* otherwise additive (+)

## Usage

Run the script:

Rscript diffexp_wedger.R --config config.json

## Output

The pipeline writes the following into `project.output_dir`:

Tables:

* raw_counts_table.txt
* raw_filtered_counts_table.txt
* samples_table.txt

QC plots:

* MDPlots.jpg
* PCA_plot.jpg
* Dispersion_plot.jpg
* Fitted_mean_ql_dispersion_plot.jpg (single-factor)
* Fitted_mean_ql_dispersion_plot_single.jpg (multi-factor)
* Fitted_mean_ql_dispersion_plot_multi.jpg (multi-factor)

Per comparison / coefficient outputs:

* <contrast>_result_qlf_test_w<lfc_cutoff>cutoff.txt
* <contrast>_summary_qlf_test_w<lfc_cutoff>cutoff.txt
* <contrast>_MDPlot_w<lfc_cutoff>cutoff.jpg
* <contrast>_volcano_plot.jpg
* <contrast>_GO_results.txt
* <contrast>_heatmap_log2_transformed.jpg (single-factor and multi-factor where applicable)

For single-factor designs, all pairwise contrasts between group levels are tested.
For multi-factor designs, each model coefficient (excluding intercept) is tested.

## Notes on filtering and thresholds

* Genes are filtered using edgeR `filterByExpr`.
* Normalization uses TMM.
* Differential testing uses quasi-likelihood framework and TREAT with an effect-size threshold based on `lfc_cutoff`.
* Significance summaries use `p_cutoff` applied to TREAT results.

## Organism and GO enrichment

GO enrichment is computed using `goana` with Entrez IDs obtained via `org.Hs.eg.db`. This implementation is currently configured for human annotation mapping; for non-human organisms, the annotation database and mapping strategy must be adapted accordingly.
