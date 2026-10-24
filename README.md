# AEDS6400_blog6
# Blog Post 6: Matching Parents and Children with SQL in Python

**Computational Methods for Economists — Blog Post 6**

## 1. Project Overview

This project explores **intergenerational income mobility** using longitudinal household data from the Panel Study of Income Dynamics (PSID).

The main objective is to demonstrate how SQL queries executed within Python can identify parent–child relationships, connect individuals to household income records, and construct income measures across multiple survey waves.

The tutorial addresses three questions:

1. How can SQL be used to construct parent–child matches from longitudinal data?
2. How does estimating intergenerational income elasticity (IGE) using a single year differ from using income averaged across multiple waves?
3. How might the age at which income is measured influence estimated intergenerational income persistence?

The project combines **Python, SQLite, pandas, and statsmodels**. It is designed as a reproducible computational tutorial rather than a causal analysis of intergenerational mobility.

**Blog post:** [From Family Records to Intergenerational Mobility](blog6.qmd)

## 2. Data Sources

The analysis uses two datasets obtained from the University of Michigan's PSID.

### 2.1 PSID Longitudinal Data

**Official website:** https://psidonline.isr.umich.edu/

**Data Center:** https://simba.isr.umich.edu/

The dataset contains individual identifiers, demographic information, family interview numbers, and household income measures.

Four survey waves are used:

- 2017
- 2019
- 2021
- 2023

Income measures refer to the preceding calendar years (2016, 2018, 2020, and 2022).

The following variables are required:

| Variable | Description |
|---|---|
| ER30001 | 1968 Interview Number |
| ER30002 | Person Number |
| ER34501 / ER34701 / ER34901 / ER35101 | Annual individual family interview numbers |
| ER34504 / ER34704 / ER34904 / ER35104 | Individual age in each survey wave |
| ER71426 | Total Family Income, 2016 |
| ER77448 | Total Family Income, 2018 |
| ER81775 | Total Family Income, 2020 |
| ER85629 | Total Family Income, 2022 |

Annual family interview identifiers and sequence numbers are also used to construct and validate the longitudinal records.

Variable definitions should be checked against the official PSID Codebook.

### 2.2 Family Identification Mapping System (FIMS)

**Official website:** https://simba.isr.umich.edu/FIMS/

The FIMS file identifies relationships between parents and children, including individuals who no longer live in the same household.

The analysis uses a prospective, balanced parent-to-child mapping file with the following identifiers:

| Variable | Description |
|---|---|
| G1ID68 | Parent's 1968 Interview Number |
| G1PN | Parent's Person Number |
| G2ID68 | Child's 1968 Interview Number |
| G2PN | Child's Person Number |

These identifiers allow parents and children to be matched through SQL, even when their current household interview numbers differ.

## 3. How to Download the Data

### Step 1: Download PSID variables

1. Open the PSID Data Center and register or sign in.
2. Select **Variable Search**.
3. Choose the survey years 2017, 2019, 2021, and 2023.
4. Under *PSID Individual-level*, select permanent identifiers, individual age, family interview number, and sequence number.
5. Under *PSID Family-level*, select total family income and yearly family interview identifiers.
6. Verify variable labels using the Codebook.
7. Add the variables to the Data Cart.
8. Generate and download the customized dataset in Excel format.

### Step 2: Download parent–child relationships

1. Visit the FIMS website.
2. Select an intergenerational prospective parent-to-child mapping.
3. Choose the balanced mapping option.
4. Export the relationship file in long-format Excel form.
5. Store it locally alongside the PSID data extract.

The main analysis uses the downloaded Excel files locally. The raw data are not included in this GitHub repository.

## 4. Repository Structure

The public-facing project uses the following organization:

```text
post6/
├── README.md
├── blog6.qmd
├── blog6.ipynb
├── figure1.png
├── figure2.png
└── figure3.png
```

**File descriptions**

| File | Description |
|---|---|
| `README.md` | Project overview and reproduction instructions |
| `blog6.qmd` | Quarto source of the blog post |
| `blog6.ipynb` | Python notebook containing the SQL and statistical workflow |
| `figure1.png` | Parent–child log-income association |
| `figure2.png` | Single-wave versus four-wave IGE |
| `figure3.png` | Age-based sensitivity analysis |

The figure names correspond to their appearance in the blog post, not necessarily to the order in which they were generated.

## 5. Software Requirements

The analysis was developed in Python and uses these packages:

- `pandas` — data manipulation
- `numpy` — numerical calculations
- `sqlite3` — SQL database operations, included with Python
- `statsmodels` — regression estimation
- `matplotlib` — figures
- `python-calamine` — reading Excel files
- `openpyxl` — optional Excel support

Install the required packages in a local Python environment:

```bash
python -m pip install pandas numpy matplotlib statsmodels python-calamine openpyxl jupyter
```

The notebook uses SQLite through Python's built-in `sqlite3` module. A separate SQL server is not required.

## 6. Reproducing the Analysis

### Step 1: Prepare the local environment

Download the authorized PSID and FIMS files and place them in a private local working directory.

Download `blog6.ipynb` from the repository and open it in Jupyter Notebook, JupyterLab, or another compatible Python environment.

### Step 2: Configure the input files

Locate the input-file settings in the notebook:

```python
DATA_FILE = Path("J366641.xlsx")
FIMS_FILE = Path("fim16231_gidpro_BO_2_BAL.xlsx")
OUTPUT = Path("blog6_results")
```

These names correspond to the downloaded files used in the original analysis. Replace them with the actual local filenames if necessary.

The notebook can use the `calamine` engine when reading Excel files:

```python
raw = pd.read_excel(DATA_FILE, engine="calamine")
fims = pd.read_excel(FIMS_FILE, engine="calamine")
```

This avoids an Excel worksheet-property compatibility issue encountered with `openpyxl` during development.

### Step 3: Run the analysis

Execute the notebook cells in order.

The workflow consists of five main stages.

**Stage A — Data preparation**

Load the PSID and FIMS files, check variable names, and transform the four-wave dataset from wide format into longitudinal records.

**Stage B — SQL matching**

Create in-memory SQLite tables for individual records, household income, and parent–child relationships.

Use:

- `SELECT` to retrieve relevant observations.
- `JOIN` to associate individuals with their households and link parents to children.
- `WHERE` to apply eligibility conditions.

The analysis excludes nonpositive income observations, applies age restrictions, and excludes records where parents and adult children share the same household income.

**Stage C — Multi-wave income construction**

Use `GROUP BY`, `AVG()`, and `COUNT(DISTINCT year)` to summarize repeated income observations for each matched pair.

The full four-wave sample is used to compare single-wave and averaged-income models consistently.

**Stage D — Regression analysis**

Estimate log–log IGE with ordinary least squares and heteroskedasticity-robust standard errors:

\[
\log(Y_{\mathrm{child}})
=
\beta_0+
\beta_1\log(Y_{\mathrm{parent}})
+\varepsilon
\]

A rank–rank regression is also estimated as a supplementary measure of income persistence.

**Stage E — Age sensitivity and visualization**

Divide parents into three average-age groups: under 55, 55–64, and 65 or older.

Estimate group-specific IGE values and generate the figures.

### Step 4: Verify the output

The main analysis creates local outputs in `blog6_results/`, including:

- `matching_diagnostics.csv`
- `regression_results.csv`
- `figure_ige_comparison.png`
- `figure_loglog_income.png`

The supplementary age-sensitivity code also produces `age_sensitivity_results.csv` and `figure_age_sensitivity.png`.

For the published blog, the corresponding figures are saved or copied as:

| Analysis output | Blog image |
|---|---|
| `figure_loglog_income.png` | `figure1.png` |
| `figure_ige_comparison.png` | `figure2.png` |
| `figure_age_sensitivity.png` | `figure3.png` |

These three numbered images are the files referenced by `blog6.qmd`.

## 7. Main Results

The final comparison uses **2,074 matched parent–child pairs** observed across all four survey waves.

| Income measurement | Estimated IGE |
|---|---:|
| 2017 | 0.252 |
| 2019 | 0.207 |
| 2021 | 0.245 |
| 2023 | 0.261 |
| Four-wave average | **0.318** |

The four-wave income measure produces a higher estimated IGE than any individual survey wave in this sample.

The age-based sensitivity analysis yields:

| Parental age | Observations | Estimated IGE |
|---|---:|---:|
| Under 55 | 228 | 0.147 |
| 55–64 | 913 | 0.296 |
| 65+ | 933 | 0.337 |

These results illustrate that measured intergenerational income persistence differs by income measurement window and parental age group.

However, the analysis is descriptive. Age-group differences should not be interpreted as causal evidence of life-cycle bias.

## 8. Methodological Limitations

Several limitations are important when interpreting the findings.

**Household versus individual income:** The income variable measures total household income, not individual earnings. Consequently, the estimated elasticity describes the association between parental and adult-child household incomes.

**Life-cycle differences:** Parents and children are observed at different stages of their lives. Averaging four survey waves does not completely eliminate life-cycle measurement concerns.

**Sample selection:** Restricting the sample to valid, independently observed households with complete income records may affect representativeness.

**Parent selection:** The analysis retains one parent per child using a deterministic identifier-based rule. This simplifies the matching structure but does not construct a combined parental-income measure.

**Inflation:** Income observations come from different calendar years. The current baseline analysis should not be interpreted as using inflation-adjusted permanent incomes unless an explicit price adjustment is implemented.

**Statistical interpretation:** Differences between estimated IGE coefficients are descriptive and do not automatically imply statistically significant differences.

## 9. Data Access and Reproducibility

PSID data and customized extracts are not redistributed in this repository.

Readers should download the necessary data directly from the official PSID and FIMS websites, comply with the applicable data-use conditions, and perform the analysis in an authorized local environment.

The repository provides the methodological workflow, variable information, SQL operations, and public-facing figures. Reproduction requires access to the relevant data and any permissions required for its use.

The published figures and reported statistics correspond to the analysis outputs described here.

## 10. References

- University of Michigan. [Panel Study of Income Dynamics](https://psidonline.isr.umich.edu/)
- [PSID Data Center](https://simba.isr.umich.edu/)
- [Family Identification Mapping System](https://simba.isr.umich.edu/FIMS/)
- [PSID Documentation and Codebooks](https://psidonline.isr.umich.edu/)

---

**Project:** Computational Methods for Economists — Blog Post 6

**Author:** Yixin Yang