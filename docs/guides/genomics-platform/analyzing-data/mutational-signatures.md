## Overview

**Mutational Signatures** (abbreviated as **mtsg**) finds and quantifies
[COSMIC mutational signatures] across samples.

mtsg uses a base set of mutational signatures extracted by [SigProfiler] for
[single-base substitutions] (SBS), i.e., single-nucleotide variants (SNV),
using 2780 whole-genome variant calls from the ICGC/TCGA [Pan-Cancer Analysis
of Whole Genomes] (PCAWG) project.

[COSMIC mutational signatures]: https://cancer.sanger.ac.uk/cosmic/signatures
[SigProfiler]: https://cancer.sanger.ac.uk/cosmic/signatures/sigprofiler.tt
[single-base substitutions]: https://cancer.sanger.ac.uk/cosmic/signatures/SBS/index.tt
[Pan-Cancer Analysis of Whole Genomes]: https://dcc.icgc.org/pcawg

## Inputs

| Name                           | Type           | Description                                                               | Example               |
|--------------------------------|----------------|---------------------------------------------------------------------------|-----------------------|
| [VCF(s)]                       | Array of files | One or more VCF sources. The files can be either uncompressed or gzipped. | [`*.vcf`, `*.vcf.gz`] |
| Genome build                   | String         | The genome build used as reference. [default: "GRCh38"]                   | GRCh38                |

[VCF(s)]: #vcfs

### Input configuration

Mutational Signatures only requires VCFs as inputs. All other inputs are
optional.

<h4 id="VCF(s)">VCF(s)</h4>

_VCF(s)_ is a list of VCF inputs. The inputs are expected to be single-sample
and uncompressed or gzipped. The basename of the filename is used as the
sample name.

## Outputs

| Name                                 | Type | Description                                                                  |
|--------------------------------------|------|------------------------------------------------------------------------------|
| [Raw signature activities]           | File | A tab-delimited file of the raw results with signature activities per sample |
| [Signature activities visualization] | File | HTML file for interactive plotting                                           |

[Raw signature activities]: #raw-signature-activities
[Signature activities visualization]: #signature-activities-visualization

## Creating a workspace

Before you can run one of our workflows, you must first create a workspace in
DNAnexus for the run. Refer to [the general workflow
guide](running-sj-workflows.md#getting-started) to learn how to create a
DNAnexus workspace for each workflow run.

You can navigate to the Mutational Signatures workflow page
[here](https://platform.stjude.cloud/workflows/mutational_signatures).

## Uploading Input Files

Mutational Signatures requires at least one VCF to be uploaded.

Refer to [the general workflow
guide](running-sj-workflows.md#uploading-files) to learn how to upload input
files to the workspace you just created.

## Running the Workflow

Refer to [the general workflow
guide](running-sj-workflows.md#running-the-workflow) to learn how to launch
the workflow, hook up input files, adjust parameters, start a run, and
monitor run progress.

## Analysis of Results

Each tool in St. Jude Cloud produces a visualization that makes understanding
results more accessible than working with spreadsheets or tab-delimited
files. This is the primary way we recommend you work with your results.

Refer to [the general workflow
guide](running-sj-workflows.md#custom-visualizations) to learn how to access
these visualizations.

We also include the raw output files for you to dig into if the visualization
is not sufficient to answer your research question.

Refer to [the general workflow
guide](running-sj-workflows.md#raw-results-files) to learn how to access raw
results files.

## Interpreting results

Upon a successful run of Mutational Signatures, twp files are saved to the
results directory: raw signature activities and a visualization file.

### Raw signature activities

_Raw signature activities_ is a tab-delimited file of the raw results with
the signature activity counts for each input sample . Column 1 is the SBS
signature identifier, and columns 2 through N are the counts of signature
matches for each input sample. Signatures that have no matches across all
samples are omitted.

<h5>Example</h5>


| Samples          | SJMEL001001_D1 | … | SJNBL001_D |
|------------------|---------------:|:-:|-----------:|
| Signature Subs-0 |            233 | … |         13 |


### Signature activities visualization

_Signature activities visualization_ is an HTML file that can be used for
interactive plotting.

When opened in a web browser, a set of controls allows plotting various
stacked bar charts: activity by signature, activity by disease, and activity
by sample per disease. The activity counts can be stacked as absolute values
or as a percentage of the total.

## Frequently asked questions

None yet! If you have any questions not covered here, feel free to reach out
on [our contact
form](https://hospital.stjude.org/apps/forms/fb/st-jude-cloud-contact/).

# References

  * Alexandrov, L.B., Kim, J., Haradhvala, N.J. _et al_. The repertoire of
    mutational signatures in human cancer. _Nature_ **578**, 94–101 (2020).
    https://doi.org/10.1038/s41586-020-1943-3

  * Bergstrom, E.N., Huang, M.N., Mahto, U. _et al_.
    SigProfilerMatrixGenerator: a tool for visualizing and exploring patterns
    of small mutational events. _BMC Genomics_ **20**, 685 (2019).
    https://doi.org/10.1186/s12864-019-6041-2

## Similar Topics

[Running our Workflows](../analyzing-data/running-sj-workflows.md)
[Working with our Data Overview](../managing-data/working-with-our-data.md)
[Upload/Download Data (local)](../managing-data/upload-local.md)

[10.1186/s13073-018-0539-0]: https://doi.org/10.1186/s13073-018-0539-0
[29695279]: https://www.ncbi.nlm.nih.gov/pubmed/29695279

[mtsg]: https://github.com/stjude/mtsg
