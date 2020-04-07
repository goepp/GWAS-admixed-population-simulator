# GWAS-admixed-population-simulator
Simulating GWAS data in [PLINK](https://www.cog-genomics.org/plink/) format with GWAsimulator tool using Hapmap3 data

# Description
This repository presents a tutorial to simulate GWAS data using [GWAsimulator](http://biostat.mc.vanderbilt.edu/GWAsimulator) tool, it consists of:
- Use HapMap3 data as reference for GWAsimulator.
- Convert HapMap3 phased data in GWAsimulator input format.
- Adjust the generated data to [PLINK](https://www.cog-genomics.org/plink/) format (.ped and .map files).
- Generate admixed population data, this could be useful in population stratification studies.

Either this repository considers ```HapMap3``` data (ftp://ftp.ncbi.nlm.nih.gov/hapmap/phasing/2009-02_phaseIII/HapMap3_r2) as input reference, the algorithm could be used to other references data such as ```1000 Genome Project``` data.
 
11 populations were included in [HapMap3](https://www.sanger.ac.uk/resources/downloads/human/hapmap3.html), we will give an example of generating a data of two admixed populations for ```CEU``` (Utah residents with Northern and Western European ancestry from the CEPH collection) and ```YRI``` (Yoruba in Ibadan, Nigeria). You can change other populations or add more than 2 populations following the same procedure.

The choice of GWAsimulator input parameters (in ```control_ceu.txt``` and ```control_yri.txt```  files) will not be discussed in this repository, please check the [documentation](http://biostat.mc.vanderbilt.edu/wiki/pub/Main/GWAsimulator/GWAsimulator_v2.0.pdf) provided by the authors for details.

# What is GWAsimulator?
A C++ software that simulates genotype (SNP chips) for the whole human genome in case-control study. Different populations could be generated by following the Linkage Disequilibrium patterns of a reference data, such as HapMap3. 
In practice, the algorithm takes as input:
- Reference data: phased data in 23 files (one file for each chromosome: from 1 to 23 for chromosome X), the alleles must be decoded as ```0``` or ```1```. The program considers only  bi-allelic SNP.
- Control file: it includes information about the desired simulated data such as: output file format, disease locus position and the number of males and females generated samples...

# Requirements
- Python2
- [GWAsimulator](http://biostat.mc.vanderbilt.edu/GWAsimulator)
- pandas
- [textwrap](https://pypi.org/project/textwrap3/)
- [PLINK](https://www.cog-genomics.org/plink2)

# Tutorial
## Clone the repository
```
git clone  https://github.com/asmanouira/GWAS-admixed-population-simulator
cd GWAS-admixed-population-simulator/
```

## Download the HapMap3 phased data for both CEU and YRI populations

#### CEU population
```
mkdir ceu
cd ceu

prefix="ftp://ftp.ncbi.nlm.nih.gov/hapmap/phasing/2009-02_phaseIII/HapMap3_r2/CEU/TRIOS/hapmap3_r2_b36_fwd.consensus.qc.poly.chr";
suffix="_ceu.phased.gz";


for chr in {1..22}; do
	wget "${prefix}${chr}${suffix}"
done

wget ftp://ftp.ncbi.nlm.nih.gov/hapmap/phasing/2009-02_phaseIII/HapMap3_r2/chrX/hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_ceu.trios.phased.gz


gunzip *.gz
mv hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_ceu.trios.phased hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_ceu.phased 

cd ..
```
#### YRI population
```

mkdir yri
cd yri

prefix="ftp://ftp.ncbi.nlm.nih.gov/hapmap/phasing/2009-02_phaseIII/HapMap3_r2/YRI/TRIOS/hapmap3_r2_b36_fwd.consensus.qc.poly.chr";
suffix="_yri.phased.gz";


for chr in {1..22}; do
	wget "${prefix}${chr}${suffix}"
done

wget ftp://ftp.ncbi.nlm.nih.gov/hapmap/phasing/2009-02_phaseIII/HapMap3_r2/chrX/hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_yri.trios.phased.gz


gunzip *.gz
mv hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_yri.trios.phased hapmap3_r2_b36_fwd.consensus.qc.poly.chr23_yri.phased 

cd ..
```
## Convert phased data to GWAsimulator input format 

```
python
from utils import convert_phased
convert_phased('ceu')
convert_phased('yri')
```
## Generate simulated data using GWAsimulator 
<ol>
<li> Prepare the control_#.txt file for each population #.</li> 
<li> Specify in the first line the path of the converted phased data. </li>
<li> Run GWAsimulator: </li>
</ol>

The maximum number of SNPs in the phased files can exceeds ```100000 SNPs``` for some chromosomes which is the default setting of GWAsimulator software, so first let's increase this threshold in ```GWAsimulator.cpp``` in the following line:

```#define MAXSNP 1000000   // maximum number of SNPs per chromosome```

#### CEU population:
	
``` 
GWAsimulator control_ceu.txt [seed number]
```
More details on GWAsimulator [manual](http://biostat.mc.vanderbilt.edu/wiki/pub/Main/GWAsimulator/GWAsimulator_v2.0.pdf).

The program generates gzipped data to save disk space and named in ```chr#.dat.gz```, where ```#``` is the chromosome number. For that, we will gunzip and rename it as ```PLINK``` format:

```
gunzip *.gz
for f in *.dat; do
    mv -- "$f" "$(basename -- "$f" .dat)_ceu.ped"
done
```
#### YRI population:
``` 
GWAsimulator control_yri.txt [seed number]
```
And then,

```
gunzip *.gz
for f in *.dat; do
    mv -- "$f" "$(basename -- "$f" .dat)_yri.ped"
done
```
## Generate .map PLINK files for each chromosome per population

```
python
from utils import phased2map
phased2map('ceu')
phased2map('yri')
```

## Merge the multiple simulated files in one .ped PLINK format
```
plink --file chr1_ceu --merge-list allfiles_ceu.txt --make-bed --out simulated_data_ceu
plink --file chr1_yri --merge-list allfiles_yri.txt --make-bed --out simulated_data_yri
```

Here,```allfiles_ceu.txt``` was a list of the to-be-merged files, one set per row. Same for ```allfiles_yri.txt```.

## Merge both populations genotype files in one file to get an admixed population data

First, let's change the Individual IDs for one file, as both files contains the same IDs:

```
python
from utils import updateID
updateID(30)
```
This function takes the number of simulated samples as input, here we took ```30``` as we simulated 30 samples (see ```control_ceu.txt``` and ```control_yri.txt```)!
It prints a file ```ids.txt ```containing 4 columns ```old_FID, IID, new_FID, IID```.

Then, we use PLINK to update the IDs for YRI population data (as an example):

```
plink --bfile simulated_data_yri --update-ids ids.txt --make-bed --out simulated_data_yri
```
More information about ```--update-ids``` in PLINK [documentation](https://www.cog-genomics.org/plink/2.0/data).
 
Finally, we merge CEU and YRI data in one file:

```
plink --bfile simulated_data_ceu --bmerge simulated_data_yri.bed simulated_data_yri.bim simulated_data_yri.fam --make-bed --out simulated_data
```



# References:
Li C et al., GWAsimulator: a rapid whole-genome simulation program, Bioinformatics (2008).
