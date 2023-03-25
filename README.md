Quick start
Preparation for software:
Download bwa via https://sourceforge.net/projects/bio-bwa/files/
Download SAMtools via http://samtools.sourceforge.net/

Preparation for database:
Download HG19 reference via http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.2bit
Download Splited HG19 reference via http://hgdownload.cse.ucsc.edu/goldenPath/hg19/chromosomes/
Concatenate as hg19.fa.

Data preparation
Input files include: 2 pair-end FQ files with for one sample, if not 1-fold 100bp, please head 15,000,000 reads and trim 100bp per FQ file.

Workflow
Step1:
For every sample, 2 FQ files as input. Taken TestID as sample ID, MGISEQ as sequencing platpform:
bwa mem -t 30 -R '@RG\\tID:"+TestID+"\\tLB:"+ TestID +"\\tPL:MGISEQ\\tPU:"+ TestID +"\\tSM:"+ TestID +"' -M hg19.fa test.read_1.fq.gz test.read_2.fq.gz | samtools view -b -S -T hg19.fa - > test.bam

Step2:
(1) Sort: samtools sort -@ 30 -o test.sort.bam test.bam
(2) Remove duplications: samtools rmdup test.sort.bam test.sort.rmdup.bam
(3) Index: samtools index -@ 30 test.sort.rmdup.bam

Step 3:
For trio-based analysis, write a file with absolute paths of bam files in proband-mother-father order, for example: vi bam.lst, then paste paths of bam files in the bam.lst:
/ABSOLUTE_PATH/Proband. sort.rmdup.bam
/ABSOLUTE_PATH/Mother. sort.rmdup.bam
/ABSOLUTE_PATH/Father. sort.rmdup.bam
For duo-based analysis, write a file with absolute paths of indexed bam files in proband-parent order:
/ABSOLUTE_PATH /Proband. sort.rmdup.bam
/ABSOLUTE_PATH /Parent. sort.rmdup.bam

Step 4:
Mpileup with SAMtools for chromosome 1-22. For example, mpileup for chromosome 1:
samtools mpileup -A -f hg19.fa -b bam.lst -r chr1 | gzip -9 > bam.lst.chr1.gz

Step 5:
Perform extraction step for chromosome 1-22. For example, extracting chromosome 1:
(a): For trio-based analysis:
./extract_family.pl bam.lst.chr1.gz bam.lst.chr1.extract.gz
(b): For duo-based analysis:
./extract_duo.pl bam.lst.chr1.gz bam.lst.chr1.extract.gz

Step 6:
List all chromosome extract.gz files into one file:
ls *.extract.gz > extract.lst
(a) For trio-based analysis:
./TriosInRate extract.lst 
./DuosInRate extract.lst 

The inconsistent rate with the parental/maternal relationship result would be generated.
