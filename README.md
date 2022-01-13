# bulk ATAC seq data analysis 

<br>
<br>

1. Install nf-core/atacseq<br>
https://nf-co.re/atacseq/1.2.1

```
curl -s https://get.nextflow.io | bash
conda create --name nf-core python=3.7 nf-core nextflow
conda activate nf-core
##(nf_core)$ nf-core download
```



2. Install RGT<br> 
http://www.regulatory-genomics.org/hint/introduction/

```
pip install --user RGT
```




3. Run nf-core/atacseq


In Python 3 


	3-1. to make output dir
	
		```
		mkdir /xxx/7_nextflow_out
		```
		
	3-2. to prepare the shell script to run nf-core/atacseq
		
		```	
		#!/usr/XXX/bin/zsh
		#SBATCH -J run_nf-core_atac
		#SBATCH -t 100:00:00
		#SBATCH --ntasks-per-node=20
		#NXF_OPTS='-Xms1g -Xmx4g'
		#SBATCH --output=output.%J.txt
	
		source /xxx/anaconda3/bin/activate nf-core
	
		dir_fq=/xxx/bulk_ATAC/input/
		dir_csv=/xxx/bulk_ATAC/input/design.csv
	
	
		cd $dir_fq
		/xxx/nextflow run nf-core/atacseq --input $dir_csv --narrow_peak --genome mm10 --outdir '/xxx/7_nextflow_out'	
		```	

	Here, 'xxx' should be your full directory 







4. Run HINT


	4-1. to download the genome data 
	
	```
	# The directory "rgtdata" would be downloaded in your HOME when you install "RGT" (No.2)
	cd ~/rgtdata
	python setupGenomicData.py --mm10
	
	# you can check your genome name just by typing "python setupGenomicData.py"
	```





	4-2. To change chromosome name (optional)
	
	https://www.biostars.org/p/13462/
		

	```
	#!/usr/XXX/bin/zsh
	#SBATCH -J samtools
	#SBATCH -t 100:00:00
	#SBATCH --output=output.%J.txt
	
	source /xxx/anaconda3/bin/activate nf-core
	
	bam_dir="/xxx/7_nextflow_out/bwa/mergedLibrary"
	
	for file in $bam_dir/*sorted.bam
	do
	  filename=`echo $file | cut -d "." -f 1`
	  samtools view -H $file | \
	      sed -e 's/SN:1/SN:chr1/' | sed -e 's/SN:2/SN:chr2/' | \
	      sed -e 's/SN:3/SN:chr3/' | sed -e 's/SN:4/SN:chr4/' | \
	      sed -e 's/SN:5/SN:chr5/' | sed -e 's/SN:6/SN:chr6/' | \
	      sed -e 's/SN:7/SN:chr7/' | sed -e 's/SN:8/SN:chr8/' | \
	      sed -e 's/SN:9/SN:chr9/' | sed -e 's/SN:10/SN:chr10/' | \
	      sed -e 's/SN:11/SN:chr11/' | sed -e 's/SN:12/SN:chr12/' | \
	      sed -e 's/SN:13/SN:chr13/' | sed -e 's/SN:14/SN:chr14/' | \
	      sed -e 's/SN:15/SN:chr15/' | sed -e 's/SN:16/SN:chr16/' | \
	      sed -e 's/SN:17/SN:chr17/' | sed -e 's/SN:18/SN:chr18/' | \
	      sed -e 's/SN:19/SN:chr19/' | sed -e 's/SN:20/SN:chr20/' | \
	      sed -e 's/SN:21/SN:chr21/' | sed -e 's/SN:22/SN:chr22/' | \
	      sed -e 's/SN:X/SN:chrX/' | sed -e 's/SN:Y/SN:chrY/' | \
	      sed -e 's/SN:MT/SN:chrM/' | samtools reheader - $file > ${filename}_chr.bam
	done
	
	
	``` 




	4-3. To make index file for the ${filename}_chr.bam (optional, followed by step 4-2)
	
	in Python3
	
	```
	import os,sys
	import glob
	import pysam
		
	bam_dir="/xxx/7_nextflow_out/bwa/mergedLibrary"
	files = glob.glob(bam_dir+"/*_chr.bam")
	for a_file in files :
	        pysam.index(a_file)
	
	```






	4-4. To run HINT
	
	https://www.regulatory-genomics.org/hint/tutorial/
	
	```
	#!/usr/local_rwth/bin/zsh
	#SBATCH -J HINT
	#SBATCH -t 100:00:00
	#SBATCH --output=output.%J.txt
	
	bam_dir="/xxx/7_nextflow_out/bwa/mergedLibrary"
	cd ${bam_dir}
	for file in *chr.bam
	do
        	filename=`echo $file | cut -d "_" -f 1`
        	/xxx/.local/bin/rgt-hint footprinting --atac-seq --paired-end --organism=mm10 --output-location=$dir1/Footprints --output-prefix=${filename} ${filename}_R1.bam $peak_dir/${filename}_R1.mLb.clN_peaks.narrowPeak
	done
	```	
	
	
	
	
5. Special thanks to 
	
http://www.costalab.org
	
	
	
**References & Good Q&A web source**

https://nf-co.re/atacseq/1.2.1<br>
http://www.regulatory-genomics.org/hint/introduction/<br>
https://www.biostars.org/p/13462/<br>
https://www.regulatory-genomics.org/hint/tutorial/<br>
	


	
