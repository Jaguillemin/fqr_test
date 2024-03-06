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

# Batch Script
#!/usr/bin/env bash
#SBATCH -J run_array
#SBATCH -c 1
#SBATCH -N 1 # on one node
#SBATCH -t 6:00:00 
#SBATCH --mem 10G 
#SBATCH -o ./slurmOutput/run_array.%A_%a.out
#SBATCH -p bluemoon
#SBATCH --array=1-100

module load Rtidyverse

#reminder of args
#args = commandArgs(trailingOnly=TRUE)
#p_series=as.numeric(args[1])
#gens=as.numeric(args[2])
#pop=as.numeric(args[3])

paramfile=param.txt

p_series=$(cat ${paramfile} |  sed '1d' | awk '{print $2}' | sed "${SLURM_ARRAY_TASK_ID}q;d" )
gens=$(cat ${paramfile} |  sed '1d' | awk '{print $3}' | sed "${SLURM_ARRAY_TASK_ID}q;d" )
pop=$(cat ${paramfile} |  sed '1d' | awk '{print $4}' | sed "${SLURM_ARRAY_TASK_ID}q;d" )

Rscript --vanilla simulate.drift.R ${p_series} ${gens} ${pop}

# The r script and batch scripts are saved as files outside of the terminal

# Run the array
## quit R
sbatch --account=biol6990 # Your_script.sh
my_job_statistics <id>

# Concatenate the files
## Load libraries
R
library(tidyverse)
library(foreach, lib.loc = "/gpfs1/cl/biol6990/R_shared")

## collation
library(foreach, lib.loc = "/gpfs1/cl/biol6990/R_shared")
command = paste("ls | grep 'g_prac_6' " )
files_o = system(command, intern = TRUE )

collated.files =
foreach(i=files_o, .combine="rbind")%do%{
message(i)
o = get(load(i))
}

# Graph output
collated.files%>% 
ggplot(aes(
x=gens,
y=p,
color=as.factor(size)
)) +
geom_line() +
facet_grid(~size) ->
my_time_plot

ggsave(my_time_plot, file = "my_time_plot.pdf", w = 8, h = 4)
