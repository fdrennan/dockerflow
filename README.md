
<!-- README.md is generated from README.Rmd. Please edit that file -->

# dockerflow

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![CRAN
status](https://www.r-pkg.org/badges/version/dockerflow)](https://CRAN.R-project.org/package=dockerflow)
<!-- badges: end -->

The goal of dockerflow is to quickly generate containers for deployment.

## Installation

You can install the released version of dockerflow from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("dockerflow")
```

## Example

This is a basic example which shows you how to solve a common problem:

[Base
Images](https://github.com/rocker-org/rocker/blob/master/README.md)

# Building the JSON File

``` r
library(dockerflow)
```

``` r
json_data <- 
  build_me_docker(name = 'DockerPlumber',
                  version = 'rocker/shiny:4.0.1',
                  packages = c('shiny'),
                  localpath_vector = 'app.R')
```

``` r
json_data
#> $json_path
#> .dockerflow.DockerPlumber.json
#> 
#> $container
#> [
#>     {
#>         "base": [
#>             "rocker/shiny:4.0.1"
#>         ],
#>         "name": [
#>             "DockerPlumber"
#>         ],
#>         "workdir": [
#>             "/"
#>         ]
#>     },
#>     {
#>         "apt_get": [
#>             "sudo",
#>             "gdebi-core",
#>             "pandoc",
#>             "pandoc-citeproc",
#>             "libcurl4-gnutls-dev",
#>             "libcairo2-dev",
#>             "libxt-dev",
#>             "xtail",
#>             "wget",
#>             "libssl-dev",
#>             "libxml2-dev",
#>             "python3-venv",
#>             "libpq-dev",
#>             "libsodium-dev",
#>             "libudunits2-dev",
#>             "libgdal-dev",
#>             "systemctl",
#>             "git",
#>             "libssh2-1",
#>             "libssh2-1-dev",
#>             "unzip",
#>             "curl"
#>         ]
#>     },
#>     {
#>         "packages": [
#>             "shiny"
#>         ],
#>         "renv": {
#> 
#>         }
#>     },
#>     [
#>         "app.R"
#>     ]
#> ]
#> 
```

# Reading the JSON File

``` r
build_container(dockerflow_path = json_data$json_path, build = TRUE)
#> # DockerPlumber
#> FROM rocker/shiny:4.0.1
#> WORKDIR /
#> RUN apt-get update --allow-releaseinfo-change -qq && apt-get install -y  \
#>  sudo \
#>  gdebi-core \
#>  pandoc \
#>  pandoc-citeproc \
#>  libcurl4-gnutls-dev \
#>  libcairo2-dev \
#>  libxt-dev \
#>  xtail \
#>  wget \
#>  libssl-dev \
#>  libxml2-dev \
#>  python3-venv \
#>  libpq-dev \
#>  libsodium-dev \
#>  libudunits2-dev \
#>  libgdal-dev \
#>  systemctl \
#>  git \
#>  libssh2-1 \
#>  libssh2-1-dev \
#>  unzip \
#>  curl
#> RUN R -e "install.packages('shiny', dependencies = TRUE)"
#> COPY ./app.R ./app.R
#> please run tail -f .dockerfiles.txt to follow the installation
#> docker build -t dockerplumber --file ./DockerPlumber . >> .dockerfiles.txt
```
