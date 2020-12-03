+++
title = "Installing cdt R prerequisites in Ubuntu without root"
author = ["Chris Cundy"]
draft = false
+++

I wanted to use some of the tools from the
[causal discovery toolbox](https://github.com/FenTechSolutions/CausalDiscoveryToolbox) which require R and the `pcalg` package to be installed.
As a complete newcomer to R, it was more hassle than I thought it would
be to install R on an ubuntu server without root access.
Here's how to do it:

First install the dev version of `curl` so we have the `libcurl` headers

```shell
git clone https://github.com/curl/curl.git
cd curl
autoreconf -fi
./configure --prefix=$HOME
make && make install
```

We then add the `curl` headers to the `LD_LIBRARY_PATH`:

```shell
export LD_LIBRARY_PATH="$HOME/include:$LD_LIBRARY_PATH"
```

We then fetch the source of R and install:

```shell
wget http://cran.rstudio.com/src/base/R-3/R-3.6.0.tar.gz
tar -xzf R-3.6.0.tar.gz
cd R-3.6.0
./configure --prefix=$HOME/R
make && make install
```

and add this `R` executable to the `PATH`:

```shell
export PATH="$HOME/R/bin/:$PATH"
```

and then install the prerequisites for `pcalg`

```R
install.packages("BiocManager")
BiocManager::install(c("RBGL"))
install.packages("pcalg")
```
