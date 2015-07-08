[title]: - "Sequence Search with BLAST"
[TOC]

## Introduction
We're going to run a real scientific application, BLAST.

I'll be honest with you. This tutorial is being written by a computer scientist (Derek Weitzel), not a biologist, so my understanding of BLAST is weak. Yet I think it's both illustrative and entertaining to run real science code instead of just toy examples. So let's work through it together.

## What is BLAST?
The [BLAST](http://blast.ncbi.nlm.nih.gov/Blast.cgi) website says: 

>The Basic Local Alignment Search Tool (BLAST) finds regions of local similarity between sequences. The program compares nucleotide or protein sequences to sequence databases and calculates the statistical significance of matches. BLAST can be used to infer functional and evolutionary relationships between sequences as well as help identify members of gene families.

I'll try to interpret this. BLAST is a pretty cool tool, particularly now that scientists have found lots of DNA and protein sequences. Let's imagine that you're a biologist, and you're curious about some gene. You've located this gene in a yeast, and you want to know if it's also in a fly. Good question. Well, biologists have nifty ways of writing down gene and protein sequences--you've probably seen the 4-letters used to represent nucleotides in DNA: GCTA. You can also have similar sequences of amino acids that make up proteins. If you have a transcription for a gene in yeast and you have the entire genome for the fly, you could just search. It's like searching for a string in a text file. Of course, it's more complicated than that. For one thing, biologists wonder if there is a similar sequence in the fly, not just an identical sequence. After all, things evolve. So BLAST is a tool for doing these searches.

Beyond this basic explanation, I can't tell you a whole lot more about it, because it gets a lot more complex than that and I'm not a biologist. If you're curious, check out the links above. Here's a short quote from the above Wikipedia page:

>In bioinformatics, Basic Local Alignment Search Tool, or BLAST, is an algorithm for comparing primary biological sequence information, such as the amino-acid sequences of different proteins or the nucleotides of DNA sequences. A BLAST search enables a researcher to compare a query sequence with a library or database of sequences, and identify library sequences that resemble the query sequence above a certain threshold. For example, following the discovery of a previously unknown gene in the mouse, a scientist will typically perform a BLAST search of the human genome to see if humans carry a similar gene; BLAST will identify sequences in the human genome that resemble the mouse gene based on similarity of sequence.

##Execution of BLAST

BLAST requires 2 things:

* BLAST executable (whether blastp or blastx)
* Database

Both need to be on the worker node when a job is executed.  The blast executable can be large (for an executable), something like 26MB.  Also, the databases can be rather large, therefore we want to use squid caching and a web server.  Luckly, on OSG Connect, we have a webserver ready: Stash.

## Preparing Input

Place your BLAST executable and your database in the public web directory on OSG Connect, `~/data/public/`.  For this tutorial, I have already done this in my public directory, and you can use it. I have provided the below links for you. You do not need to download them into your home directory, you can use mine for now.
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
Since the files are hosted on a webserver, they can be cached at sites by using forward proxies, which are widely deployed on the OSG.

## Job Submission

To obtain the blast tutorial files, type
	$ tutorial blast
The submit file will need to list all of the input files, the executable, the database, and the input query file.  We will use a quick wrapper around blast in order to execute blast correctly.  
	#!/bin/sh
	module load blast
	chmod +x $1
	"$@"
Next, we will write the BLAST submit file.
	universe = vanilla
	 
	executable = blast_wrapper.sh
	arguments  = ./blastp -db yeast.aa -query query1
	 
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
	 
	output = job.out
	error = job.err
	log = job.log
	
	queue

Submit the job with condor_submit:
	$ condor_submit blast.submit
You can watch the job then with `condor_q`.  The output of the blast run will be in `blast.out`.  

## Next Steps
Not all BLAST databases are small enough to use HTTP.  Any files that are larger than a few hundred MB's is too large for HTTP. The current nr database is several GB's.  In that case, a possible solution is to partition the database, and run several jobs for each query (or set of queries) to search each of the partitions.  In that case, you only transfer the partition of the database that you need, reducing the required input data.

For references on how to partition the database, see [BLAST Parallelization on Partitioned Databases with Primary Fragments](http://vecpar.fe.up.pt/2008/hpdg08_papers/4.pdf). The issue with partitioning the database is not how to cut the database, but rather how to stitch back together the output of BLAST. Especially the E value and and output.

## Getting Help
For assistance or questions, please email the OSG User Support team  at [user-support@opensciencegrid.org](mailto:user-support@opensciencegrid.org) or visit the [help desk and community forums](http://support.opensciencegrid.org).
