# fqr_test

# Make Parameter File
## Load R and libraries

module load Rtidyverse
R
library(tidyverse)
library(foreach, lib.loc = "/gpfs1/cl/biol6990/R_shared")

# Loop for parameters
o = foreach(i = seq(from=0.1, to=0.5, by=0.1), .combine = "rbind")%do%{ #First loop, Allele Freq (p_series)
foreach(k = c(10, 50, 100, 1000, 10000), .combine = "rbind")%do%{ #Second Loop, population size (pops)
foreach(j = seq(from=10, to=100, by=1), .combine = "rbind")%do%{  #third loop, Generations (gens)
data.frame(gens=j, pops= k, p_series= i) } #close first loop
 } #close second loop
 }#close third loop
 # Make a parameter file from the loop output
 write.table(o, file = "param.txt")
