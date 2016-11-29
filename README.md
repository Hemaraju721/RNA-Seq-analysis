# RNA Seq Analysis to detect and categorize noncoding RNAs and further identify respective targets
### Introduction
RNA-Seq technology enables deep profiling of the transcriptome. Unlike microarray, RNA-Seq provides unbiased detection of novel transcripts, broader dynamic range for read counts, and easier detection of rare and low-abundance transcripts.
### Dataset
The raw dataset is in *.fastq format

### Methodology :

#### Quality Control
**Step 1**.	FASTQC: Prior to running RNA Seq alignment, it is important ro run fastqc http://www.bioinformatics.babraham.ac.uk/projects/fastqc/. 
Command line to run fastqc on unix is $FastQC DRG1.Fasta. 

This generates folder as an output with many report files. Open the index file after transferring to your windows desktop. All the green should ideally light up but that seldom happens. In the file there is a section "Per base sequence content" which should tell you how much to trim from either end. Usually it is better to choose base position which has variable content. Repeat the same steps for remaining samples as well.

**Step 2**.	TRIMMOMATIC: To do trimming use trimmomatic. Clip using the values you found using previous steps and also provide a minimum Mapping Q score (we used 20) you want to keep, but here more stringent values are preferred. Minimum length to be kept depends on your original sequence length. Don’t go below 36bp.

Code :  
java –jar  /projects/MainFolder/SubFolder/ProjName/Trimmomatic-0.36/trimmoatic-0.36.jar SE –phred64 Sample1.fastq Sample1_new.fastq HEADCROP:14 AVGQUAL:20 MINLEN:36.  Once you get the output files from trimmomatic use fastqc again to see if the green light come up for most sections. The Kmer section most of the time is yellow or red, so ignore. Once all clear, align against transcriptome/genome.

**Step 3**.	DOWNLOAD GENOME: In the Ensembl directory there are 3 versions of genome. repeatmasked(rm), small-masked(sm) and regular. 
"Mus_musculus.GRCm38.dna_rm.primary_assembly.fa.gz" [repeat masked entire genome]
Make sure to index them after you download them
$ wget -c 'ftp://ftp.ensembl.org/pub/release-84/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna_rm.primary_assembly.fa.gz'
cd ../rm_masked
gunzip Homo_sapiens.GRCh38.dna_rm.primary_assembly.fa.gz

**Step 4**.	To filter out non-conventional chromosomes using script(filter_genome.pl)

CODE for filter_genome.pl : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/filtergenome.pl

To run this script : $perl filtergenome.pl <Mus_musculus.GRCm38.dna_rm.primary_assembly.fa> <Mus_musculus.GRCm38_Rmasked.fa>

To build the index : $bowtie2-build <Mus_musculus.GRCm38_RMasked.fa><mm38>

To rename the file: $cp <Mus_musculus.GRCm38_RMasked><mm38.fa>

To create symbolic link : $ln-a <mm38.fa> <mm38.fa.fa>
$ ln -s Hs_RMasked.fa Hs_RMasked.fa.fa

####Alignment and transcriptome assembly

**Step 5**.	ALIGNMENT USING TOPHAT 2.0
To align the sample files against the mouse genome using TopHat2.0, use the following script.

Script : MyTopHat.sh
CODE for MyTopHat.sh : https://github.com/hrccspipeline/HRCCSPipeline/blob/master/scripts/DRG1.sh
To run the script : $bsub < MyTopHat.sh

**Step 6**.	TRANSCRIPT ASSEMBLY USING CUFFLINKS

To assemble the transcripts using cufflinks, use the following script

Script : DRG_CGN_Cufflinks.sh

To run the script : $bsub < MyCufflinks.sh

**Step 7**.	Combine all the transcript files created from Step 6:

Script : Cufflinks.gtf.list

To run the script : $bsub < Cufflinks.gtf.list

**Step 8**.	Merge GTF file along with the combined gtf file created from step 7

Script : Cuffmerge.sh

To run the script : $bsub < Cuffmerge.sh

**Step 9**.	Detect the normalized gene expression

Script : Cuffnorm.sh

To run the script : $bsub < Cuffnorm.sh

**Step 10**.	Categorize the noncoding RNAs
Script : ????

**Step 11**.	LINCRNA AND TARGETS : Extract all lincRNAs, chromosome and their genomic coordinates from step 10 and put them in a file called lincRNA.txt. Extract all protein coding genes, chromosome and their genomic coordinates from step 10 and put them in a file called PC.txt. Use the script called lincToPc.pl to identify the neighboring expressed protein coding genes

Sample lincRNA.txt : SamplelincRNA.txt

Sample PC.txt : SamplePC.txt

Script : lincToPC.pl

To run the script : $bsub < lincToPC.pl <OutputFileName>

Sample OutputFile : SampleLincAndPC.txt

**Step 12**.	ANTISENSE AND TARGETS : Extract all Antisense RNAs, chromosome, strand and genomic coordinates from step 10 and put them in a file called lincRNA.txt. Extract all protein coding genes chromosome, strand and genomic coordinates from step 10 and put them in a file called PC.txt. Use the script called AntisenseToPConOppositeStrand.pl to identify the neighboring expressed protein coding genes

Sample asRNA.txt : SamplelincRNA.txt

Sample PC.txt : SamplePC.txt

Script : AntisenseToPConOppositeStrand.pl

To run the script : $bsub < AntisenseToPConOppositeStrand.pl  <OutputFileName>

Sample OutputFile : AsRNAOutput.txt

**Step 13**.	PSEUDOGENE AND PARENT PROTEIN CODING GENE (TARGETS)


