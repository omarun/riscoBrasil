
riscoBrasil
-----------

riscoBrasil is a (tiny) R package to load the 'Brazil Risk' data that the Instituto Brasileiro de Geografia e Estatística [IBGE](http://www.ibge.gov.br/english/) maintain publicly from J.P. Morgan's Emerging Markets Bond Index.

### Installation

``` r
if(!require(devtools)) install.packages("devtools")
#> Loading required package: devtools
devtools::install_github("RobertMyles/riscoBrasil")
#> Downloading GitHub repo RobertMyles/riscoBrasil@master
#> from URL https://api.github.com/repos/RobertMyles/riscoBrasil/zipball/master
#> Installing riscoBrasil
#> '/Library/Frameworks/R.framework/Resources/bin/R' --no-site-file  \
#>   --no-environ --no-save --no-restore --quiet CMD INSTALL  \
#>   '/private/var/folders/j5/dg53sh6s5lz3v80h_d8vl9lm0000gn/T/RtmpL39MEz/devtools155c662d801ed/RobertMyles-riscoBrasil-539ca8b'  \
#>   --library='/Library/Frameworks/R.framework/Versions/3.3/Resources/library'  \
#>   --install-tests
#> 
```

### Usage

The package has one function, `riscoBrasil()`, which takes two optional arguments, `start` and `end`. Without specifying either of these two, `riscoBrasil()` returns a data frame with data going back to 1994. The data frame has two columns, 'date' and 'risk', the first being POSIXct, the second being a numeric column. A specific period may be requested with `start` and/or `end`. If these parameters are used, they must be in a certain format, as a character string. For those familiar with the [lubridate](https://github.com/hadley/lubridate) package, the format is "Ymd", for those not familiar with this style, the 1st of March 2017 is specified as: "2017-03-01". Both `start`and `end` must be in this format, otherwise the function will stop and return an error.

    riscoBrasil <- function(start = NULL, end = NULL)

These data are then ideal for time series analysis, particularly with ARIMA-style models that posit that the values of a time-series are explained mostly by their past values.

### Example

Here is how we can download and plot the series starting from May 2007:

``` r
library(riscoBrasil)
series <- riscoBrasil(start = "2007-05-01")

library(ggplot2)
ggplot(series, aes(x = date, y = risk)) + 
  geom_line(colour = "#1874CD") +
  theme_classic()
```

![](http://i.imgur.com/zDL8cZx.png)

The 'Brazil Risk' numbers, [published daily](http://www.ipeadata.gov.br/ExibeSerie.aspx?serid=40940&module=M) by the [Brazilian Institute of Geography and Statistics](http://www.ibge.gov.br/english/) from JP Morgan's [Emerging Market Bond Index](https://en.wikipedia.org/wiki/JPMorgan_EMBI), are a time-series spanning 1994 to the present, and as such, are a great source of data to use for the teaching of time series in R.

Forecasting with riscoBrasil
----------------------------

The data returned from `riscoBrasil()` can be used with a variety of packages and functions. In this example, I'll use facebook's new prophet package.

``` r
if(!require(prophet)) install.packages("prophet", 
                                       repos = "http://cran.us.r-project.org")
#> Loading required package: prophet
#> Loading required package: Rcpp
library(prophet)
library(riscoBrasil)

series <- riscoBrasil(start = "2002-01-01")

colnames(series) <- c("ds", "y")   # for prophet to work
model <- prophet(series)
#> Initial log joint probability = -37.3105
#> Optimization terminated normally: 
#>   Convergence detected: relative gradient magnitude is below tolerance
future <- make_future_dataframe(model, periods = 365)
forecast <- predict(model, future)
plot(model, forecast)
```

![](README-unnamed-chunk-4-1.png)
