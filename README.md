# Oxford Nanopore sequencing tips, tricks, and hacks

**Made for the San Francisco Community meeting 2018**

Here is a collection of tips, tricks and hacks for use with nanopore data.
Not everything will still be valid as ONT update their file structures and data types.
Happy to include your favourite tip, and take requests if you have something really bugging you.


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

#### Dirty cmd lind way to visualise fastq quality scores

```bash
    sed -e 'n;n;n;y/!"#$%&'\''()*+,-.\/0123456789:;<=>?@ABCDEFGHIJKL/▁▁▁▁▁▁▁▁▂▂▂▂▂▃▃▃▃▃▄▄▄▄▄▅▅▅▅▅▆▆▆▆▆▇▇▇▇▇██████/' example.fastq
```



## More to come!!!

I'll be expanding this soon.
Also, if you have some quick and dirty hacks, let me know, i'll chuck them in.

## Acknowledgements

I would like to thank my lab (Shaun Carswell, Kirston Barton, Hasindu Gamaarachchi, Kai Martin) in Genomic Technologies team from the [Garvan Institute](https://www.garvan.org.au/) for their feedback on the development of these scripts.

## License

[The MIT License](https://opensource.org/licenses/MIT)
