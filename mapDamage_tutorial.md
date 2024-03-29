#  mapDamage tutorial Tutorial

Read about the program, its output, and follow instructions for setup in the [mapDamaga's main page](http://ginolhac.github.io/mapDamage/)


## Start here

Download mapDamage from git into desired directory 
```sh
git clone https://github.com/ginolhac/mapDamage.git
```

In my current working station, I would then get a working node
```sh
salloc
```

Load mapDamage according to your system. Example using a container:
```sh
enable_lmod
module load container_env mapdamage2
```

mapDamage creates plots using R, which you should make sure to install and load in your current version before running mapDamage

Load required R libraries in your current session
```sh
crun R		# Accessing R inside the container in my system
R			# without a container 

# Load required libraries
library("inline")
library("gam")
library("Rcpp")
library("ggplot2")
library("RcppGSL")
```

You can check that they were loaded by
```sh
sessionInfo()
````

Quit R and save the session
```sh
quit()
y
```

Run mapDamage (don't use `crun` if not using a container) while specifying a BAM (or SAM) file and the corresponding reference in fasta format using the `-i` and `-r` flags, respectively. Example:
```sh
crun mapDamage -i ../../files -r ../../reference.fasta
```
**Use relative paths as necessary**

This creates a directory with results named: results<name_of_bam_file>

A successful run delivers a message similar to this:
```sh
Started with the command: /opt/mapdamage2/bin/mapDamage -i ../../PIRE2019-Ssp-A-Atu_010-Plate1Pool5Seq1-1E-L4.5.5-RAW.bam -r ../../reference.5.5.fasta
WARNING: Alignment contains a large number of reference sequences (28525)!
  This may lead to excessive memory/disk usage.
  Consider using --merge-reference-sequences

	Reading from '../../PIRE2019-Ssp-A-Atu_010-Plate1Pool5Seq1-1E-L4.5.5-RAW.bam'
	Writing results to 'results_PIRE2019-Ssp-A-Atu_010-Plate1Pool5Seq1-1E-L4.5.5-RAW/'
pdf results_PIRE2019-Ssp-A-Atu_010-Plate1Pool5Seq1-1E-L4.5.5-RAW/Fragmisincorporation_plot.pdf generated
No length distributions are available, plotting length distribution only works for single-end reads
Warning: DNA damage levels are too low, the Bayesian computation should not be performed (0.001559 < 0.01)

Performing Bayesian estimates
Warning, To few substitutions to assess the nick frequency, using constant nick frequency instead
Starting grid search, starting from random values
Adjusting the proposal variance iteration  1  
Adjusting the proposal variance iteration  2  
Adjusting the proposal variance iteration  3  
Adjusting the proposal variance iteration  4  
Adjusting the proposal variance iteration  5  
Adjusting the proposal variance iteration  6  
Adjusting the proposal variance iteration  7  
Adjusting the proposal variance iteration  8  
Adjusting the proposal variance iteration  9  
Adjusting the proposal variance iteration  10  
Done burning, starting the iterations
Done with the iterations, finishing up
Writing and plotting to files
Warning messages:
1: package ‘inline’ was built under R version 4.0.2 
2: package ‘gam’ was built under R version 4.0.2 
Successful run
```

You can then created a sbatch script to run all albatross files in parallel. For example, `mapDamage_loop.sh`:
```sh
#!/bin/bash -l

#SBATCH --job-name=mapDamage
#SBATCH -o mapDamage-%j.out
#SBATCH -p main
#SBATCH -c 4
#SBATCH --mail-user=youremail
#SBATCH --mail-type=begin
#SBATCH --mail-type=END

enable_lmod
module load container_env mapdamage2

ls ../../PIRE2019-Ssp-A*bam | parallel --no-notice -kj40 "crun mapDamage -i{} -r ../../reference.5.5.fasta"

# Running a few jobs with contemporary files for comparisons as well

crun mapDamage -i../../PIRE2019-Ssp-C-Gub_002-Plate1Pool3Seq1-2D-L4.5.5-RAW.bam -r ../../reference.5.5.fasta
crun mapDamage -i../../PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW.bam -r ../../reference.5.5.fasta
```

**Don't forget you load R and required libraries before running a script like this**

*out message*
```sh
crun mapDamage -i../../PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW.bam -r ../../reference.5.5.fasta
Started with the command: /opt/mapdamage2/bin/mapDamage -i../../PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW.bam -r ../../reference.5.5.fasta
WARNING: Alignment contains a large number of reference sequences (28525)!
  This may lead to excessive memory/disk usage.
  Consider using --merge-reference-sequences

	Reading from '../../PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW.bam'
	Writing results to 'results_PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW/'
pdf results_PIRE2019-Ssp-C-Gub_005-Plate1Pool12Seq1-3E-L4.5.5-RAW/Fragmisincorporation_plot.pdf generated
No length distributions are available, plotting length distribution only works for single-end reads
Warning: DNA damage levels are too low, the Bayesian computation should not be performed (0.005677 < 0.01)

Performing Bayesian estimates
Warning, To few substitutions to assess the nick frequency, using constant nick frequency instead
Starting grid search, starting from random values
Adjusting the proposal variance iteration  1  
Adjusting the proposal variance iteration  2  
Adjusting the proposal variance iteration  3  
Adjusting the proposal variance iteration  4  
Adjusting the proposal variance iteration  5  
Adjusting the proposal variance iteration  6  
Adjusting the proposal variance iteration  7  
Adjusting the proposal variance iteration  8  
Adjusting the proposal variance iteration  9  
Adjusting the proposal variance iteration  10  
Done burning, starting the iterations
Done with the iterations, finishing up
Writing and plotting to files
Warning messages:
1: package ‘inline’ was built under R version 4.0.2 
2: package ‘gam’ was built under R version 4.0.2 
Successful run

```
