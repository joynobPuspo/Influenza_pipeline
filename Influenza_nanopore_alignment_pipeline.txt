# Run this bash file in linux OS
###Automated cat file run
##Open the terminal in your data file or in terminal change your path to the working directory where all the required data are stored
#here, change the seq range like from:initial barcode number to:end barcode number
for i in $(seq 41 69)
do
  cat barcode$i/*.fastq > barcode$i/barcode$i.fastq
done 

#change the directoty in the LOC(location/directory file) name according to your batch name.like in this case, you can make the directory - Batch_09
LOC=ALL
mkdir $LOC
cp barcode*/barcode*.fastq $LOC

###Automated pipeline for sequence alignment
#change ONLY the REF and LOC file in first two command, (and if necessary change *.fastq/*.fastq.gz in Barcode variable )


#In REF file, select your own reference of interest(WITH APPROPRIATE PATH FILE!)
REF=/home/nanopore/nanopore/Reference_fasta/Influenza_AllReference

#make directories and select the path on the basis of where you want to store these file and on which path
mkdir $LOC/Porechopped $LOC/Aligned_Sam $LOC/Aligned_Bam $LOC/Sorted_Bam $LOC/Coverage $LOC/Qualimap $LOC/Consensus $LOC/Fasta_files

for Barcode in $LOC/*.fastq
do
     IFS='.' read -r -a array <<< $Barcode
     porechop -i $Barcode -o "${array[0]}_porechopped.fastq" --discard_middle
     minimap2 -ax map-ont $REF "${array[0]}_porechopped.fastq" > "${array[0]}_aln.sam"
     samtools view -S -b "${array[0]}_aln.sam" > "${array[0]}_aln.bam"
     samtools sort "${array[0]}_aln.bam" -o "${array[0]}_sorted.bam"
     samtools depth "${array[0]}_sorted.bam" > "${array[0]}.coverage"
     qualimap bamqc -bam "${array[0]}_sorted.bam" -outfile "${array[0]}.pdf" -outformat pdf
     samtools mpileup -uf $REF "${array[0]}_sorted.bam" | bcftools call -c | vcfutils.pl vcf2fq > "${array[0]}_consensus.fastq"
     seqtk seq -a "${array[0]}_consensus.fastq" > "${array[0]}.fasta"
     mv "${array[0]}_porechopped.fastq" $LOC/Porechopped
     mv "${array[0]}_aln.sam" $LOC/Aligned_Sam
     mv "${array[0]}_aln.bam" $LOC/Aligned_Bam
     mv "${array[0]}_sorted.bam" $LOC/Sorted_Bam
     mv "${array[0]}.coverage" $LOC/Coverage
     mv "${array[0]}_consensus.fastq" $LOC/Consensus
     mv "${array[0]}.fasta" $LOC/Fasta_files
done

