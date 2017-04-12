# palumbi_scripts
Most up to date scripts used in the Palumbi lab

# Transcriptome assembly & analysis in the Palumbi Lab*

## GENERAL INFORMATION

### * Scripts are formatted for Stanford internal use. We use SLURM to send jobs to Stanford's Sherlock cluster.


### How to create a Sherlock account 
- to get access and support, e-mail research-computing-support[at]stanford.edu
- download [Kerberos](https://uit.stanford.edu/service/kerberos)
- [Sherlock Wiki] (http://sherlock.stanford.edu/mediawiki/index.php/Main_Page)
- To configure your SSH client to pass those Kerberos credentials for MAC, open terminal (Click Spotlight, type in terminal) and run the following commands:
```
	mkdir -p ~/.ssh 
	echo "Host sherlock sherlock.stanford.edu sherlock* sherlock*.stanford.edu   
	GSSAPIDelegateCredentials yes   
	GSSAPIAuthentication yes" >> ~/.ssh/config 
```
- create project directory in `$PI_SCRATCH/` (1TB space available)
	- more space and accessible to other users in the lab 
- your personal `$HOME` is 15GB
- [more info on Sherlock storage](http://sherlock.stanford.edu/mediawiki/index.php/DataStorage)

### Downloading & storing data 
- install [fdt.jar](http://monalisa.cern.ch/FDT/) in your chosen Sherlock directory
- use command line from Gnomex to download to Sherlock
- backup raw files on external harddrive in lab

### Sherlock basics 

- `kinit user@stanford.edu` & type in pw to gain permission
- `ssh user@sherlock.stanford.edu` to access cluster
- `$HOME` your personal directory
	 - limited to 15GB storage
	 - not accessible to other users
-	`$PI_HOME/scripts` & `$PI_HOME/programs` contains all common use lab scripts & programs
	 - add your contributions to these two directories
	 - add a program to Sherlock
	 - `cd $PI_HOME/programs`
	 - `wget <linktofiledownload>`
	 - `unzip <file>` or `gzip <file.gz>` 
- 	modifying your path to include scripts & programs:
	-  this allows you to access these directories from any directory without having to hardcode the path
	 
	 ```
	 $ cd ~
	 $ ls -a #list hidden items
	 $ nano .bashrc
	 #paste the following into your .bashrc
	 $ export PATH="$PI_HOME/programs:$PATH"
	 $ export PATH="$PI_HOME/scripts:$PATH"
	 $ export PATH="/share/PI/spalumbi/programs/anaconda/bin:$PATH"
	 ```
- 	`login_node` for moving around sherlock
- 	`sdev` a node for running small test jobs straight from the command line
	- one hour time limit
	- use `sdev -t 2:00:00` to get max 2 hrs
- 	`bash` use for small jobs in terminal
- 	`sbatch` submits the job generated by your script to the cluster
-  to run your job on different nodes, in your batch script, add: 
	-  	`#SBATCH -p owners` access to 600 nodes only available to owners. You will be kicked off an owner node if that owner logs on. Always check your slurm file to see if your job was aborted
	-  `#SBATCH -p spalumbi` 256GB memory node
	-  `#SBATCH -p hns` 1TB memory node
	-  you can list multiple nodes in your script with `#SBATCH -p spalumbi,hns`
- to check on status of job
	- squeue -u username
	- squeue | grep 'spalumbi'
- to cancel a job
	- `scancel <jobID>` to cancel 1 job
	- `scancel -u <username>` will cancel all jobs
- to see how much memory your job has used so far
	- ` sstat --format JobID,NTasks,nodelist,MaxRSS,MaxVMSize,AveRSS,AveVMSize $JOBNUMBER`

### Best practices 🌊 :
 
🌊 Back up your scratch directory!
You can use the Sherlock data transfer node to move large datasets onto a backup hard drive
```
kinit username@stanford.edu
rsync -avz --progress --stats -e 'ssh -o GSSAPIAuthentication=yes' user@sherlock-dtn.stanford.edu:/<sherlock directory> <backup location>
```
You can use your Stanford Google Drive (with unlimited storage) to back up your files
```
ml load gdrive
gdrive -help
gdrive upload --recursive <path>
```

Check to see how much space you're taking up in the shared SCRATCH directory
`du -sh * | sort -h`

🌊 `chmod 775 *` programs, scripts, some files that you create so others can use them

🌊 `chmod 444` files you don’t want to accidentally write over, 
	`chmod -R ###` for directories

🌊 if downloading a program for the first time, move the executable of the program to the programs folders
 `mv <program> $PI_HOME/programs`

🌊 if downloading a new version of a program, rename with version to not override any executables in that directory

🌊 try to create scripts that are not hard-coded 

🌊 comment your scripts!

🌊 Tips for checking outputs along the way:

- always look at your slurm files for errors
	- `cat slurm*`
- when making scripts that split up data into TEMP files for parallel processing, add a line that states your input file in the slurm output
	- `echo $1` if $1 is your input file
- to count lines in a file
	- `wc - l <filename>`
- to count contigs in a file
	- `grep -c “>” <filename>`



## DE NOVO TRANSCRIPTOME ASSEMBLY

### 1) Trim & Clip (Trimmomatic) 
- this program removes adapters, low quality seqs, etc.
- [webpage](http://www.usadellab.org/cms/?page=trimmomatic)
-  for paired end reads:
	- `bash batch-trimmomatic-pe.sh *_1.txt`
	- outputs 4 files: 1\_paired.fq, 2\_paired.fq, 1\_unpaired.fq, 2\_unpaired.fq
-  for single end reads:
	`batch-trimmomatic-se.sh`


### 2) Quality check (FastQC) 
- this step is useful for picking samples to use in your Trinity assembly
- this step is also helpful for PE data, where you use the length of your sequences for merging reads (i.e. FLASH)
-  `batch-fastqc.sh`
- to move a file from the cluster to your computer
- open a terminal that is not logged into cluster
	`rsync user@sherlock.stanford.edu:/share/PI/spalumbi/... /Users/username/Desktop/...`
	- use `rsync -a` to transfer an entire directory

### 3) Merge Paired End reads (FLASH) 
- 	This step is necessary for PE data to merge your two reads (\_1 and \_2) that you will use in transcriptome assembly, but is not necessary for SE data 
-  use your quality outputs from FastQC 
- 	[webpage](https://ccb.jhu.edu/software/FLASH/)
- 	get lengths of your trimmed seqs above and use in FLASH script
- 	scripts: `batch-flash.sh`
- 	You can pick samples with the best FLASH %assembled to use for your assembly


### 4) Pick what samples to assemble into a transcriptome  
- 	pick samples relevant to your biological question
- 	restrict sample # for computational capacity
- 	can use FLASH results for quality filtering
- 	Example assembly for population analysis: 
	-  16 Balanus samples from Hopkins
	-  we created 3 separate Trinity assemblies of 3 individuals with best FLASH merging scores
		- the 3 Trinity assemblies ranged from 94,000 - 250,000 contigs
		- after taking the longest isoform of each assembly: 75,000 - 180,000
	-  We then meta-assembled the 3 Trinity assemblies with a long read assembler (CAP3)
		- we found that meta-assembling allowed us to put more samples/diversity into our assembly without inflating the number of contigs
		- for comparison
			- Trinity assembly of 8 individuals: 930,000 contigs and 830,000 longest isoforms
			- CAP3 meta assembly of 3 Trinity assemblies (each made of 3 individuals, 9 individuals total): 200,000 contigs+singlets

### 5a) De novo assembly (Trinity) 
- 	[Trinity wiki] (https://github.com/trinityrnaseq/trinityrnaseq/wiki)
- 	to see insides of Trinity: 
	-  	`.Trinity --show_full_usage_info`
- 	example script: `example_Trinity_mkm.sbatch` 
	-  you need to update this example for your assembly 
- 	[Trinity computing requirements](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity-Computing-Requirements)
	-  1GB RAM for 1M reads
	- 	Example: 3 Balanus samples: 11649437 + 11728787 + 11556893
	-  total pairs of reads = 34,835,117 is 35GB RAM and 35 hours of time

### 5b)Transcriptome assembly quality assessment 

- 	to get median contig length:

  ```
  $ samtools faidx Bg_2assembliesof3.fa
  $ module load R
  $ R
  $ index<-read.delim(‘<file.fa.fai>’, header=F)
  $ median(index[,2])
  $ table(cut(index[,2],30)) #bins into groups of 30
  $ table(index[,2]>300) #how many contigs are greater than 30bp
  ```

### 6a)Take the longest Isoform from each contig 
- 	to call program: `perl longestisoform_trinity.pl <input.fasta> <output.fasta>`
- 	this reduces computational load for CAP3

### 6b)If meta-assembling Trinity assemblies 
- 	rename contigs in one file by adding “a” to end of file name, for ex: "TRINITYa_blahblah"
- 	otherwise CAP3 may get confused if names are repeated
- 	`cat Trinityrun1.fa Trinityrun2.fa > all_assemblies.fasta`

### 7)Meta-Assembly (CAP3) 
- 	[paper](http://genome.cshlp.org/content/9/9/868.full)
- 	[manual](http://computing.bio.cam.ac.uk/local/doc/cap3.txt)
- 	CAP3 is an overlap consensus assembler that will merge reads that would not assemble in Trinity due to high heterozygosity
- 	to call program: `sbatch cap3.sh`
- 	merge your contigs and singlets files into one assembly file
	- `cat file.fasta.cap.contigs file.fasta.cap.singlets > newfile.fasta`
	- contigs are what CAP3 merged, singlets did not merge and still contain the Trinity output name
- 	to look at contig #s after CAP3:  
	-  `grep -c '>' file.fasta`
- 	to check for contig name duplicates: 
	-   `grep ">" <assembly_file.fa> | perl histogram.pl | head -n`

### 8)Annotate (BLAST) 
- 	[Blastx program download](http://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)
-  [blastx commands, table C4](http://www.ncbi.nlm.nih.gov/books/NBK279675/)

#### How to download and create a blast database on your cluster
- download your database
	- ex: uniprot, blast-nr, genome of species of interest
- to open .tar files downloaded from genbank: 
	`for i in *.tar ; do tar -xvf $i ; done &`
- to update an exisitng database (i.e. blast-nr)
	- `perl /share/PI/spalumbi/programs/ncbi-blast-2.3.0+/bin/update_blastdb.pl <database_directory>`
- 	how to test blast script on small file:

	```
	sdev
	head <assembly.fa> > <test_assembly.fa>
	srun --mem=12000 —pty bash
	blastx -db /share/PI/spalumbi/genbank_nr_Feb_2016/nr -query <test_assembly.fa> -out <test.out> -outfmt 11 &  

	```

### 8a)Annotate with Uniprot database
- 	Uniprot is a more curated database and is recommended over NCBI-nr



#### How to download & create the uniprot database for the first time:
- 	[download Swiss-Prot & Trembl databases](http://www.uniprot.org/downloads)

	```
	wget <link>
	gunzip <file>.gz
	# merge two files into one database
	cat uniprot_sprot.fasta uniprot_trembl.fasta > unitprot_db.fasta
	# make your new fasta into a database
   makeblastdb -in uniprot_db.fasta -dbtype prot -out uniprot_db
   ```
### Run uniprot blast
- usage: `bash batch-blast-uniprot.sh infile`
	- the script above splits your assembly into smaller files and calls the blastx on your uniprot database
- 	after running, check if you get an error during your blasts
	- `cat slurm*` 
	- most often, your blast may time out
- if you get an error:
 	- `grep -B 1 error slurm*.out`
  	- for any file with error, take line before (the tempfile)
	- `cat <TEMPwerror.fa> <TEMPwerror.fa> > <didnotfinish.fa>`
		- this concatenates all TEMP files that contain an error into a new file
	- you may want to reduce the # of contigs the batch-blast-uniprot.sh script generates for each TEMP file
	- `bash batch-blast-uniprot.sh <didnotfinish.fa>` 

### 8b)Annotate with NCBI-nr database
#### How to download & create the nr database for the first time
- Make sure local database on sherlock is up to date
- to open .tar files downloaded from genbank: 
	`for i in *.tar ; do tar -xvf $i ; done &`
	- blast-nr files are already databases, no need to use makedb script
- 	usage
	- `bash batch-blast-nr.sh`
	- splits your assembly into TEMP files for parallel processing
- 	after running, check if you get an error during your blasts
	- `cat slurm*` 
	- most often, your blast may time out
- if you get an error:
 	- `grep -B 1 error slurm*.out`
  	- for any file with error, take line before (the tempfile)
	- `cat <TEMPwerror.fa> <TEMPwerror.fa> > <didnotfinish.fa>`
		- this concatenates all TEMP files that contain an error into a new file
	- you may want to reduce the # of contigs the batch-blast-uniprot.sh script generates for each TEMP file
	- `bash batch-blast-uniprot.sh <didnotfinish.fa>`
		
### 8c)Reciprocal BLAST 
- 	`makeblastdb -in file.fasta -dbtype nucl -out file.fasta –parse_seqids`
- 	can do this to check overlap between your multiple Trinity alignments 
	- i.e., does heterozygosity cause issues in your alignments?

### 8d)Downloading a genome of interest to blast against
- ex: Lottia giganteam 
- download cDNA files for blasting your nucleotides against real transcripts
- `makeblastdb `

### 9)Parse XML blast files
- Both the uniprot and nr batch scripts above output XML formatted results
- XML files are difficult to work with, pick what information you want and parse to a tab delimited
- XML files currently hold the most information about each blast result
	- i.e. gene name, taxonomy 
- 	to submit to cluster:
	-  `bash batch-parse-uniprot.sh`
- 	usage in sdev on small file
	- `python parse-uniprot-xml.py`

### 10)Filter Assembly 
- If you BLAST to a general database and not to a specific genome, this step is necessary
- remove blasts that are likely environmental contamination, i.e. bacteria, fungi, viruses, alveolata, viridiplantae, haptophyceae, etc. 
- 	step 1:
	- `cat *_parsed.txt > all_parsed.txt` #combine all your parsed blast results
-  step 2:
	- create a file of good contigs
	- `bash grep-good-contigs.sh all_parsed.txt assembly.fa`
- 	step 3:
	- pull only good contigs from your assembly file
	- on cluster: 
	- `sbatch batch-filter-assembly.sh assembly.fa goodcontigs.txt`
	- 	check your parsed outputs to see if there are taxa in there you don’t want and change/add them to the script
- 	to check how filtering went:
	- `grep -c “Contig” goodcontigs.txt`
	- `grep -c "TRINITY" goodcontigs.txt`
	- `grep -c “^>” filteredassembly.fa`


#### Filtering other ideas 
- 	multiple blasts against different taxa (i.e. coral vs symbiont)
- 	High stringency blast
- 	Contig length cutoff
- 	phylogenetic filtering, i.e. microbes using MEGAN, KRAKEN

#### Check out characters about your assembly
- `perl abyss-fac.pl <assembly.fa>`
- number of contigs, mean length of contigs, etc.

## TRANSCRIPTOME ANALYSIS

### Map reads to assembly (Bowtie2) 
- 	[Bowtie2 download](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#the-bowtie2-build-indexer)

```
	# make a bowtie index from your final assembly
   bowtie2-build <input.fa> <name_bt2index> 
   # check that it outputs 6 files .bt2
   # Option 1: PE reads
   bash batch-bowtie2-fq-paired.sh b2index 1 *_1.txt.gz
   #check TEMPBATCH.sbatch after submitting to see if it started correctly 
   cat TEMPBATCH.sbatch
   # after it completes, check for errors
   cat slurm*
   # if rerunning, make sure you remove files that don't allow writing over, i.e.
   rm -metrics.txt 
```

#### other Bowtie2 script options:
- there are several other bowtie2 scripts available in the scripts folder for SE reads, flash merged reads, etc. 


	
## SNP CALLING, FILTERING, & ANALYSIS

### SNP Calling (Freebayes) 
- 	step 1: make a contig list for input into freebayes
- 	`bash fasta2bed.sh assembly.fa outfile`
- step 2:

```
mkdir vcfout #in same directory as your flash merged samples
sbatch freebayes-cluster.sh assembly.fa vcfout contiglist ncpu *bam

```
- if not using cluster script, see `freebayes-sequential-intervals.sbatch`

### Filter SNPs (vcflib)
- 	[vcflib website](https://github.com/vcflib/vcflib#vcflib)
- 	[vcflib scripts](https://github.com/vcflib/vcflib/tree/master/scripts)
- 	can filter for: read depth, read mapping quality, base quality, minor allele frequency, min number of reads mapped, etc.
- 	step 1: 
-  `$ fastVCFcombine.sh <outfile> *.vcf`
- 	step 2, option 1: filter by genotype (i.e. all individuals must have a quality score of 30 at that SNP)
- 	`$ sbatch vcf-filter-nomissing-maf05-allgq30.sh #samples <outfile> *.vcf`
-  this is our most strict filter
- 	step 2, option 2: filter by locus with quality score of 30
- `$ sbatch vcf-filter-nomissing-maf05-qual30.sh #samples <outfile> *.vcf`
- this is slightly less strict and may output more SNPs than the script above
- step 2, option 3: filter for eSNPs
- `$ sbatch vcf-filter-nomissing-maf05-eSNPs.sh #samples <outfile> *.vcf`
- this filter uses min number of mapped reads instead of GQ score because we expect eSNP alternates to have different numbers of reads

### Create 0,1,2 genotype SNP matrix (vcftools)
- 	`$ bash vcftools-012genotype-matrix.sh <combined_filtered_file.vcf> <outfil>`

### Format SNP Matrix
- do this in R
-  easier to do this outside of the cluster
	-  `rsync user@sherlock.stanford.edu:<files> <path to location on your computer>
- 	take 3 output files from vcftools and create one file

```
snps<-read.delim('file.012', header=F)
pos<-read.delim('file.012.pos',header=F)
indv<-read.delim<-('file.012.indv',header=F)

colnames(snps)<-paste(pos[,1],pos[,2],sep='-')
rownames(snps)<-indv[,1]
snps<-as.matrix(snps)

#PCA of SNPs
pc.out<-prcomp(snps)
summary(pc.out)
plot(pc.out$x[,1],pc.out$x[,2])	#PC1 v PC2

```

### Add Meta data to Matrix 
- 	make a meta data file with info about individuals (location, date, etc.)
- 	make sure your meta file is ordered the same as your vcfs! (i.e. ls your samples in the terminal to see their order)
- script TBD


### Test for loci under selection (BayeScan)
- [download program](http://cmpg.unibe.ch/software/BayeScan/download.html)
- 	identifies putative loci under selection

## GENE EXPRESSION ANALYSIS 

### Expression counts
- `bash get-bam-counts.sh *.bam`


### WGCNA 
- in program R
- [website](https://labs.genetics.ucla.edu/horvath/CoexpressionNetwork/Rpackages/WGCNA/)
- script TBD


### Gene expression (DESeq2)
- use this program in R

```
library(DESeq2) 
counts<-read.delim(‘counts.txt,row.names=1) 
cds<-DESeqDataSetFromMatrix(counts.txt,meta,~cluster) 
cds<-DESeq(cds) 
res<-results(cds) 
sig<-res[which(res$padj<0.05),] 
write.table(sig,file=‘DEcontigs.txt’,quote=F,sep=‘\t') 
```

### Samtools
- [manual](http://www.htslib.org/doc/samtools.html)
- to view reads mapped to contig of interest
	- `$ samtools tview <sample.bam> <assembly.fa>`
	- `$ g` #type in contig name of interest

	- create a file of one contig
	-  `$ samtools view -bh <file.bam> "Contigname" > <outfile.bam>`
-  convert bam to fasta for viewing
		-  `$ samtools bam2fq <infile.bam> | seqtk seq -A > <outfile.fa>`
