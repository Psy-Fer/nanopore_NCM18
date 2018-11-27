# Oxford Nanopore sequencing tips, tricks, and hacks

**Made for the San Francisco Community meeting 2018**

Here is a collection of tips, tricks and hacks for use with nanopore data.
Not everything will still be valid as ONT update their file structures and data types.
Happy to include your favourite tip, and take requests if you have something really bugging you.


# ONT H4X


##### Combine fastq.gz files

Kept simple.

```bash
for file in fastq/*.fastq.gz; do cat $file; done >> huntsman.fastq.gz
```

##### Combine sequencing_summary.txt.gz files

```bash
# create header
zcat $(ls logs/sequencing_summary*.txt.gz | head -1) | head -1 > sequencing_summary.txt

# combine all files, skipping first line header
for file in logs/sequencing_summary*.txt.gz; do zcat $file | tail -n +2; done >> sequencing_summary.txt

gzip sequencing_summary.txt
```

#### dump fastq from fast5

**install  hdf5-tools**

    sudo apt install hdf5-tools

```bash
    FILE=your_file.fast5
    # Get summary of contents (parent directories)
    h5ls $FILE
    # Get more detailed contents (all directories)
    h5ls -r $FILE
    # Print out the whole file
    h5dump $FILE
    # Print out a specific entry (directory) in the file
    # Here, the Fastq entry is extracted. Use h5ls -r to see options for -d
    h5dump -d /Analyses/Basecall_1D_000/BaseCalled_template/Fastq $FILE
```

#### Get read lengths

```bash
    awk '{if (NR%4==2) print length}' file.fastq > read_lens.txt
```

#### Extract a set of reads from a large fastq file in parallel

```bash
    LIST=identifiers.txt
    FASTQ=largeFile.fastq
    LINES=$( awk 'END { print NR }' ${FASTQ} )
    ## NUM needs to be a multiple of four
    NUM=8
    split -l $(( ${LINES} / %{NUM} )) $FASTQ
    for file in x?? ; do grep -A 3 -f ${LIST} ${file} > ${file}.grep & ; done
    cat *.grep > ${FASTQ%*.fastq}_extracted.fastq
    #rm *.grep
```

#### Dirty cmd lind way to visualise fastq quality scores

```bash
    sed -e 'n;n;n;y/!"#$%&'\''()*+,-.\/0123456789:;<=>?@ABCDEFGHIJKL/▁▁▁▁▁▁▁▁▂▂▂▂▂▃▃▃▃▃▄▄▄▄▄▅▅▅▅▅▆▆▆▆▆▇▇▇▇▇██████/' example.fastq
```

## Tool tips

Tips for using various nanopore software tools

Here is a link to the most up to date set of nanopore tools I have found yet:
<https://tinyurl.com/yar8r6wp>

If your tool is missing, please add it with a short description.


#### Nanopolish

When using Nanopolish, be sure to use the -s option to use the sequencing summary files. This speeds up the indexing dramatically.

    nanopolish index -d /path/to/raw_fast5s/ -s sequencing_summary.txt albacore_output.fastq

If you are multiplexing samples, filter your reads and files with fast5_fetcher first to get even more speedups. (<https://github.com/Psy-Fer/fast5_fetcher>)

#### Albacore

When basecalling with albacore, one dirty method of getting progress is to compare the number of files in the workspace, with the number of files in the sequencing_summmary file.

    ls -U workspace | wc -l; wc -l sequencing_summary.txt

#### Installing albacore into a conda environment using pip wheel
Requirements
  * `Python` 2.7, 3.4, 3.5, and/or 3.6 (conda requirement)
  * `Conda` - check by running `conda list`
  * `wget`

1. Create a new conda environment with python version 3.4, 3.5 or 3.6. Replace `3.5` with `3.4` or `3.6` in the command below to install those specific versions of python into your conda environment. You can also make the name of your environment whatever you want.
```bash
conda create --name albacore-env python=3.5
```
2. Download Albacore pip wheel for your specific python3 version from ONT's software downloads page (ONT account required). Right click on a blue "Python 3.X" button to copy the link to your clipboard. https://community.nanoporetech.com/downloads 
```bash
wget https://mirror.oxfordnanoportal.com/software/analysis/ont_albacore-2.3.3-cp35-cp35m-manylinux1_x86_64.whl
```
3. Activate your conda environment, check that pip 8.1 or higher is installed in the environment
```bash
conda activate albacore-env
(albacore-env) user@host:~$ conda list
```
4. Install Albacore using pip wheel that you just downloaded (while conda env is active!), check that all dependencies were installed.
```bash
(albacore-env) user@host:~$ pip install ont_albacore-2.3.3-cp35-cp35m-manylinux1_x86_64.whl
(albacore-env) user@host:~$ conda list
# output should look similar to this:
# packages in environment at /home/user/miniconda3/envs/albacore-env:
#
# Name                    Version                   Build  Channel
bzip2                     1.0.6                h470a237_2    conda-forge
ca-certificates           2018.10.15           ha4d7672_0    conda-forge
certifi                   2018.8.24             py35_1001    conda-forge
h5py                      2.8.0                     <pip>
libffi                    3.2.1                hfc679d8_5    conda-forge
libgcc-ng                 7.2.0                hdf63c60_3    conda-forge
libstdcxx-ng              7.2.0                hdf63c60_3    conda-forge
ncurses                   6.1                  hfc679d8_1    conda-forge
numpy                     1.15.4                    <pip>
ont-albacore              2.3.3                     <pip>
ont-fast5-api             1.0.1                     <pip>
openssl                   1.0.2p               h470a237_1    conda-forge
pip                       18.0                  py35_1001    conda-forge
progressbar33             2.4                       <pip>
python                    3.5.5                h5001a0f_2    conda-forge
python-dateutil           2.7.5                     <pip>
readline                  7.0                  haf1bffa_1    conda-forge
setuptools                40.4.3                   py35_0    conda-forge
six                       1.11.0                    <pip>
sqlite                    3.25.3               hb1c47c0_0    conda-forge
tk                        8.6.9                ha92aebf_0    conda-forge
wheel                     0.32.0                py35_1000    conda-forge
xz                        5.2.4                h470a237_1    conda-forge
zlib                      1.2.11               h470a237_3    conda-forge
```
5. Test Albacore install. Should see help options appear.
```bash
(albacore-env) user@host:~$ read_fast5_basecaller.py -h
```

## More to come!!!

I'll be expanding this soon.
Also, if you have some quick and dirty hacks, let me know, i'll chuck them in.

## Acknowledgements

I would like to thank my lab (Shaun Carswell, Kirston Barton, Hasindu Gamaarachchi, Kai Martin) in Genomic Technologies team from the [Garvan Institute](https://www.garvan.org.au/) for their feedback on the development of these scripts, as well as the nanopore community for their input (attribution added best I could, if I have something wrong, let me know)

## License

[The MIT License](https://opensource.org/licenses/MIT)
