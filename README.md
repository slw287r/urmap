# urmap
URMAP ultra-fast read mapper

Home page [https://drive5.com/urmap](https://drive5.com/urmap)

## URMAP quick start

Download the binary file to a directory in your path. Make sure the binary file has execute permission (`chmod +x`) and its directory is in your `PATH` variable (or define a shell alias 'urmap' with the full path name).

Use the `make_ufi` command to create an index for the reference genome. Input is FASTA. Currently, **urmap is not alt-aware and I therefore recommend that alt chromosomes should be removed from the human reference**.

```
urmap -make_ufi hg38.fa -output hg38.ufi
```

Use the `map2` command to map paired reads, alignments in SAM format.

```
urmap -map2 sample_R1.fastq.gz -reverse sample_R2.fastq.gz \
  -ufi hg38.ufi -samout sample.sam
```

Use the `map` command to map unpaired reads.

```
urmap -map sample.fastq.gz -ufi hg38.ufi -samout sample.sam
```

## `make_ufi` command

Builds a reference genome index in UFI (Ultra-Fast Index) format. Input is a FASTA file containing the reference genome.

Currently, urmap is not alt-aware and I therefore recommend that alt chromosomes should be removed from the human reference.

**Example**

```
urmap -make_ufi hg38.fa -veryfast -output hg38.ufi
```
**Description**

The UFI index file is roughly 8 to 10x larger than the FASTA file. The UFI file must fit into available memory, so as a rule of thumb your RAM size should be at least 10x the genome size. For the human genome, 32Gb RAM should be enough with default options though the index may build more quickly if you have more RAM.

For the human genome, it typically takes half an hour to an hour to build the index, depending on CPU and disk speed.

Currently, the maximum supported genome size is 4Gb. Support for larger genomes is in development, let me know if you need this feature.

**Options**

The FASTA filename for the reference genome must be specified following -make_ufi.

The UFI file name is specified by the -output option (required).

The `-veryfast` option optimizes the index for use with the -veryfast option of the mapping commands (map and map2). Note that an index built with `-veryfast` will be less accurate regardless of whether `-veryfast` is specified while mapping. If you want to map both with and without `-veryfast`, you should create two separate indexes.

**Obscure / advanced options you probably don't need**

The `-wordlength` option sets the k-mer length. Default 24.

The `-maxix` option is an integer value setting the maximum abundance of a k-mer hash which is indexed. Default 32, or 3 if -veryfast is used.

The `-load_factor` option is a floating-point number < 1 specifying the fraction of the hash table slots which will be occupied when the index has been built. Default is 0.6. For technical reasons, this value is lower than typically used in hash tables. Increasing the load factor to, say, 0.7 significantly degrades index performance. Using values smaller than 0.6 may given some improvement in performance (probably not much) at the expense of increased index size.

The `-slots` option specifies the size of the hash table. This should be a prime number substantially larger than the genome size (number of bases). It is usually better to use the -load_factor option to specify the hash table size. Default is to calculate a suitable prime number given the genome size and desired load factor.

## `map` command

Maps unpaired reads to a reference genome. The reference genome is stored as a UFI file created by the make_ufi command. The original FASTA file for the reference is not needed.

Reads must be in FASTQ format or gzip-compressed FASTQ format. It is generally faster to use uncompressed reads because decompression is typically more expensive than the additional file i/o time required to read the larger uncompressed file.

**Options**

The FASTQ filename is specified following `-map`. Use "-" (minus sign) to get input from a pipe. If the reads are gzip-compressed, the filename must end in `.gz`. Piped input must be decompressed (you can always add gunzip to the pipe if needed).

The reference genome index filename is given by the `-ufi` option.

A filename for SAM format output is specified by `-samout`. Use `/dev/stdout` or "`-`" (minus sign) if you want to pipe output to another program.

The `-threads` option specifies the number of processor threads to use for mapping. Default is the number of CPU cores or 10, whichever is smaller.

The `-veryfast` option specifies a variant mapping algorithm which is ~3x faster but somewhat less accurate. To get the full speed improvement, the index should also be built with the `-veryfast` option, though UFI indexes are compatible with map regardless of whether -veryfast is used for mapping or for indexing.

**Pipes**

To get input from a pipe, use `/dev/stdin` as the filename.

To send output to a pipe, use "`-`" (minus sign) as the filename. Using `/dev/stdout` will not work because urmap attempts to create the file.

**Example**
```
urmap -map reads.fastq -ufi hg38.ufi -samout reads.sam
```

## `map2` command

Maps paired reads to a reference genome. This command is almost identical to map except that the -reverse option is used to specify the reverse (R2) FASTQ filename. See documentation for the map command for options and discussion.

**Example**

```
urmap -map2 sample_R1.fastq.gz -reverse sample_R2.fastq.gz -ufi hg38.ufi \
  -samout sample.sam
```

## `sam2aln` command

Convert alignments in SAM format to a human-readable, BLAST-like format.

**Options**

The SAM filename to convert is specified following `-sam2aln`.

A FASTA filename for the reference genome is specified by the `-ref` option (required).

The output filename is specified by the `-output` option. To send output to a pipe, use `/dev/stdout` as the filename.

**Example**

```
urmap -sam2aln reads.sam -ref hg38.fasta -output reads.aln
```
