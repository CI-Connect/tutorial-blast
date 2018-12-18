[title]: - "Sequence Search with BLAST"
[TOC]

## Overview
This tutorial will cover how to setup and submit a BLAST alignment as a job on OSG resources.

## What is BLAST?
The [Basic Local Alignment Search Tool (BLAST)](http://blast.ncbi.nlm.nih.gov/Blast.cgi) is used to map or align gene sequences to one or more reference genome(s). It uses an adaptation of the [Smith-Waterman alignment algorithm](https://www.sciencedirect.com/science/article/pii/0022283681900875) to pinpoint similar matches at optimized speed. Scientists can customize their alignments by [implementing one of many options](https://www.ncbi.nlm.nih.gov/books/NBK279684/#_appendices_Options_for_the_commandline_a_) when performing searches.

##Execution of BLAST

To run a BLAST alignment on OSG, you need three things:

* BLAST executable (such as blastp or blastx)
* Reference Database
* Query sequence(s)

All of these need to be on the worker node when a job is executed for the alignment to succeed.  The BLAST executable can be fairly large compared to other executables, something like 26MB.  Also, some reference databases can be rather large.  Therefore we want to use [squid caching](https://wiki.squid-cache.org/SquidFaq/AboutSquid) and a web server to speed up data transfer speeds to our jobs.  Luckly, on OSG Connect, we have a webserver ready: Stash.

## Preparing Input

Place your BLAST executable and your database in the public web directory on OSG Connect, `~/stash/public/`.  For this tutorial, We have already done this in the public directories listed below. There is no need to download these files, we will have our jobs access them directly.

	Executable: http://stash.osgconnect.net/+dweitzel/blast/bin/blastp
	Database: 
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.phr
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pin
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pnd
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pni
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psd
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psi
	http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psq

Since the files are hosted on a webserver, they can be cached at sites by using forward proxies, which are widely deployed on the OSG. Also, note that these files are already in an indexed BLAST database format. If you are using a custom reference database, it will need to be indexed prior to running your BLAST alignment. Directions for creating custom BLAST databases can be found on [NCBI Bookshelf](https://www.ncbi.nlm.nih.gov/books/NBK279688/). It is recommended to do this on Stash before submitting your jobs.


## Job Submission

To obtain the blast tutorial files, type

	$ tutorial blast
	
We will use this quick wrapper script, `blast_wrapper.sh` in order to ensure our BLAST executable runs correctly:
  
	#!/bin/sh
	module load blast
	chmod +x $1
	"$@"
	
Next, we will write the BLAST submit file, `blast.submit`.
	 
	executable = blast_wrapper.sh
	arguments  = ./blastp -db yeast.aa -query query1 -out query1.alignment
	 
	should_transfer_files = YES
	when_to_transfer_output = ON_EXIT
	transfer_input_files = http://stash.osgconnect.net/+dweitzel/blast/bin/blastp, \
	    http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa, \
	    http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.phr, \
	    http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pin, \
		http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pnd, \
		http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.pni, \
		http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psd, \
		http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psi, \
		http://stash.osgconnect.net/+dweitzel/blast/data/yeast.aa.psq, \
		http://stash.osgconnect.net/+dweitzel/blast/queries/query1
	transfer_output_files = query1.alignment
	transfer
	 
	output = job.out
	error = job.err
	log = job.log
	
	queue 1

The submit file will need to include all of our three necessary items in `transfer_input_files`, the executable, the database, and the input query file. Also, we need to be sure that we specify our alignemnt file in `transfer_output_files` so it is returned at the end of the job. Notice that since we are using a wrapper script, the script is the executable and the BLAST executable needs to be included in our list of `transfer_input_files`:

Submit the job with condor_submit:

	$ condor_submit blast.submit
	
You can watch the job then with `condor_q`.  The output of the BLAST alignment will be in the file we specified, `query1.alignment`. We also 

## Next Steps
Not all BLAST databases are small enough to use HTTP.  Any files that are larger than a few hundred MB's is too large for HTTP. The current nr database is several GB's.  In that case, a possible solution is to partition the database, and run several jobs for each query (or set of queries) to search each of the partitions.  In that case, you only transfer the partition of the database that you need, reducing the required input data.

For references on how to partition the database, see [BLAST Parallelization on Partitioned Databases with Primary Fragments](http://vecpar.fe.up.pt/2008/hpdg08_papers/4.pdf). The issue with partitioning the database is not how to cut the database, but rather how to stitch back together the output of BLAST. Especially the E value and and output.

## Getting Help
For assistance or questions, please email the OSG User Support team  at [user-support@opensciencegrid.org](mailto:user-support@opensciencegrid.org) or visit the [help desk and community forums](http://support.opensciencegrid.org).
