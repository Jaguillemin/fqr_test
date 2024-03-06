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

# R-script
## Script name: simulate.drift.R
## Purpose of script: FQR practicum 5
## Author: Jackie, Emily, Jake
## Date Created: 2024-01-07
## ---------------------------
## Notes: For classroom use!
## ---------------------------
### Load libraries
library(tidyverse)
library(foreach, lib.loc = "/gpfs1/cl/biol6990/R_shared")
#debugging units
#t=100
#p0=0.5

#Lets the script know where items are in the parameter file
args = commandArgs(trailingOnly=TRUE)
p_series=as.numeric(args[1])
gens=as.numeric(args[2])
pop=as.numeric(args[3])

drift_func = function(t, p0, n){
freqs = as.numeric()
		for( i in 1:t){
			if(i == 1){
			Sampler=rbinom(1,n, p0)
			p1=Sampler/n
			freqs[i] = p1
					} else if(i > 1){
					Sampler=rbinom(1,n, freqs[i-1])
					p1=Sampler/n
					freqs[i] = p1
}
}
return(freqs)
}
time_series=
foreach(n = c(2*5, 2*25, 2*50, 2*500, 2*5000), .combine = "rbind")%do%{
p_out=drift_func(gens,p_series,n)
data.frame(p=freqs, gens = 1:t, size = n, simulations=paste(“s”,gens,p_series,n, sep = “.”) )
save(,file = paste("g_prac_6",p_series,gens,pop, "Rdata", sep ="."))
}
