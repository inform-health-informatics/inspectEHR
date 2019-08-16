
<!-- README.md is generated from README.Rmd. Please edit that file -->
inspectEHR <a href='https://cc-hic.github.io/inspectEHR/'><img src='man/figures/logo.png' align="right" height="139" /></a>
===========================================================================================================================

<!-- badges: start -->
<!-- badges: end -->
Overview
--------

inspectEHR is a data wrangling, cleaning and reporting tool for CC-HIC. It is designed to run with the CC-HIC EAV table structure (which at present exists in PostgreSQL and SQLite flavours). We are about to undergo a major rewrite to a OHDSI CDM version 6, so this package will be in flux. Once these functions have been ported across and tested, we will aim to submit to CRAN. Please see the `R` vignettes for further details on how to use the package to perform the most common data wrangling tasks:

-   `extract_demographics()` produces a table for time invariant variables
-   `extract_timevarying()` produces a table for longitudinal variables
-   `clean()` cleans the above tables according to pre-defined standards.
-   `report()` produces a data quality report (note, being fixed for IDHS usage)
-   `shiny_report()` starts an interactive shiny session for the data quality reports (under development)

Installation
------------

``` r
# install directly from github with
library(devtools)
install_github("cc-hic/inspectEHR")
```

A copy should already be installed into the group library for the CC-HIC team inside the UCL safe haven. If you are having problems with this, please contact me directly.

Usage
-----

A synthetic database ships with inspectEHR for you to explore. Actual values are essentially garbage, but everything is internally consistent.

``` r
library(inspectEHR); library(dbplyr)

# Synthetic database ships with inspectEHR
ctn <- connect(sqlite_file = "./data-raw/synthetic_db.sqlite3")

# Extract static variables. Rename on the fly.
dtb <- extract_demographics(
  connection = ctn,
  code_names = c("NIHR_HIC_ICU_0017"),
  rename = c("height")
)

head(dtb)
#> # A tibble: 6 x 2
#>   episode_id height
#>        <int>  <dbl>
#> 1          1   152.
#> 2          2   185.
#> 3          3   167.
#> 4          4   168.
#> 5          5   139.
#> # … with 1 more row

# Extract time varying variables. Rename on the fly.
# Pull out to any arbitrary temporal resolution and custom define the
# behaviour for information recorded at resolution higher than you are sampling
ltb <- extract_timevarying(
  connection = ctn,
  code_names =  "NIHR_HIC_ICU_0108", rename = "hr"
)
#> [1] "0.0049 hours to process"

head(ltb)
#> # A tibble: 6 x 3
#>    time    hr episode_id
#>   <dbl> <int>      <int>
#> 1     0    85          1
#> 2     1    71          1
#> 3     2    95          1
#> 4     3    82          1
#> 5     4    72          1
#> # … with 1 more row
```

Getting help
------------

If you find a bug, please file a minimal reproducible example on [github](https://github.com/cc-hic/inspectEHR/issues).

Reporting Data Quality Issues
-----------------------------

Please submit an issue and tag it with "data quality". Data quality issues are often related to site in specific. If this is the case, please also tag the site.

Data Quality Rules
------------------

The data quality rules are largely based upon standards set by OHDSI<sup>1</sup>, and Khan et al.<sup>2</sup>. The CC-HIC currently uses an episode centric model. As such, many of the data quality checks are based around this way of thinking. As we move to OMOP (a patient centric model) many of these will change accordingly.

### Episode Characterisation

-   Internal consistency check of episode
-   Each episode has a unique identifier
-   Each episode has a start date
-   Each episode has an end date defined by one of:
    -   End of episode date time (often administrative in nature and used with caution)
    -   On unit death date time from any means
    -   Death
    -   Brain stem Death
    -   "Body Removal": strongly suggesting death
    -   The last measured bedside physiological result (HR and SpO2)

Records that are flagged as "open" are dropped.

### Event Characterisation

-   Events occur within their episode (within a reasonable buffer time)
-   Events fall within range validation checks
-   Events are not duplicates
-   Events are contributed over time in consistent manner

### Event Restrictions

-   Co-occurrence events:
    -   Systolic &gt; MAP &gt; Diastolic
    -   BMI (derived from height and weight) in plausible range
    -   Organ support days cannot exceed episode duration

### Statistical Evaluations

-   Events that are known to follow a particular distribution, conform to this distribution.
-   Kolmogorov-Smirnov testing checks for major distributional deviation between contributing sites. Deviations are often either due to a data quality issue, or a major shift in case mix.

Synthetic Database
------------------

There is a copy of the CC-HIC database located in `data-raw/synthetic_db.sqlite3`. This is a structurally sound copy of the real database, but entirely hand crafted (so there is no patient data here). It's quite sparse at present, but I will add more variables as time goes on. For now, it acts as a test database to make sure inspectEHR works as it should.

------------------------------------------------------------------------

1.  <https://www.ohdsi.org/analytic-tools/achilles-for-data-characterization/>
2.  Kahn, Michael G.; Callahan, Tiffany J.; Barnard, Juliana; Bauck, Alan E.; Brown, Jeff; Davidson, Bruce N.; Estiri, Hossein; Goerg, Carsten; Holve, Erin; Johnson, Steven G.; Liaw, Siaw-Teng; Hamilton-Lopez, Marianne; Meeker, Daniella; Ong, Toan C.; Ryan, Patrick; Shang, Ning; Weiskopf, Nicole G.; Weng, Chunhua; Zozus, Meredith N.; and Schilling, Lisa (2016) "A Harmonized Data Quality Assessment Terminology and Framework for the Secondary Use of Electronic Health Record Data," eGEMs (Generating Evidence & Methods to improve patient outcomes): Vol. 4: Iss. 1, Article 18.
