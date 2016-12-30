# Detect and categorize noncoding RNAs and identify respective targets for RNA-Seq dataset
### Introduction
RNA-Seq technology enables deep profiling of the transcriptome. Unlike microarray, RNA-Seq provides unbiased detection of novel transcripts, broader dynamic range for read counts, and easier detection of rare and low-abundance transcripts.

###Software/Hardware requirements###

A computer with a decent amount of RAM. Runtimes for Perl and the Bash shell. Basic command shell programming abilities for unix platform.

### Dataset
The raw dataset is in *.fastq format

### Methodology :

#### Quality Control
**Step 1**.	FASTQC: Prior to running RNA Seq alignment, it is important ro run fastqc http://www.bioinformatics.babraham.ac.uk/projects/fastqc/. 

Command line to run fastqc on unix is : $FastQC Sample1.Fasta. 

This generates an output folder with many report files in it. Open the index file after transferring to your windows desktop. All the green should ideally light up but that seldom happens. In the file there is a section "Per base sequence content" which should tell you how much to trim from either end. Usually it is better to choose base position which has variable content. Repeat the same steps for remaining samples as well.

**Step 2**.	TRIMMOMATIC: To do trimming use trimmomatic. Clip the values you found using previous steps and also provide a minimum Mapping Q score (we used 20) you want to keep, but here more stringent values are preferred. Minimum length to be kept depends on your original sequence length. Don’t go below 36bp.

Code :  
java –jar  /projects/MainFolder/SubFolder/ProjName/Trimmomatic-0.36/trimmoatic-0.36.jar SE –phred64 Sample1.fastq Sample1_new.fastq HEADCROP:14 AVGQUAL:20 MINLEN:36.  Once you get the output files from trimmomatic use fastqc again to see if the green light come up for most sections. The Kmer section most of the time is yellow or red, so ignore. Once all clear, align against transcriptome/genome.

**Step 3**.	DOWNLOAD GENOME: In the Ensembl directory there are 3 versions of genome. repeatmasked(rm), small-masked(sm) and regular. 
"Mus_musculus.GRCm38.dna_rm.primary_assembly.fa.gz" [repeat masked entire genome]
Make sure to index them after you download them

$ wget -c 'ftp://ftp.ensembl.org/pub/release-84/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna_rm.primary_assembly.fa.gz'

cd ../rm_masked

gunzip Mus_musculus.GRCm38.dna_rm.primary_assembly.fa.gz

**Step 4**.	To filter out non-conventional chromosomes using script(filter_genome.pl)

filter_genome.pl : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/filtergenome.pl

To run this script : $perl filtergenome.pl Mus_musculus.GRCm38.dna_rm.primary_assembly.fa Mus_musculus.GRCm38_Rmasked.fa

To build the index : $bowtie2-build Mus_musculus.GRCm38_RMasked.fa mm38

To rename the file: $cp Mus_musculus.GRCm38_RMasked mm38.fa

To create symbolic link : $ln-s mm38.fa mm38.fa.fa


####Alignment and transcriptome assembly

**Step 5**.	ALIGNMENT USING TOPHAT 2.0
To align the sample files against the mouse genome using TopHat2.0, use the following script.

Script : MyTopHat.sh

MyTopHat.sh : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MyTopHat.sh

To run the script : $bsub < MyTopHat.sh

**Step 6**.	TRANSCRIPT ASSEMBLY USING CUFFLINKS

To assemble the transcripts using cufflinks, use the following script

Script :MyCufflinks.sh

MyCufflinks.sh :https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MyCufflinks.sh

To run the script : $bsub < MyCufflinks.sh

**Step 7**.	Combine all the transcript files created from Step 6:

Script : Create a Cufflinks.gtf.list file

cufflinks.gtf.list :https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/cufflinks.gtf.list

**Step 8**.	Merge GTF file along with the combined gtf file created from step 7

Script : MyCuffMerge.sh

MyCuffMerge :https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MyCuffMerge.sh

To run the script : $bsub < MyCuffmerge.sh

**Step 9**.	Detect the normalized gene expression

Script : MyCuffQuant.sh

MyCuffQuant.sh : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MyCuffQuant.sh

To run the script : $bsub < MyCuffQuant.sh

**Step 10**.	Detect the normalized gene expression

Script : MyCuffnorm.sh

MyCuffnorm.sh : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MyCuffNorm.sh

To run the script : $bsub < MyCuffnorm.sh

**Step 11**.	LINCRNA AND TARGETS : Extract all lincRNAs, chromosome and their genomic coordinates from step 10 and put them in a file called PC.txt. Extract all protein coding genes, chromosome and their genomic coordinates from step 10 and put them in a file called PC.txt. Use the script called lincToPc.pl to identify the neighboring expressed protein coding genes

Sample lincRNA.txt : SamplelincRNA.txt

Sample PC.txt : SamplePC.txt

Script : lincToPC.pl

To run the script : $bsub < lincToPC.pl OutputFileName

lincToPC.pl :https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/lincToPc.pl

Sample OutputFile : SampleLincAndPC.txt

**Step 12**.	ANTISENSE AND TARGETS : Extract all Antisense RNAs, chromosome, strand and genomic coordinates from step 10 and put them in a file called asRNA.txt. Extract all protein coding genes chromosome, strand and genomic coordinates from step 10 and put them in a file called PC.txt. Use the script called AntisenseToPConOppositeStrand.pl to identify the neighboring expressed protein coding genes

Sample asRNA.txt : SampleasRNA.txt

Sample PC.txt : SamplePC.txt

Script : antiOppGene.pl

antiOppGene.pl :https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/antiOppGene.pl


To run the script : $bsub < antiOppGene.pl  OutputFileName

Sample OutputFile : AsRNAOutput.txt

**Step 13**.	PSEUDOGENE AND PARENT PROTEIN CODING GENE (TARGETS)

The pseudogene sequences are mapped against the protein coding gene sequences using BLAST, and only the unique hits are  assigned to parent gene-pseudogene associations. 

Fasta sequences for protein-coding genes and pseudogenes are downloaded from the reference genome, Mus_musculus.GRCm38.dna_rm.primary_assembly.fa using the script (MusFastaExtract.pl)

MusFastaExtract.pl: https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MusFastaExtract.pl

These two sets of sequences are mapped against each other using BLAST using the script (MousePseudoToProtein.sh)and the parameter “-max_target_seqs :1” to detect only those protein-coding genes that have a high-level of sequence homology to the pseudogenes used as query. 

MousePseudoToProtein.sh: https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/MousePseudoToProtein.sh

Finally, the output obtained from BLAST analyzed to overcome the multiple associations, i.e. when a pseudogene aligns to multiple protein-coding genes. The BLAST output was filtered using the “sort –u” option, which sorts the file and pipes the output that contains only unique hits. Therefore, only unique hits are obtained for further analysis.

