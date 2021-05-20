
# missDiag

Incomplete data is a common challenge when analyzing real-world data.
Another challenge is choosing from many available multiple imputation
approaches to fill-in the missing values. Marbach (2021) suggests
adopting the imputation approach that generates a density of imputed
values most similar to those of the observed values for an incomplete
variable after balancing all other covariates. The package facilitates
the computation of discrepancy statistics summarizing differences
between the density of imputed and observed values and the construction
of weights to balance covariates.

### Installation

``` r
remotes:::install_github("sumtxt/missDiag")
```

To install the `remotes` package use `install.packages("remotes")`.

### Requirements

The package can be installed with few dependencies but to construct
weights the package relies on either the
[ebal](https://cran.r-project.org/web/packages/ebal/index.html) package
(Hainmueller, 2012) or the
[sbw](https://cran.r-project.org/web/packages/sbw/index.html) package
(Zubizarreta 2015). These two packages have be installed by the users
separately.

`ebal` can be installed from CRAN without dependencies:

``` r
remotes::install_cran("ebal")
```

The `sbw` package relies on the `spatstat` package. A recent update of
the `spatstat` deprecates functions `sbw` has been relying on. To
install the `sbw` package, one has to first install an older version of
`spatstat` and then the `sbw` package:

``` r
remotes::install_version("spatstat", "1.64-1", upgrade="never")
remotes::install_version("sbw", "1.1.1", upgrade="never")
```

### Usage

The package comes with some example data from the 2008 American National
Election Study (`anes08`) in which, for example, about 11% of the values
in the vote-choice variable are missing. The data have been imputed five
times using random value imputation (output data: `anes08_rng`) and
predictive mean matching imputation (`anes08_pmm`). We use the
`missDiag` package to corroborate that predictive mean matching produces
a density of imputed values that is more similar to the (reweighted)
density of observed values and should therefore be preferred over random
value imputation. We use entropy-balancing weights from the `ebal`
package as their computation is faster but stable weights from the `sbw`
package are preferable as their variance is smaller.

``` r
library(tidyverse)
library(missDiag)

diag_rng <- missDiag( 
 original=anes08, 
 imputed=anes08_rng, 
 adjust = 'ebal',
 formula = vote ~ .)

diag_pmm <- missDiag( 
 original=anes08, 
 imputed=anes08_pmm, 
 adjust = 'ebal',
 formula = vote ~ .)
```

Averaging across the five multiple imputed datasets, we compare the
standardized mean differences (SMD) for each of the vote choice
categories and find that predictive mean matching does much better than
random value imputation. The standardized mean differences are, on
average, lower for predictive mean matching.

``` r
diag_rng %>% group_by(vname) %>% 
    summarize( smd=mean(diff_adj) )
#> # A tibble: 3 x 2
#>   vname                 smd
#>   <chr>               <dbl>
#> 1 vote_McCain        0.0621
#> 2 vote_No vote/Other 0.0731
#> 3 vote_Obama         0.0577

diag_pmm %>% group_by(vname) %>% 
    summarize( smd=mean(diff_adj) )
#> # A tibble: 3 x 2
#>   vname                 smd
#>   <chr>               <dbl>
#> 1 vote_McCain        0.0562
#> 2 vote_No vote/Other 0.0417
#> 3 vote_Obama         0.0512
```

### References

Moritz Marbach. 2021. Choosing Imputation Models.

José R Zubizarreta. 2015. [Stable Weights that Balance Covariates for
Estimation with Incomplete Outcome
Data](https://doi.org/10.1080/01621459.2015.1023805), Journal of the
American Statistical Association, 110(511): 910-922.

Jens Hainmueller. 2012. [Entropy Balancing for Causal Effects: A
Multivariate Reweighting Method to Produce Balanced Samples in
Observational Studies](https://doi.org/10.1093/pan/mpr025), Political
Analysis 20(1): 25–46.
