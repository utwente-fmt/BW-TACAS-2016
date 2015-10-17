Bandwidth and Wavefront Reduction for Static Variable Ordering in Symbolic Model Checking
================
Instructions for installing LTSmin and replicating the experiments reported in our TACAS 2016 submission.

__Warning__: this is a large repository to check out. It contains ~2GB of raw uncrompressed data from our benchmark and 361 Petri net models in `pnml` format, from the 2015 model checking contest: http://mcc.lip6.fr.

This readme first instructs how to replicate experiments, then it shows how to perform the data analysis on our benchmarks. The third section in this readme contains instructions for converting dependency matrices into scatter plots, to make them human readable.

1. Reproducing experiments
-
Prerequisites
--
The requirements for reproducing our experiments are the following.
- Linux OS: we tested on Ubuntu 14.02 LTS, OS X should work as well.
- Python3: for converting PNML to C.
- C/C++ compiler: we tested with GCC 4.8.
- pnml2pins: the PNML language front-end for LTSmin.
- Boost: http://www.boost.org
- ViennaCL: http://viennacl.sourceforge.net
- LTSmin: the model checker
- Memtime: for measuring memory and time

Installing LTSmin
--
Manual installation instructions can be found at http://fmt.cs.utwente.nl/tools/ltsmin. Additionally, for the TACAS 2016 submission, LTSmin requires two extra dependencies; Boost and ViennaCL. The dependencies should be automatically recognized by autoconf, when installed. Make sure to check out our specific LTSmin *tag* for the TACAS 2016 submission: https://github.com/Meijuh/ltsmin/releases/tag/BW-TACAS-2016.
Also make sure to provide the option `--recursive` to `git clone`. After cloning the repository be sure to checkout our *tag*: `git checkout tags/BW-TACAS-2016`.

pnml2pins
---
pnml2pins is the PNML language front-end for LTSmin.
Prerequisites and manual installation instructions can be found at: https://github.com/utwente-fmt/pnml2pins.

Memtime
---
We use memtime for measuring time and memory usage.
The manual installation instructions are the regular commands for autotools:
 - `git clone http://fmt.ewi.utwente.nl/gitweb/?p=memtime.git`
 - `cd memtime`
 - `autoreconf -i`
 - `./configure`
 - `make`
 - `make install`

Running experiments
--

To run an experiment, pick a Petri net from the `pnml` folder, say `Vasy2003.pnml`.

1. Compile the Petri net to a shared library: `pnml2pins Vasy2003.pnml`.
2. Pick a particular option to run LTSmin from `statistics.txt` or `peformance.txt`. The first file lists all options to gather statistics. The latter lists all options to measure performance.
3. Say we want to run the `bi,Sloan,hf` category and measure performance, use the following command `pins2lts-sym Vasy2003.so --when --vset=lddmc --lace-workers=1 --lddmc-cachesize=26 --lddmc-tablesize=26 --lddmc-maxtablesize=26 --lddmc-maxcachesize=26 --order=chain --next-union --saturation=sat-like --no-matrix -rsw,bg,bs,hf`. This should give similar output as the file `data/0/1__pins2lts-sym_--order=chain---next-union---saturation=sat-like---no-matrix--rsw,bg,bs,hf_Vasy2003.so`. Note that all log files from LTSmin are prefixed with 0...5. Files beginning with zero contain statistics, files beginning with 1..5 show peformance.
4. To measure time and memory, use `memtime`, by prefixing the above commandline with `memtime -m4000000000 -c1800`. This limits memory usage to 4GB and 30 minutes.

The command lines in `statistics.txt` and `performance.txt` show several options, of which some are fixed. The fixed options are:

- `--when`: print timestamps
- `--vset=lddmc`: use the LDDMC package for decision diagrams
- `--lace-workers=1`: run with 1 thread
- `--lddmc-cachesize=26`: use an operation cache with 2^26 elements
- `--lddmc-tablesize=26`: use a node table with 2^26 elements
- `--lddmc-maxcachesize=26`: don't increase the operation cache beyond 2^26 elements
- `--lddmc-maxtablesize=26`: don't increase the node table beyond 2^26 elements
- `--order=chain`: use chaining for grouped transitions
- `--next-union`: perform the identity on transition relations
- `--saturation=sat-like`: use the sat-like strategry
- `--no-matrix`: don't print the dependency matrix
- `--peak-nodes`: measure peak nodes (used for statistics)
- `--graph-metrics`: Print the graph metrics from Boost and ViennaCL (used for statistics)
- `<model>`: The compiled Petri net model
- `-r<regroup-option>`: the regrouping option that transforms the dependency matrix

The value `<regroup-option>` is a sequence of transformations/operations on the dependency matrix. Each transformation/operation is separated by a `,`. The list of transformations/operations are as follows.

- `sw`: Select Write matrix
- `sc`: Select Combined matrix
- `sr`: Select Read matrix
- `bg`: Use the Bipartite Graph
- `tg`: Use the Total Graph
- `bcm`: Apply Boost's Cuthill Mckee algorithm
- `bk`: Apply Boost's King algorithm
- `bs`: Apply Boost's Sloan algorithm
- `vcm`: Apply Viennacl's Cuthill Mckee algorithm
- `vacm`: Apply Viennacl's Advanced Cuthill Mckee algorithm
- `vgps`: Apply Viennacl's Gibbs Poole Stockmeyer algorithm
- `cw,rs`: Apply the Column Swap algorithm
- `mm`: Print the Matrix Metrics (used for statistics)
- `hf`: Perform a Horizontal Flip
- `vf`: Perform a Vertical Flip

Miscellaneous
--

- The running example in our paper, is in the file `example.pnml`. To obtain the same permuted matrix (and metrics) use `pnml2pins example.pnml && pins2lts-sym example.so --row-perm=1,2,0,5,3,4 --col-perm=1,2,3,4,0 --matrix -rmm`.
- To reorder dependency matrices using the Approximate Minimum Degree algorithm, checkout the tag: https://github.com/Meijuh/ltsmin/releases/tag/BW-TACAS-2016-SuiteSparse and supply the option `-rsamd` to `pins2lts-sym`. In this tag, autoconf does not warn if SuiteSparse is not installed. Thus before compiling this LTSmin tag, make sure to have installed SuiteSparse.
- The files `pnml/TokenRing-50-unfolded.pnml`, `pnml/galloc_res-5.pnml`, `pnml/shared_memory-pt-200.pnml`, `pnml/distributeur-01-unfolded-10.pnml` are too large to put in this git repository, if needed download them manually from: http://mcc.lip6.fr/models.php.

2. Data analysis
-

Prerequisites
--
To peform the same analysis on the data set from our benchmark, R >= 3.2 is required: https://www.r-project.org.
Additionally the following R packages are required (see http://www.r-bloggers.com/installing-r-packages):
 - ggplot2
 - plyr
 - reshape2
 - scales
 - data.table
 - doMC
 - grid

Data analysis
--
The file `data.csv` contains all relevant LTSmin output from the *114798* experiments.
To analyse the data, download `data.csv` and run `Rscript analysis.r data.csv`. This produces 4 pdf files; the two graphs with *mean standard score* and the two scatter plots with *time* measurements and *peak nodes*.

Parsing LTSmin output
--
Parsing the raw stderr/stdout output from LTSmin can also be done. The folder `data` contains all the data from the *114798* experiments. There are two subfolders in `data`. The folder `0` contains all experiments that completed within 30 minutes and 4GB of memory. The folder `1` contains all experiments that timed out or ran out of memory.
To convert all LTSmin output to a `csv` file, run: `sh ltsmin2csv.sh data/0 > my-data.csv`.
__This can take a very long time__, since all files have to be parsed with `awk` and `grep`.
When completed, the above mentioned Rscript can be run on `my-data.csv`.

3. Dependency matrix 2 scatter plot
-
LTSmin can print the dependency matrix to stdout. For larger matrices these are not human readable. We offer scripts that can convert dependency matrices to scatter plots, with R. These scatter plots *are* human readable.

Prerequisites
--
For installing R and packages see Section 2 in this readme. The following extra R packages are needed.
 - cowplot
 - tools

Converting matrices with R
--
To obtain a human readable dependency matrix of, say Vasy2003.pnml, perform the following steps.

1. Download `pnml/Vasy2003.pnml`, `mtrx2csv` and `csv2mtrx.r`. Make sure `mtrx2csv` is executable (`chmod +x mtrx2csv`).
2. Convert the Petri net to a shared library: `pnml2pins Vasy2003.pnml`.
3. To generate the matrix, convert it to CSV with coordinates and then to pdf, run `pins2lts-sym Vasy2003.so --matrix | sh mtrx2csv | Rscript csv2mtrx.r Vasy2003-matrix`. This produces `Vasy2003-matrix.pdf` and can be viewed with a PDF viewer.
4. To generate a reordered matrix, say, with Boost's Sloan implementation, run `pins2lts-sym Vasy2003.so --matrix -rbs | sh mtrx2csv | Rscript csv2mtrx.r Vasy2003-matrix-bs`.
