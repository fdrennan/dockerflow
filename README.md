
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

``` r
library(dockerflow)
library(glue)
library(jsonlite)
library(readr)
library(fs)
library(purrr)
#> 
#> Attaching package: 'purrr'
#> The following object is masked from 'package:jsonlite':
#> 
#>     flatten
## basic example code

drf_base = function(name = 'Dockerfile', version='r-base:4.0.2', workdir = '/') {
  base_version <- glue('{version}')
  return(list(base = base_version, name = name, workdir = workdir))
}

drf_apt_get = function(apt_get='everything', packages = NULL) {
  
  if (apt_get == 'everything' & is.null(packages)) {
    packages <- c(
      "sudo", "gdebi-core", "pandoc", "pandoc-citeproc", 
      "libcurl4-gnutls-dev", "libcairo2-dev", "libxt-dev", 
      "xtail", "wget", "libssl-dev", "libxml2-dev", 
      "python3-venv", "libpq-dev", "libsodium-dev", 
      "libudunits2-dev", "libgdal-dev", "systemctl", 
      "git", "libssh2-1", "libssh2-1-dev",
      "unzip", "curl"
    )
  } 
  
  
  return(list(apt_get = packages))
}


drf_packages <- function(renv_location = NULL) {
  if (!is.null(renv_location)) {
    message('In renv_location')
  }
  
  packages <- c('devtools')
  
  list(packages = packages, renv = renv_location)
}

drf_packages <- function(renv_location = NULL, packages = 'devtools') {
  
  if (!is.null(renv_location)) {
    message('In renv_location')
  }
  
  list(packages = packages, renv = renv_location)
}

drf_copy <- function(localpath_vector = NULL) {
  return(localpath_vector)
}


container <- list(
  drf_base(name = 'DockerPlumber',version = 'rocker/shiny:4.0.1' ),
  drf_apt_get(),
  drf_packages(packages = c('shiny')),
  drf_copy(localpath_vector = 'app.R')
)
```

``` r
meta <- container[[1]]
json_path <-  glue('.dockerflow.{meta$name}.json')
write_file(x = prettify(toJSON(container)), path = json_path)

build_container <- function(dockerflow_path = '.dockerflow.DockerPlumber.json', build = FALSE) {
  
  config_file <- read_json(dockerflow_path)
  toJSON(config_file, pretty = TRUE)
  
  meta_data <- map(config_file[[1]], ~ unlist(.))
  apt_get <- map(config_file[[2]], ~ unlist(.))
  r_packages <- map(config_file[[3]], ~ unlist(.))
  copy_in <- map(config_file[[4]], ~ unlist(.))
  

  
  
  title <- glue('# {meta_data$name}')
  base <- glue('FROM {meta_data$base}')
  workdir <- glue('WORKDIR {meta_data$workdir}')
  base_apt <- 'RUN apt-get update --allow-releaseinfo-change -qq && apt-get install -y '
  apt_get_query <- paste(c(base_apt, apt_get$apt_get), collapse = ' \\\n\t')
  
  
  # install_renv <- 'RUN R -e "install.packages(\'renv\');renv::consent(provided=TRUE);renv::init()"'
  # install_renv <- 'RUN R -e "install.packages(\'renv\')"'
  preferred_packages <- install_packages <- map_chr(
    r_packages$packages,
    function(pkg, dependencies = TRUE) {
      glue('RUN R -e "install.packages(\'{pkg}\', dependencies = {dependencies})"')
    }
  ) 
  copy_in <- map_chr(copy_in, function(file) {
    glue('COPY ./{file} ./{file}')
  })

  
  dockername <- meta_data$name
  if(file_exists(dockername))
    file_delete(dockername)
  
  walk(
    list(title, base, workdir, apt_get_query, 
         # install_renv,
         preferred_packages, copy_in),
    function(file_line) {
      file_line <- unlist(file_line)
      file_line <- paste(file_line, '\n')
      map(file_line, message)
      map(file_line,  ~ write_file(x = ., path = dockername, append = TRUE))
    }
  )
  # write_file(x = title, path = dockername, append = TRUE)
  
  
  # docker build -t productor_api --file ./DockerfileApi .
  command_to_run <- glue('docker build -t {tolower(dockername)} --file ./{dockername} . >> .dockerfiles.txt')
  
  message('please run tail -f .dockerfiles.txt to follow the installation')
  message(command_to_run)
  if (build) {
    system(command_to_run)
  }
  

}

build_container(dockerflow_path = '.dockerflow.DockerPlumber.json', build = TRUE)
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