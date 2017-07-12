*HIFI-BARCODES-PACBIO-PIPELIN* User's Guide V1.0 20170707


### DESCRIPTION
HIFIBarcode is used to produce full-length COI barcodes from pooled PCR
amplicons generated by individual specimens.

### INSTALLATION
- Clone from github
```bash
$ git clone https://github.com/comery/HIFI-barcode-hiseq.git
```
- Download a ZIP file
Go to website https://github.com/comery/HIFI-barcode-hiseq and click 'Download ZIP'
### Requirements 
#### (1) software 
- PicBio smrtanalysis | http://www.pacb.com/products-and-services/analytical-software/smrt-analysis/

#### (2) programming language
 - standard perl
 - standard python(python2 is ok)

#### (3) perl module
 - Bio::Perl( exactlly Bio::Seq )

#### (4) main perl and python scripts in bin/
 - 1.primer_like_extract.pl
 - 2.cluster_count_passes_length.pl
 - ccs_passes.py | copy from [here](https://github.com/PacificBiosciences/Bioinformatics-Training/raw/master/scripts/ccs_passes.py)
 - fish_ccs.pl

### DATA requirements:

#### (1) pacbio original H5 file input
 - 01.data/*.h5 ( linkage will be available soon )
#### (2) primers list
 -	experiment_data/primer.lst
	primer.lst  like this:
	for     GGTCAACAAATCATAAAGATATTGG
	rev     TAAACTTCAGGGTGACCAAAAAATCA

#### (3) index(barcodes for identifying samples) list
 -  experiment_data/index.xls
	index.xls like this:
	001     AAAGC
	002     AACAG
	003     AACCT
	004     AACTC
	005     AAGCA
	...		.....

#### (4) samples_location.tab
- samples name and corresponding location in 96-cell plate
	like this:
	1	A01
	2	B01
	3	C01
	4	D01
	5	E01
	.	...

### Overview of steps

If you installed PacBio smrtanalysis, I suppose you get the  file path of setup.sh,
more about Pacbio Data : http://www.pacb.com/wp-content/uploads/SMRT-Link-User-Guide-v4.0.0.pdf

e.g:
	setup_path='/path/PicBio/smrtanalysis/current/etc/setup.sh'

#### step 1 extract CCS from h5 files
Input:
- my_inputs.fofn

Output:
- log
- data
	*.ccs.fasta
	*.ccs.fastq
	*.ccs.h5
	reads_of_insert.fasta
	reads_of_insert.fastq
	slots.pickle

- workflow
- results

my_inputs.fofn contains files list of Pacbio H5 file in 01.data/, like this:
	./01.data/m170506_092957_42199_c101149142550000001823255607191735_s1_p0.1.bax.h5
	./01.data/m170506_092957_42199_c101149142550000001823255607191735_s1_p0.bas.h5
	./01.data/m170506_092957_42199_c101149142550000001823255607191735_s1_p0.3.bax.h5
	./01.data/m170506_092957_42199_c101149142550000001823255607191735_s1_p0.2.bax.h5

run:
```bash
$ fofnToSmrtpipeInput.py my_inputs.fofn > my_inputs.xml
$ smrtpipe.py --params=settings.xml xml:input.xml
```

#### step 2 extract passes number from CCS h5 files
Input:
- /data/*.ccs.h5 

Output:
- ccs_passes.lst

run:
```bash
$ python bin/ccs_passes.py  data/*.ccs.h5 >ccs_passes.lst
```

#### step 3 filtering CCS by passes number (>15)
Input:
- ccs_passes.lst
- data/reads_of_insert.fasta

Output:
- ccs_passes_15.fa

run:
```bash
$ awk '$2>=15{print $1}' ccs_passes.lst >ccs_passes_15.lst
$ perl ./bin/fish_ccs.pl ccs_passes_15.lst data/reads_of_insert.fasta >ccs_passes_15.fa
```
#### step 4 assigning CCS  to samples by index
Input:
- experiment_data/primer.fa
- experiment_data/index.xls
- ccs_passes_15.fa

Output: "outdir" name is up to you, here is 02.assignment/

02.assignment/
- assign.log.txt
- ccs.successfully_assigned.fa
- check.ccs_passes_15.fa.log

run:
```bash
$ perl ./bin/1.primer_like_extract.pl -p experiment_data/primer.fa -index experiment_data/index.xls -fa ccs_passes_15.fa -cm 2 -cg 1 -o outdir
```
#### step 5 clustering CCS of each sample to find best one
Input:
- ccs.successfully_assigned.fa
- check.ccs_passes_15.fa.log
- ccs_passes.lst

Output:
- cluster.top1.fas
- cluster.id.txt
- cluster.all.fa

run:
```bash
$ cd 02.assignment/
$ perl ../bin/cluster_lens_count.pl -ccs ../ccs_passes_15.fa -pattern check.ccs_passes_15.fa.log -passes ccs_passes.lst
$ perl ../bin/change_name-location.pl cluster.top1.fas >hifi-barcode-pacbio.cluster.top1.fa
```

ALL DONE!
	
So, "hifi-barcode-pacbio.cluster.top1.fa" is final result!

-------------------------------------------------------------------------------------------
### CONTACT US

Email:
yangchentao at genomics dot cn

### CITATION
Liu, Yang, Zhou, Zhou.(2017). Filling reference gaps via assembling DNA barcodes using high-throughput sequencing - moving toward to barcoding the world. Unpublished.

### LATEST RELEASE
Version 1.0 201707


