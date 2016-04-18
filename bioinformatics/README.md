#README.md (scripts/bioinformatics)

## Overview
This repository contains a set of scripts for use in bioinformatics work. The dependencies (and possibly languages) may vary for each script. Please refer to the individual **README** details for each script.

## Scripts
The current set of scripts includes:

* [`calculate_ani.py`](#calculate_ani): calculates whole genome similarity measures ANIb, ANIm, and TETRA, with tabular text and graphical output.
* [`draw_gd_all_core.py`](#draw_gd_all_core): example script for use of GenomeDiagram. **NOTE: script will not run as provided without modification**
* [`find_asm_snps.py`](#find_asm_snps)
* [`get_NCBI_cds_from_protein.py`](#get_NCBI_cds_from_protein): Given input of protein sequences with suitably-formatted identifiers, retrieves the corresponding coding sequence from NCBI, using Entrez.
* [`nucmer_to_crunch.py`](#nucmer_to_crunch): Convert the output of **MUMmer**'s `show-coords` package to `.crunch` format, for use with **ACT**.
* [`restrict_long_contigs.py`](#restrict_long_contigs): From a directory of FASTA files, generates a new directory of corresponding FASTA files where all sequences shorter than a specified length have been removed.
* [`run_MLST.py`](#run_mlst): carries out MLST analysis on input sequences, based on [PubMLST](http://pubmlst.org) data.
* [`run_signalp.py`](#run_signalp): splits large files and parallelises for input to `SignalP`.
* [`run_tmhmm.py`](#run_tmhmm): splits large files and parallelises for input to `TMHMM`.
* [`stitch_six_frame_stops.py`](#stitch_six_frame_stops): joins sequence files with a spacer that has start/stop codons in each reading frame.

## Script READMEs

### <a name="calculate_ani">`calculate_ani.py`</a>

***THIS SCRIPT IS DEPRECATED - PLEASE USE PYANI ([https://github.com/widdowquinn/pyani](https://github.com/widdowquinn/pyani)) INSTEAD***

This script calculates Average Nucleotide Identity (ANI) according to one of a number of alternative methods described in, e.g.

* Richter M, Rossello-Mora R (2009) Shifting the genomic gold standard for the prokaryotic species definition. Proc Natl Acad Sci USA 106: 19126-19131. doi:10.1073/pnas.0906412106. (ANI1020, ANIm, ANIb)
* Goris J, Konstantinidis KT, Klappenbach JA, Coenye T, Vandamme P, et al. (2007) DNA-DNA hybridization values and their relationship to whole-genome sequence similarities. Int J Syst Evol Micr 57: 81-91. doi:10.1099/ijs.0.64483-0.

ANI is proposed to be the appropriate *in silico* substitute for DNA-DNA 
hybridisation (DDH), and so useful for delineating species boundaries. A 
typical percentage threshold for species boundary in the literature is 95% 
ANI (e.g. Richter et al. 2009).

All ANI methods follow the basic algorithm:

- Align the genome of organism 1 against that of organism 2, and identify the matching regions
- Calculate the percentage nucleotide identity of the matching regions, as an average for all matching regions

Methods differ on: (1) what alignment algorithm is used, and the choice of parameters (this affects the aligned region boundaries); (2) what the input is for alignment (typically either fragments of fixed size, or the most complete assembly available).

* **ANIm**: uses MUMmer (NUCmer) to align the input sequences.
* **ANIb**: uses BLASTN to align 1000nt fragments of the input sequences
* **TETRA**: calculates tetranucleotide frequencies of each input sequence

This script takes as input a directory containing a set of correctly-formatted FASTA multiple sequence files. All sequences for a single organism should be contained in only one sequence file. The names of these files are used for identification, so it would be advisable to name 
them sensibly.

Output is written to a named directory. The output files differ depending on the chosen ANI method.

* **ANIm**: MUMmer/NUCmer .delta files, describing the sequence alignment; tab-separated format plain text tables describing total alignment lengths, and total alignment percentage identity
* **ANIb**: FASTA sequences describing 1000nt fragments of each input sequence; BLAST nucleotide databases - one for each set of fragments; and BLASTN output files (tab-separated tabular format plain text) - one for each pairwise comparison of input sequences. There are potentially a lot of intermediate files.
* **TETRA**: Tab-separated text file describing the Z-scores for each tetranucleotide in each input sequence.

In addition, all methods produce a table of output percentage identity (ANIm and ANIb) or correlation (TETRA), between each sequence.

If graphical output is chosen, the output directory will also contain PDF files representing the similarity between sequences as a heatmap with row and column dendrograms.

#### Usage

```
calculate_ani.py [options]

Options:
   -h, --help            show this help message and exit
   -o OUTDIRNAME, --outdir=OUTDIRNAME
                         Output directory
   -i INDIRNAME, --indir=INDIRNAME
                         Input directory name
   -v, --verbose         Give verbose output
   -f, --force           Force file overwriting
   -s, --fragsize        Sequence fragment size for ANIb
   --skip_nucmer         Skip NUCmer runs, for testing (e.g. if output already
                         present)
   --skip_blast          Skip BLAST runs, for testing (e.g. if output already
                         present)
   --noclobber           Don't nuke existing files
   -g, --graphics        Generate heatmap of ANI
   -m METHOD, --method=METHOD
                         ANI method
   --maxmatch            Override MUMmer settings and allow all matches in 
                         NUCmer
   --nucmer_exe=NUCMER_EXE
                         Path to NUCmer executable
   --blast_exe=BLAST_EXE
                         Path to BLASTN+ executable
   --makeblastdb_exe=MAKEBLASTDB_EXE
                         Path to BLAST+ makeblastdb executable
```

Example data and output can be found in the directory `test_ani_data`. The data are chromosomes of four isolates of *Caulobacter*. Analyses can be performed with the command lines:

```
$ ./calculate_ani.py -i test_ani_data/ -o test_ani_data_ANIb -m ANIb -g --format=png
$ ./calculate_ani.py -i test_ani_data/ -o test_ani_data_ANIm -m ANIm -g --format=png
$ ./calculate_ani.py -i test_ani_data/ -o test_ani_data_TETRA -m TETRA -g --format=png
```

which generate the following graphical output, supporting the assignment of `NC_002696` and `NC_011916` to the same species (*C.crescentus*), and the other two isolates to distinct species (`NC_014100`:*C.segnis*; `NC_010338`:*C.* sp K31):

![ANIb graphical output for *Caulobacter* test data](test_ani_data/ANIb.png "ANIb graphical output")
![ANIm graphical output for *Caulobacter* test data](test_ani_data/ANIm.png "ANIm graphical output")
![TETRA graphical output for *Caulobacter* test data](test_ani_data/TETRA.png "TETRA graphical output")

#### Dependencies

(mandatory)

* **Biopython** <http://www.biopython.org>
* **BLAST+** executable in the `$PATH`, or available on the command line (**ANIb**) <ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/>
* **MUMmer** executables in the $PATH, or available on the command line (**ANIm**) <http://mummer.sourceforge.net/>

(optional, only required for graphical output)

* **R** with shared libraries installed on the system <http://cran.r-project.org/>
* **Rpy2** <http://rpy.sourceforge.net/rpy2.html>

### <a name="draw_gd_all_core">`draw_gd_all_core.py`</a>

This script was made available to provide demonstration GenomeDiagram usage code, as part of a [Twitter conversation](https://twitter.com/widdowquinn/status/411165587297951744/photo/1). The script renders the image in the photograph, which is the same as that at the foot of [this blog post](http://armchairbiology.blogspot.co.uk/2012/09/the-colours-man-colours.html).

**NOTE: The script will not run without modification unless you have both the required data, and the [`iadhore.py`](https://github.com/widdowquinn/pyADHoRe) module.**

#### Usage

```
draw_gd_all_core.py
```

With the appropriate data for input, this renders the image:

![Graphical output from `draw_gd_all_core.py`](images/Dickeya_core_collinear.png "Dickeya core collinear regions")

#### Dependencies

* **Biopython** <http://www.biopython.org>
* **Reportlab** <http://www.reportlab.org> 
* **ColorSpiral** <https://github.com/widdowquinn/ColorSpiral> (this has since been incorporated into Biopython, but this script relies on the standalone library)
* **iAdhore.py** <https://github.com/widdowquinn/pyADHoRe>
* **relevant data** (not yet publicly available)


### <a name="get_NCBI_cds_from_protein">`get_NCBI_cds_from_protein.py`</a>

Given input of protein sequences with suitably-formatted identifier strings (e.g. those in the NCBI nr database, having `>gi|[0-9]|.*` as their identifier), this script uses Entrez tools to find corresponding coding sequences where possible.

If the `--sto` option is used, sequence identifiers of the format `<seqid>|/<start>-<end> >gi|[0-9].*|/[0-9]*-[0-9]*` will be interpreted as Stockholm sequence headers, as described at <http://sonnhammer.sbc.su.se/Stockholm.html>

Unless the `--keepcache` option is passed, all data obtained from Entrez retained in local cache files, to aid debugging and limit network traffic, will be deleted when the script completes. By default, caches will have a  filestem reflecting the time that the script was run.  Cache filestems can be specified by the `-c` or `--cachestem` option; if the cache already exists, it  will be reused, and not overwritten.  This is useful for debugging.

An email address must be provided for Entrez, using the `-e` or `--email` option.

Queries will be batched in groups of 500 using EPost by default.  Batch size can be specified with `-b` or `--batchsize`.

Sequences can be provided via stdin, and output sequences are written to  stdout, by default.  Specifying a filename with `-o` or `--outfilename` writes output to that file.

The script runs as follows:

* Collect input sequences with valid sequence identifiers
* Check identifiers against an ELink cache of `protein_nuccore` searches. Where there is an entry in the cache, use this.  For sequences with no cache entry, compile identifiers in batches and submit queries in a `protein_nuccore `search with EPost. Add the recovered results to the ELink cache.
* Collect identifiers from the Elink results, and check against a cache of   GenBank headers.  For identifiers with no cache entry, compile identifiers in batches and submit an EFetch for the GenBank headers using EPost, caching the results.
* For each of the query protein sequences, choose the shortest potential nucleotide coding sequence from the GenBank header cache, and check its    identifier against the full GenBank record cache.  For identifiers with no cache entry, compile them in batches and submit an EFetch for the complete GenBank record, and cache the results.
* For each of the query protein sequences, get the corresponding CDS from the cached full GenBank sequence on the basis of a matching `GI:[0-9]*` value in a `db_xref` qualifier (and/or a matching `protein_id` qualifier if there's a suitable accession in the query sequence header.
* Extract the CDS nucleotide sequence from the parent sequence, and write to the output stream, with appropriate headers.

While the script runs, the important query and CDS data are kept in the results dictionary, keyed by query ID, with value a dictionary containing the sequence identifier, query AA sequence, query GI ID, and eventually the matching CDS feature, its sequence and, if the Stockholm header format is  set, the sequence that codes for the region specified in the query header.

#### Usage

```
Usage: get_NCBI_cds_from_protein.py [options]

Options:
  -h, --help            show this help message and exit
  -o OUTFILENAME, --outfile=OUTFILENAME
                        Output FASTA sequence filename
  -i INFILENAME, --infile=INFILENAME
                        Input FASTA sequence file
  -v, --verbose         Give verbose output
  -e EMAIL, --email=EMAIL
                        Entrez email
  -c CACHESTEM, --cachestem=CACHESTEM
                        Suffix for cache filestem
  -b BATCHSIZE, --batchsize=BATCHSIZE
                        Batchsize for EPost submissions
  -r RETRIES, --retries=RETRIES
                        Maximum number of Entrez retries
  -l LIMIT, --limit=LIMIT
                        Limit number of sequences processed (for testing)
  --keepcache           Keep cache files for debugging purposes
  --sto                 Parse .*/start-end identifiers as sequence regions
```

#### Dependencies

* **Biopython** <http://www.biopython.org>

### <a name="nucmer_to_crunch">`nucmer_to_crunch.py`</a>

A short script that converts the tabular output of **MUMmer**'s `show-coords` script to `.crunch` format, so it can be visualised with Sanger's **ACT** application. This script uses the alignment length on the reference sequence as the score.

The script acts equivalently to the one-liner:

```
tail -n +6 in.coords | awk '{print $7" "$10" "$1" "$2" "$12" "$4" "$5" "$13}' > out.crunch
```

but has the advantage that you don't have to remember which columns go in which order, and the Python boilerplate provides nicer logging and usage information.

#### Usage

```
Usage: nucmer_to_crunch.py [-h] [-o OUTFILENAME] [-i INFILENAME] [-v]

optional arguments:
  -h, --help            show this help message and exit
  -o OUTFILENAME, --outfile OUTFILENAME
                        Output .crunch file
  -i INFILENAME, --infile INFILENAME
                        Input .coords file
  -v, --verbose         Give verbose output
```

The script can be run on test data in the `test_nucmer` directory, as follows:

```
$ python nucmer_to_crunch.py -i test_nucmer/E_coli_nucmer.coords -o test_nucmer/E_coli_nucmer.crunch

$ head test_nucmer/E_coli_nucmer.coords 
NC_002695.fna NC_004431.fna
NUCMER

    [S1]     [E1]  |     [S2]     [E2]  |  [LEN 1]  [LEN 2]  |  [% IDY]  | [TAGS]
=====================================================================================
       1      327  |        1      309  |      327      309  |    93.88  | gi|15829254|ref|NC_002695.1|	gi|26245917|ref|NC_004431.1|
     321     5577  |     1015     6270  |     5257     5256  |    96.98  | gi|15829254|ref|NC_002695.1|	gi|26245917|ref|NC_004431.1|
    5680     9935  |     6281    10536  |     4256     4256  |    98.26  | gi|15829254|ref|NC_002695.1|	gi|26245917|ref|NC_004431.1|
    9943    15838  |    10714    16606  |     5896     5893  |    97.83  | gi|15829254|ref|NC_002695.1|	gi|26245917|ref|NC_004431.1|
   15911    18278  |    20606    22971  |     2368     2366  |    89.28  | gi|15829254|ref|NC_002695.1|	gi|26245917|ref|NC_004431.1|
   
$ head test_nucmer/E_coli_nucmer.crunch
327 93.88 1 327 gi|15829254|ref|NC_002695.1| 1 309 gi|26245917|ref|NC_004431.1|
5257 96.98 321 5577 gi|15829254|ref|NC_002695.1| 1015 6270 gi|26245917|ref|NC_004431.1|
4256 98.26 5680 9935 gi|15829254|ref|NC_002695.1| 6281 10536 gi|26245917|ref|NC_004431.1|
5896 97.83 9943 15838 gi|15829254|ref|NC_002695.1| 10714 16606 gi|26245917|ref|NC_004431.1|
2368 89.28 15911 18278 gi|15829254|ref|NC_002695.1| 20606 22971 gi|26245917|ref|NC_004431.1|
7407 95.87 25167 32573 gi|15829254|ref|NC_002695.1| 23024 30444 gi|26245917|ref|NC_004431.1|
30534 97.32 32619 63152 gi|15829254|ref|NC_002695.1| 30872 61369 gi|26245917|ref|NC_004431.1|
6138 96.84 64619 70756 gi|15829254|ref|NC_002695.1| 61489 67627 gi|26245917|ref|NC_004431.1|
5325 95.36 70758 76082 gi|15829254|ref|NC_002695.1| 68341 73750 gi|26245917|ref|NC_004431.1|
4403 95.26 76062 80464 gi|15829254|ref|NC_002695.1| 75557 79986 gi|26245917|ref|NC_004431.1|
```

### <a name="restrict_long_contigs">`restrict_long_contigs.py`</a>

A short script that takes as input a directory containing (many) FASTA files describing biological sequences, and writes to a new, named directory multiple FASTA files containing the same sequences, but restricted only to those sequences whose length is greater than a passed value.

Example usage: You have a directory with many sets of contigs from an assembly. This script will produce a new directory of the same data where the contig lengths are restricted to being greater than a specified length.

#### Usage

```
Usage: restrict_long_contigs.py [options] <input_directory> <output_directory>

Options:
  -h, --help            show this help message and exit
  -l MINLEN, --minlen=MINLEN
                        Minimum length of sequence
  -s SUFFIX, --filesuffix=SUFFIX
                        Suffix to indicate the file was processed
  -v, --verbose         Give verbose output
```

The script can be run on the test data in the directory `test_contigs` as follows:

```
$ ./restrict_long_contigs.py test_contigs/ test_contigs_out
WARNING: Output directory test_contigs_out does not exist: creating it
$ grep -c '>' test_contigs/*
test_contigs/contigs_0.fa:27
test_contigs/contigs_1.fa:45
test_contigs/contigs_2.fa:47
$ grep -c '>' test_contigs_out/*
test_contigs_out/contigs_0_restricted.fa:23
test_contigs_out/contigs_1_restricted.fa:24
test_contigs_out/contigs_2_restricted.fa:24
```

#### Dependencies

* **Biopython** <http://www.biopython.org>

### <a name="run_mlst">`run_MLST.py`</a>

This script takes a PubMLST (http://pubmlst.org) profile table, and the corresponding MLST sequences in FASTA format, to apply the protocol described in:

* Larsen MV, Cosentino S, Rasmussen S, Friis C, Hasman H, et al. (2012) Multilocus Sequence Typing of Total Genome Sequenced Bacteria. *J Clin Microbiol* **50**: 1355-1361. [doi:10.1128/JCM.06094-11](http://dx.doi.org/10.1128/JCM.06094-11)

in order to assign sequence types to a set of input sequences.

For any large-scale, persistent analysis, you're probably better off using BIGSdb, or a Galaxy MLST workflow. See, for example:

* Jolley KA, Maiden MCJ (2010) BIGSdb: Scalable analysis of bacterial genome variation at the population level. BMC Bioinformatics 11: 595. [doi:10.1186/1471-2105-11-595](http://dx.doi.org/10.1186/1471-2105-11-595).
* [http://bit.ly/MuT9fe](http://bit.ly/MuT9fe) A presentation from GCC2011 describing an MLST workflow in Galaxy. I've not yet been able to find it in the Galaxy toolshed at [http://toolshed.g2.bx.psu.edu/](http://toolshed.g2.bx.psu.edu/)

But, if you have a one-shot, one-time use this script may be quicker to run than configuring the Apache and PostgreSQL infrastructure used by BIGSdb, or installing a local Galaxy instance, and all the backend that requires.

#### Usage

```
run_MLST.py [-h] [-o OUTDIRNAME] [-i INDIRNAME] [-g GENOMEDIR]
                   [-p PROFILE] [-l LOGFILE] [-v] [-f] [--blast_exe BLAST_EXE]
                   [--formats FORMATS]

optional arguments:
  -h, --help            show this help message and exit
  -o OUTDIRNAME, --outdir OUTDIRNAME
                        Output MLST classification data directory
  -i INDIRNAME, --indirname INDIRNAME
                        Directory containing MLST allele sequence files
  -g GENOMEDIR, --genomedir GENOMEDIR
                        Directory containing genome sequence files
  -p PROFILE, --profile PROFILE
                        Tab-separated plain text table describing MLST
                        classification scheme.
  -l LOGFILE, --logfile LOGFILE
                        Logfile location
  -v, --verbose         Give verbose output
  -f, --force           Force overwriting of output directory
  --blast_exe BLAST_EXE
                        Path to BLASTN+ executable
  --formats FORMATS     Comma-separated list of output formats
```

The script can be run on test (*Bordetella*) data, as downloaded from [http://pubmlst.org/bordetella/](http://pubmlst.org/bordetella/) and [ftp://ftp.ncbi.nih.gov/genomes/Bacteria/](ftp://ftp.ncbi.nih.gov/genomes/Bacteria/) and provided in the `test_MLST` directory:

```
$ time ./run_MLST.py -i test_MLST/alleles -g test_MLST/chromosomes/ -p test_MLST/Bordetella_profiles.tab -o test_MLST/MLST_output -f
                              pepA pgm icd tyrB glyA fumC adk  ST
gi_33591275_ref_NC_002929.2_     1   1   1    1    1    1   1   1
gi_384202563_ref_NC_017223.1_    1   1   1    1    1    1   1   1
gi_33594723_ref_NC_002928.3_     2   5   2    2    2    2   2  19
gi_412337338_ref_NC_019382.1_    3   2   1    1    3    3   5  27
gi_187476514_ref_NC_010645.1_   12  15  15    8   16    8   9  76

real	0m3.283s
user	0m8.314s
sys	0m1.179s
```

#### Dependencies

* **Biopython**: [http://www.biopython.org](http://www.biopython.org)
* **Python 2.6+** (for multiprocessing): [http://python.org](http://python.org)
* **Pandas**: [http://pandas.pydata.org/](http://pandas.pydata.org/)
* **OpenPyXL** (for Excel output): [http://pythonhosted.org/openpyxl/](http://pythonhosted.org/openpyxl/) This may also require libxml2 and lxml if not present on your system


### <a name="run_signalp">`run_signalp.py`</a>

This script takes a FASTA format file containing protein sequences as input, and runs a local copy of `signalp` (in the `$PATH`) on the contents, collecting the output generated with the `-short` option of `signalp`. 

The script splits the input sequence file into a number of smaller FASTA files suitable for distributed processing using, e.g. with Python's `multiprocessing` module. It prepares intermediate files with a maximum of 1000 sequences per file, to avoid falling foul of the undocumented upper input limit of `signalp`.  

The script runs `signalp` independently on  each split file, and the results are concatenated into a single output file. If no output file is specified, then the output file shares a stem with the input file. The organism type is specified at the command line in the same way as for `signalp`.

#### Usage ####

```
[python] run_signalp.py [euk|gram+|gram-] <FASTAfile> [-o|--outfilename <output file>]
```

#### Dependencies

* **signalp** <http://www.cbs.dtu.dk/cgi-bin/nph-sw_request?signalp>
* **Python 2.6+** <http://www.python.org> (2.6+ required for `multiprocessing`)


### <a name="run_tmhmm">`run_tmhmm.py`</a>

This script takes a FASTA format file containing protein sequences as input, and runs a local copy of `tmhmm` (in the `$PATH`) on the contents, collecting the output generated with the `-short` option of `tmhmm`. 

The script splits the input sequence file into a number of smaller FASTA files suitable for distributed processing using, e.g. with Python's `multiprocessing` module. It prepares intermediate files with a maximum of 1000 sequences per file, to avoid falling foul of the undocumented upper input limit of `tmhmm`.  

The script runs `tmhmm` independently on  each split file, and the results are concatenated into a single output file. If no output file is specified, then the output file shares a stem with the input file.

#### Usage ####

```
run_tmhmm.py <FASTAfile> [-o|--outfilename <output file>]
```

#### Dependencies

* **tmhmm** <http://www.cbs.dtu.dk/cgi-bin/nph-sw_request?tmhmm>
* **Python 2.6+** <http://www.python.org> (2.6+ required for `multiprocessing`)


### <a name="stitch_six_frame_stops">`stitch_six_frame_stops.py`</a>

Takes an input (multiple) FASTA sequence file, and replaces all runs of N with the sequence NNNNNCATTCCATTCATTAATTAATTAATGAATGAATGNNNNN, which contains start and stop codons in all frames.  All the sequences in the input file are then stitched together with the same sequence.

Overall, the effect is to replace all regions of base uncertainty with the insert sequence, forcing stops and starts for gene-calling, to avoid chimeras or frame-shift errors due to placement of Ns.

This script is intended for use in assembly pipelines, where contigs are
provided in the correct (or, at least, an acceptable) order.

If no input or output files are specified, then STDIN/STDOUT are used.

The script also produces a GFF file describing the resulting contig locations on the stitched assembly, which is useful for visualisation, e.g. with Sanger's Artemis tool, or JHI's Tablet program.

#### Usage ####

```
Usage: stitch_six_frame_stops.py [options]

Options:
  -h, --help            show this help message and exit
  -o OUTFILENAME, --outfile=OUTFILENAME
                        Output filename
  -i INFILENAME, --infile=INFILENAME
                        Input filename
  --id=SEQID            ID/Accession for the output stitched sequence
  -v, --verbose         Give verbose output
```

#### Dependencies

* **Biopython** <http://www.biopython.org>
* **Python 2.6+** <http://www.python.org>


## Licensing

Unless otherwise indicated in the script, all code is subject to the following agreement:

(c) The James Hutton Institute 2013
Author: Leighton Pritchard

Contact:
`leighton.pritchard@hutton.ac.uk`

Address: 
> Leighton Pritchard,
> Information and Computational Sciences,
> James Hutton Institute,
> Errol Road,
> Invergowrie,
> Dundee,
> DD6 9LH,
> Scotland,
> UK

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but **WITHOUT ANY WARRANTY**; without even the implied warranty of **MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  If not, see <http://www.gnu.org/licenses/>.
