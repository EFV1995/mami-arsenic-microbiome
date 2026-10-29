# MAMI arsenic analysis — standalone R Markdown files

**One README + 15 Rmds.

Each Rmd contains its own data loading, analysis, plotting and export functions.
It can be saved anywhere on your laptop. Set the workbook path at the top, save,
and **Knit from the beginning**. You do not need an `.Rproj`, `.Renviron`, installer
script, launcher, helper `.R` file or outputs from an earlier analysis.

Module 14 is a review, not an upstream analysis: it independently checks the input
and optionally gathers completed result tables from the other modules. Run it last
for a complete review, using the same results folder.

## 1. Open one Rmd, set the paths, Knit

Install [R](https://cran.r-project.org/) and
[RStudio Desktop](https://posit.co/download/rstudio-desktop/) first. The supplied
source analysis used R 4.4.3. Other versions/packages may give different results;
each run records the actual versions. If RStudio requests the R Markdown packages,
accept the installation, or run this **once** in its Console:

```r
install.packages(c("rmarkdown", "knitr"), repos = "https://cloud.r-project.org")
```

Open any Rmd. Its first code chunk is called **`user-settings`**. Set:

```r
INPUT_FILE <- "C:/my-study/ALL_DATA_clean_for_analysis.xlsx"
OUTPUT_ROOT <- "C:/MAMI_results_simple"
AUTO_INSTALL_PACKAGES <- TRUE
```

Replace the example input path with your actual workbook location. Use forward
slashes `/`, even on Windows. Do not enter only a folder: include `.xlsx`.
If the workbook sits beside the Rmd, the supplied default
`INPUT_FILE <- "ALL_DATA_clean_for_analysis.xlsx"` is sufficient. Relative paths
are interpreted relative to the saved Rmd, not to a required project layout.

**Use the same input workbook and `OUTPUT_ROOT` in every Rmd.** The output root is
created automatically. A short local path avoids the long-path problems encountered
with BLAST and graphics. If `C:/` is not writable, choose another short writable
folder, such as `D:/MAMI_results_simple`. On macOS/Linux the default is
`~/MAMI_results_simple`.

With `AUTO_INSTALL_PACKAGES <- TRUE`, each module installs missing R packages from
CRAN. Internet access and package-installation permissions are needed. A failed
installation stops with the missing package names; do not ignore that error.
The full set used across the modules is `readxl`, `dplyr`, `tidyr`, `ggplot2`,
`openxlsx`, `sandwich`, `lmtest`, `vegan`, `scales`, `knitr`, `data.table`, `curl`,
and optionally `ragg` for PNG export. R Markdown also needs Pandoc; it is bundled
with RStudio. Check `rmarkdown::pandoc_available()` if rendering cannot find it.

Save the file, restart R with **Ctrl + Shift + F10** on Windows when switching from
an old session, then click **Knit → HTML**. Run one document at a time. Do not paste
long functions into the Console or resume a failed run halfway through setup.
Start with 00 to audit your data, then work through the desired modules. All of
00–13 calculate their own results directly from the workbook; numeric order is a
convenience, not a dependency chain.

**This download contains only the README and 15 Rmds in one flat folder.**
The extra folders appear only after execution, because results and downloaded
references need somewhere to be saved. There is nothing to build or arrange first.

## 2. BLAST+ — needed only for the default module 07

Install [NCBI BLAST+](https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/)
for your operating system. On Windows choose the `win64.exe` installer, not the
source-code archive. The executable used here is **`blastp`**.

In the **first chunk of module 07**, use the path to your installed executable:

```r
BLASTP <- "C:/Program Files/NCBI/blast-2.17.0+/bin/blastp.exe"
RUN_BLAST <- TRUE
```

The version/path above matches the supplied setup; it is not a requirement to
install that exact version. If BLASTP is already on your system PATH, you can set
`BLASTP <- ""`; the module then uses `Sys.which("blastp")`. A path that does not
exist also falls back to PATH. The module stops if the requested search has no
working executable.

You can check a known path in the Console:

```r
blastp <- "C:/Program Files/NCBI/blast-2.17.0+/bin/blastp.exe"
stopifnot(file.exists(blastp))
system2(blastp, args = "-version", stdout = TRUE, stderr = TRUE)
```

Module 07 also runs its own reference-protein self-alignment test before the large
search. This tests execution and file access, not protein function.

### What the default reference search covers

The supplied defaults request annotations and BLASTP searches for **all species
representatives in UHGG v2.0.2**, with no per-genus/top-N cap:

```r
ANNOTATION_SCOPE <- "all_representatives"
BLAST_SCOPE <- "all_representatives"
```

The versioned catalogue describes 4,744 species representatives, not all 289,232
constituent genomes or every strain. Reference downloads can occupy several
gigabytes; check free disk space and internet access. The module includes the
90 public UniProt reference queries and their provenance; these are **not
participant sequences**. No live UniProt request is required for that snapshot.
Uncached UHGG annotations/proteomes do require internet access.

Successful validated downloads and fingerprinted BLAST results are reused after
an interrupted run. Do not delete the cache merely to rerun the document. Coverage
is explicitly audited: requested, completed, failed and unassessed are different
statuses. A cached positive does not establish complete catalogue coverage.

To reuse an older **public-reference** cache, optionally edit this line in the
advanced settings of 07:

```r
LEGACY_REFERENCE_CACHE <- "C:/old_public_references"
```

It copies usable public files; it does not import old study associations. Otherwise
leave it empty and the module creates its own cache under the output directory.

The same advanced block contains `LOCAL_UHGG_METADATA <- ""` for an optional
existing pinned metadata TSV, and `TAXONOMY_CROSSWALK_FILE <- ""` for an optional
approved taxonomy crosswalk. Neither file is needed for the default run.
A crosswalk, when used, must have `source_label,target_rank,target_name,approved,evidence_url`;
`target_rank` is `genus`, `species` or `representative`. Only approved rows with
documented evidence and existing catalogue targets are accepted. Formatting
normalisation is automatic; biological synonyms and group-level matches are not
guessed just to obtain a positive hit.

Separate maternal and infant networks/forest plots are produced only when that
population has links meeting **both** the chosen statistical rule and candidate
gene evidence on a mapped human-gut reference representative. All tested links
remain in the tables. The default `FOLLOWUP_RULE <- "nominal"` means exploratory
p < 0.05, **not** milk-wide FDR support. `"FDR"` requires q(all milk) < 0.05.
No rule is relaxed to force a network. Public-reference genes are not genes measured
in participants. Broad arsR-family similarity does not establish arsenic specificity.

`RUN_BLAST <- FALSE` is permitted only as an explicitly annotation-only analysis;
it is not equivalent to a completed sequence screen. Use a new `OUTPUT_ROOT` for
an alternative specification and retain its coverage/status records.

## 3. Your Excel workbook

Keep the workbook private, anywhere on your computer. **No real or dummy data
workbook is included in this download.** Create these six sheets exactly, with
column headings in row 1 and no extra title rows:

| Sheet | Contents |
|---|---|
| `clinical_data` | Linked clinical records and the listed covariates |
| `genus_mother_1month` | Maternal one-month stool: IDs followed by raw genus counts |
| `genus_infant_1month` | Infant one-month stool: IDs followed by raw genus counts |
| `maternal_foods_48h` | Maternal third-trimester food intakes |
| `maternal_nutrients_48h` | Matching energy and nutrient intakes |
| `hm_arsenic` | Milk and infant urine, one row per specimen |

The legacy `_48h` sheet suffix is retained for compatibility; it **does not** mean
a 48-hour dietary recall. Diet is maternal third trimester. Both stool collections
are one month postpartum. Arsenic has the recorded one-/two-month sampling labels.

**All heads below are fictional layout examples, not a dataset to analyse.** They
do not have sufficient sample size or variability to fit the intended models.
Columns are normalised for punctuation/case; retain the documented names to avoid
unintended missing exposures. All six sheets should be present for the complete
workflow, even though 07 reads only arsenic and the two microbiota sheets.


#### `clinical_data`

```csv
sample_id,family_code,gender,maternal_age,pregestational_bmi,gestational_age,delivery_mode,breastfeeding_exposure_at_1_month,breastfeeding_exposure_at_2_month,infant_ab_before_2_months,infant_ab_before_1_months,siblings
DEMO_C_A,FAM_A,Female,32,22.4,40,V,Breastfed_at_1month,Breastfed_at_2months,0,0,1
DEMO_C_B,FAM_B,Male,29,24.1,39,C,Breastfed_at_1month,Stopped_before_2months,1,0,0
```

Use numeric ages/BMI/gestational age, text sex and delivery codes, and consistent
binary indicators. Missing values are blank cells or text `NA`, never zero unless
zero is genuinely observed. Keep the source spelling
`infant_ab_before_1_months`. The code records clinical conflicts; it does not
arbitrarily treat conflicting repeated records as independent participants.
Duration-derived categories mean **any breastfeeding**, not exclusivity.
Antibiotic recipient and timing need a codebook; they are not inferred from a
column name.

#### `genus_mother_1month`

```csv
sample_id,family_code,g__Bifidobacterium,g__Bacteroides,g__Blautia,g__Fusicatenibacter
DEMO_M_A,FAM_A,450,1800,1300,380
DEMO_M_B,FAM_B,700,2100,900,510
```

#### `genus_infant_1month`

```csv
sample_id,family_code,g__Bifidobacterium,g__Bacteroides,g__Streptococcus,g__Escherichia-Shigella
DEMO_I_A,FAM_A,6500,300,900,700
DEMO_I_B,FAM_B,4200,800,1200,1100
```

Counts must be **nonnegative integers**, not percentages, relative abundances or
CLR coordinates. Include the full supplied genus count universe; its row sum is
the denominator for relative abundance. Genus columns should begin with `g__`
(or `g_` after normalisation). Do not add absent placeholder genera as zeros unless
those are genuine measured zero counts. Empty libraries are excluded; missing
counts are not silently made zero. At most one valid profile per family is
supported in each one-month stool sheet.

#### `maternal_foods_48h`

This is a compact head. The expandable header lists at the end of this README
provide the complete original food and nutrient column names.

```csv
sample_id,family_code,Mussel,Squid,Prawns/shrimp,Hake,Salmon,Tuna_in_oil,White_rice,Red_wine,White_wine,Cava_(sparkling_wine),Beer
DEMO_D_A,FAM_A,5,8,4,20,12,10,30,0,0,0,0
DEMO_D_B,FAM_B,0,6,5,18,10,8,40,3,0,0,0
```

These numbers illustrate numeric cells only. **Document the actual intake units
and reference period in your codebook.** The supplied source did not fully verify
all food/nutrient units; the repository does not silently assign them. There is
no `Energy` field required in this sheet: the analysis links energy from the
nutrient worksheet using `family_code`. Missing composite constituents produce
an unavailable composite, not a partial sum relabelled as the complete group.

#### `maternal_nutrients_48h`

```csv
sample_id,family_code,Energy,Total_protein,Animal_protein,Plant_based_protein,Methionine,Folate_total,Choline_total
DEMO_D_A,FAM_A,2200,80,48,32,1.5,330,280
DEMO_D_B,FAM_B,2400,90,51,39,1.7,360,300
```

The expandable header list below documents the source's broader nutrient screen.
`Energy` must be a numeric daily energy estimate; the supplied analysis treats it
as **kcal/day**, which must be confirmed against your source codebook. Retain each
nutrient's documented source unit and do not mix units between rows. The script
calculates energy-adjusted variables; do not pre-adjust or rescale the workbook
unless your analytical design is explicitly revised.

#### `hm_arsenic`

```csv
sample_id,family_code,timepoint_code,timepoint_month,sample,sample_site,AsB,As(III),DMA,MMA,As(V)
DEMO_H_A,FAM_A,1,1m,milk,milk,8.0,0.12,1.4,0.08,0.25
DEMO_U_A,FAM_A,1,1m,urine,urine,0.20,0.01,0.15,0.02,0.04
DEMO_H_B,FAM_B,2,2m,milk,milk,10.0,0.09,1.1,0.10,0.18
DEMO_U_B,FAM_B,2,2m,urine,urine,0.18,0.02,0.12,0.03,0.05
```

These are fictitious numbers in unspecified illustrative units, not expected
ranges. Use **`milk` or `urine`** in `sample_site`, and `1`/`2` or `1m`/`2m` in
`timepoint_month`. `timepoint_code` and `sample` are retained compatibility columns;
the actual matching uses matrix, month and the harmonised family key. Do not
enter `iAs`, totals, percentages or ratios in place of the five measured species:
the code derives them.

Use nonnegative numeric concentrations in documented, consistent units **within
each matrix**. Do not replace missing values with zero or label zeros as below-LOD
unless the laboratory established that meaning. Zero denominators make ratios
undefined. The software does not impute LOD/LOQ or invent urinary dilution factors.


### Input and matching rules that matter

- Keep `sample_id` and `family_code` as text. The same mother–infant family must use
  the same family code across sheets. `sample_id` identifies the specimen/record;
  `family_code` is the linkage key. Simple consistent keys like `FAM_A` are suitable
  for a new workbook. The inherited legacy MAMI numeric/W/DB rules are not a
  general-purpose identifier resolver: inspect module 00's linkage audit.
- Include every supplied genus-count column, not just the taxa you intend to plot.
  Relative abundances use the full count sum. A genuine zero is different from a
  missing count. Do not encode missingness, unmeasured taxa or unavailable food
  items as zero. One valid stool profile per family/compartment is supported here;
  twin or ambiguous infant records need explicit resolution.
- The supplied design has independent one-/two-month groups within each arsenic
  matrix, not repeated specimens from the same infant. Repeated longitudinal data
  require a different model. Exact milk–urine pairing requires the same family
  and nominal month; do not merge families to increase sample size.
- Use numeric cells for measured concentrations, counts, ages, BMI, energy and
  intakes. Blank cells or text `NA` are missing. Unit consistency and analytical
  LOD/LOQ remain laboratory/codebook responsibilities. Formula-fed status and
  maternal intrapartum antibiotics must not silently become infant treatment or
  exclusive-breastfeeding information.
- Missing composite constituents make that composite unavailable. Missing energy
  makes energy density unavailable. Optional nutrient/food columns without data
  cannot reproduce tests for those variables. The broader screens use eligible
  numeric columns actually supplied. At least 20 valid dietary records with
  variability are needed for the inherited nutrient residual calculation; each
  model also has its own complete-case/parameter gate. A skipped model is not a
  negative finding.

## 4. Files to run

| Order | Standalone R Markdown | Purpose |
|---|---|---|
| 00 | `00_data_integrity_flow.Rmd` | Matching, missingness, dietary energy and source QC |
| 01 | `01_speciation_paired_milk_urine.Rmd` | Matrix-specific profiles and exact milk–urine correspondence |
| 02 | `02_diet_to_human_milk.Rmd` | Targeted/broad foods and nutrients; adjusted effects and sensitivity |
| 03 | `03_exploratory_diet_to_urine.Rmd` | Exploratory diet–urine concentration/fraction models |
| 04 | `04_maternal_microbiota.Rmd` | Maternal one-month stool associations and compositional QC |
| 05 | `05_infant_microbiota.Rmd` | Infant one-month stool associations and compositional QC |
| 06 | `06_stratification_maturation.Rmd` | Exploratory strata and dispersion; no validated maturation index |
| 07 | `07_milk_genus_gene_network.Rmd` | Full-catalogue reference screen and separate supported networks |
| 08 | `08_paired_maternal_infant_microbiota.Rmd` | Paired common-composition similarity/feasibility |
| 09 | `09_strict_triad_evidence.Rmd` | Diet–milk–urine edges on identical complete cases; not mediation |
| 10 | `10_arsenic_speciation_detection.Rmd` | Matrix-separated zero, percentage, ratio and month-group audits |
| 11 | `11_infant_urine_speciation_microbiota.Rmd` | Ratio-eligible infant microbiota and urinary-speciation analyses |
| 12 | `12_sparse_exposure_robustness.Rmd` | Consumer support, influence and fixed-scale resampling |
| 13 | `13_plant_protein_indicators.Rmd` | Direct plant-protein indicators; **no dietary PCA** |
| 14 | `14_final_results_review.Rmd` | Source audit and optional checksum-verified results review |

Only one active file exists for each module. The 07 file is the full-catalogue
network implementation, not the obsolete taxonomic proxy. The 13 file uses direct
plant-protein indicators.

## 5. Results are saved automatically

With the Windows defaults, the modules create:

```text
C:/MAMI_results_simple/
    00_data_integrity_flow/
    01_speciation_paired_milk_urine/
    ...
    07_milk_genus_gene_network/
    ...
    14_final_results_review/
```

Each result folder contains tables, figures and logs; most have `ALL_TABLES.xlsx`.
Module 07 has `07_MILK_GENUS_FULL_CATALOGUE.xlsx`, `GALLERY.html`, reference-cache
and BLAST-work subfolders. You do not create or configure these subfolders.
The HTML created by the Knit button normally sits beside its Rmd; the numerical
and image exports go to `OUTPUT_ROOT` regardless. HTML can contain private data.

Module 07's `31_reference_search_coverage.csv` and
`32_genus_reference_search_denominators.csv` distinguish intended from completed
reference coverage. `19_plot_selection_counts.csv` explains why a maternal or
infant network was or was not generated. Graphics export manifests record whether
SVG/PDF/PNG files actually succeeded.

Each run removes its previous completion receipt at the start and writes a new
receipt at the end. Module 14 checks the workbook/settings signature and script
checksum before importing results. It looks for the other Rmds beside itself, then
at their paths recorded in their run receipts, so those Rmds need not share a
folder. If a source was moved, rerun it at its new location (or put its unchanged
copy beside 14). Do not interpret a stale receipt or existing figure as proof of a
completed current run. Different inputs/settings must use a **new results root**.
Run modules sequentially, not concurrently into the same folder.

## 6. The analysis itself has not been simplified away

The code retains the supplied analytical definitions. In particular, genus
filtering is applied **within each eligible endpoint/model subset**, before zero
replacement: raw relative abundance >=0.01 (**1%**) in >=50% of those specimens.
This is not 0.01%. Only microbial zeros are replaced by 1e-6; sensitivity values
are 1e-7 and 1e-5. Positive microbial values and arsenic values are not given that
pseudocount. CLR uses the retained reclosed composition. All-zero or raw-constant
genera cannot become discoveries merely because their CLR varies. Diversity uses
original valid genus counts, not the filtered/pseudocounted composition.

Actual complete-case N, effect sizes, intervals, nominal p, declared-family q and
model status remain available. The full eligible family is corrected before
selecting interesting or reference-supported links. Different pseudocount/inference
families are not independent replications or interchangeable q-values. Unestimated
models are not reported as null results. No urinary-dilution values, assay units,
LOD/LOQ, exclusive feeding or causal transfer are invented. Gene networks remain
external candidate-evidence maps, not participant-level gene assays or mediation.

This is a packaging/path-setup revision, not a reanalysis. The preparation checks
compare the analytical functions and analysis chunks with the preceding repository
and scan the new files for external local-code dependencies and overlong lines.
Native R/Windows-BLAST end-to-end execution was not available in the authoring
environment. Each Rmd parses its saved code with your local R before lengthy work;
input validity, package installation and a successful full knit still require your
local run. No claim of fresh statistical results is made.


## 7. External reference and software provenance

Module 07 embeds the unchanged 90 usable public UniProt protein queries from the
source snapshot dated 27 October 2026, with accessions, marker labels, query URLs
and checksums. These are reference proteins, not study sequences. No live refresh
is claimed. The UniProt Consortium, *UniProt: the Universal Protein Knowledgebase
in 2025*, DOI **10.1093/nar/gkae1010**; consult the
[UniProt database terms](https://www.uniprot.org/help/license) (CC BY 4.0 for
copyrightable database content). Query values are retained; execution paths and
packaging were adapted.

UHGG v2.0.2 annotations/proteomes are downloaded on execution, not bundled.
[Versioned release documentation](https://ftp.ebi.ac.uk/pub/databases/metagenomics/mgnify_genomes/human-gut/v2.0.2/README_v2.0.2.txt).
Catalogue publication: Almeida et al., DOI **10.1038/s41587-020-0603-3**.
BLAST+ publication: Camacho et al., DOI **10.1186/1471-2105-10-421**.
R packages/NCBI binaries retain their own licences and are not redistributed.
[Official R Markdown rendering documentation](https://rmarkdown.rstudio.com/docs/reference/render.html)
and [NCBI BLAST+ installation documentation](https://www.ncbi.nlm.nih.gov/books/NBK52637/)
provide upstream installation details.
