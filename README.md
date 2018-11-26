# Oxford Nanopore sequencing tips, tricks, and hacks

**Made for the San Francisco Community meeting 2018**

Here is a collection of tips, tricks and hacks for use with nanopore data.
Not everything will still be valid as ONT update their file structures and data types.
Happy to include your favourite tip, and take requests if you have something really bugging you.

## File structures

The file structure is not overly important, however it will modify some of the commands used in various examples. I have endeavoured to include a few diverse uses, starting from different file states, but of course, I can't think of everything.

#### 1. Raw structure

This is the most basic structure, where all files are present in an accessible state. (**not preferred**)

    ├── huntsman.fastq
    ├── sequencing_summary.txt       
    ├── huntsman_reads/              # Read folder
    │   ├── 0/                       # individual folders containing ~4000 fast5s
    |   |   ├── huntsman_read1.fast5
    |   |   └── huntsman_read2.fast5
    |   |   └── ...
    |   ├── 1/
    |   |   ├── huntsman_read#.fast5
    |   |   └── ...
    └── ├── ...

#### 2. Local basecalled structure

This structure is the typical structure post local basecalling for long term storage
fastq and sequencing_summary files have been gzipped and the folders in the reads folder have been tarballed into one large file

    ├── huntsman.fastq.gz            # gzipped
    ├── sequencing_summary.txt.gz    # gzipped
    ├── huntsman_reads.tar           # Tarballed read folder
        |                            # Tarball expanded
        |-->│   ├── 0/               # individual folders inside tarball
            |   |   ├── huntsman_read1.fast5
            |   |   └── huntsman_read2.fast5
            |   |   └── ...
            |   ├── 1/
            |   |   ├── huntsman_read#.fast5
            |   |   └── ...
            └── ├── ...
                                     # or each folder is also tarballed (ideal)
        |-->│   ├── 0.tar
            |   ├── 1.tar
            |   ├── #.tar
            └── ├── ...

#### 3. Parallel basecalled structure

This structure is post massively parallel basecalling, and looks like multiples of the above structure.

    ├── fastq/
    |   ├── huntsman.1.fastq.gz
    |   └── huntsman.2.fastq.gz
    |   └── huntsman.3.fastq.gz
    |   └── ...
    ├── logs/
    |    ├── sequencing_summary.1.txt.gz
    |    └── sequencing_summary.2.txt.gz
    |    └── sequencing_summary.3.txt.gz
    |    └── ...
    ├── fast5/
    |    ├── 1.tar
    |    └── 2.tar
    |    └── 3.tar
    |    └── ...

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

# ONT H4X

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

* * *

## More to come!!!

I'll be expanding this soon.
Also, if you have some quick and dirty hacks, let me know, i'll chuck them in.

## Acknowledgements

I would like to thank my lab (Shaun Carswell, Kirston Barton, Hasindu Gamaarachchi, Kai Martin) in Genomic Technologies team from the [Garvan Institute](https://www.garvan.org.au/) for their feedback on the development of these scripts.

## License

[The MIT License](https://opensource.org/licenses/MIT)
