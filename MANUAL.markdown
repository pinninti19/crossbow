%
% Bowtie: an Ultrafast, Lightweight Short Read Aligner
% by Ben Langmead and Cole Trapnell
%
% This manual is written in "markdown" format and thus contains some
% mildly distracting visual clutter encoding information about how to
% convert to HTML.  The document is still quite readable as raw text,
% but if the clutter is too distracting, try the man or HTML versions
% instead.
%

What is Bowtie?
===============

[Bowtie] is an ultrafast, memory-efficient short read aligner geared
toward quickly aligning large sets of short DNA sequences (reads) to
large genomes. It aligns 35-base-pair reads to the human genome at a
rate of 25 million reads per hour on a typical workstation. Bowtie
indexes the genome with a Burrows-Wheeler index to keep its memory
footprint small: for the human genome, the index is typically about
2.2 GB (for unpaired alignment) or 2.9 GB (for paired-end alignment).
Multiple processor cores can be used simultaneously to achieve greater
alignment speed.  Bowtie can also output alignments in the standard
[SAM] format, allowing Bowtie to interoperate with other tools
supporting SAM, including the [SAMtools] consensus, SNP, and indel
callers.  Bowtie runs on the command line under Windows, Mac OS X,
Linux, and Solaris.

[Bowtie] also forms the basis for other tools, including [TopHat]: a
fast splice junction mapper for RNA-seq reads, [Cufflinks]: a tool for
transcriptome assembly and isoform quantitiation from RNA-seq reads,
and [Crossbow]: a cloud-computing software tool for large reseuqncing
projects.

If you use [Bowtie] for your published research, please cite the
[Bowtie paper].

[Bowtie]:       http://bowtie-bio.sf.net
[SAM]:          http://samtools.sourceforge.net/SAM1.pdf
[SAMtools]:     http://samtools.sourceforge.net/
[TopHat]:       http://tophat.cbcb.umd.edu/
[Cufflinks]:    http://cufflinks.cbcb.umd.edu/
[Crossbow]:     http://bowtie-bio.sf.net/crossbow
[Bowtie paper]: http://genomebiology.com/2009/10/3/R25

What isn't Bowtie?
==================

Bowtie is not a general-purpose alignment tool like [MUMmer], [BLAST]
or [Vmatch].  Bowtie works best when aligning short reads to large
genomes, though it supports arbitrarily small reference sequences and
reads as long as 1024 bases.  Bowtie is designed to be extremely fast
for sets of reads where a) many of the reads have at least one good,
valid alignment, b) many of the reads are relatively high-quality, and
c) the number of alignments reported per read is small (close to 1).

Bowtie does not yet report gapped alignments; this is future work.

[MUMmer]: http://mummer.sourceforge.net/
[BLAST]:  http://blast.ncbi.nlm.nih.gov/Blast.cgi
[Vmatch]: http://www.vmatch.de/

Obtaining Bowtie
================

You may download either Bowtie sources or binaries for your platform
from the [Download] section of the Sourceforge project site.  Binaries
are currently available for Intel architectures (i386 and x86_64)
running Linux, Windows, and Mac OS X.

Building from source
--------------------

Building Bowtie from source requires a GNU-like environment that
includes GCC, GNU Make and other basics.  It should be possible to
build Bowtie on a vanilla Linux or Mac installation.  Bowtie can also
be built on Windows using [Cygwin] or [MinGW] (MinGW recommended).  If
building with MinGW, first install MinGW and [MSYS], the [zlib]
library, and the [pthreads] library.  You may also need the [GnuWin32]
core and other utilities to drive the build process.

Extract the sources, change to the directory where they were
extracted, and build the Bowtie tools by running GNU `make` (usually
with the command `make`, but sometimes with `gmake`) with no
arguments.  If building with MinGW, run `make` from the MSYS
environment.

Due to the `-p` option, Bowtie needs the pthreads library in order to
compile and run with multithreaded support.  To compile Bowtie without
pthreads (which disables `-p`), use `make BOWTIE_PTHREADS=0`.

[Cygwin]:   http://www.cygwin.com/
[MinGW]:    http://www.mingw.org/
[MSYS]:     http://www.mingw.org/wiki/msys
[zlib]:     http://cygwin.com/packages/mingw-zlib/
[pthreads]: http://sourceware.org/pthreads-win32/
[GnuWin32]: http://gnuwin32.sf.net/packages/coreutils.htm
[Download]: https://sourceforge.net/projects/bowtie-bio/files/bowtie/

Using the `bowtie` Aligner
--------------------------

`bowtie` takes an index and a set of reads as input and outputs a list
of alignments.  Alignments are selected according to a combination of
the `-v/-n/-e/-l` options (plus the `-I/-X/--fr/--rf/--ff` options for
paired-end alignment), which define which alignments are legal, and the
`-k/-a/-m/--best/--strata` options which define which and how many
legal alignments should be reported.

By default, Bowtie enforces an alignment policy similar to [Maq]'s
default quality-aware policy (`-n 2 -l 28 -e 70`).  See [the -n
alignment mode] section of the manual for details about this mode.  But
Bowtie can also enforce a simpler end-to-end k-difference policy (e.g.
with `-v 2`).  See [the -v alignment mode] section of the manual for
details about that mode.

Bowtie works best when aligning short reads to large genomes (e.g.
human or mouse), though it supports arbitrarily small reference
sequences and reads as long as 1024 bases.  Bowtie is designed to be
very fast for read sets where a) many of the reads have at least one
good, valid alignment, b) many of the reads are relatively high-
quality, c) the number of alignments reported per read is small (close
to 1).  These criteria are generally satisfied in the context of modern
short-read analyses such as RNA-seq, ChIP-seq, other types of -seq, and
mammalian resequencing.  You may observe longer running times in other
research contexts.

If you find Bowtie's performance to be disappointingly slow, please try
some of the performance-tuning hints described in the [High Performance
Tips] section below.

Alignments involving one or more ambiguous reference characters (`N`,
`-`, `R`, `Y`, etc.) are considered invalid by Bowtie.  This is true
only for ambiguous characters in the reference; alignments involving
ambiguous characters in the read are legal, subject to the alignment
policy.  Also, alignments that "fall off" the reference sequence are
not considered legal by Bowtie.

The process by which `bowtie` chooses an alignment to report is
randomized in order to avoid "mapping bias" - the phenomenon whereby
an aligner systematically fails to report a particular class of good
alignments, causing spurious "holes" in the comparative assembly.
Whenever `bowtie` reports a subset of the valid alignments that exist,
it makes an effort to sample them randomly.  This randomness flows
from a simple seeded pseudo-random number generator and is
deterministic in the sense that Bowtie will always produce the same
results for the same read when run with the same initial "seed" value
(see documentation for `--seed` option).

In the default mode, `bowtie` can exhibit strand bias.  Strand bias
occurs when input reference and reads are such that (a) some reads
align equally well to sites on the forward and reverse strands of the
reference, and (b) the number of such sites on one strand is different
from the number on the other strand.  When this happens for a given
read, `bowtie` effectively chooses one strand or the other with 50%
probability, then reports a randomly-selected alignment for that read
from among the sites on the selected strand.  This tends to overassign
alignments to the sites on the strand with fewer sites and underassign
to sites on the strand with more sites.  The effect is mitigated,
though it may not be eliminated, when reads are longer or when paired-
end reads are used.  Running Bowtie in `--best` mode eliminates strand
bias by forcing Bowtie to select one strand or the other with a
probability that is proportional to the number of best sites on the
strand. 

Gapped alignments are not currently supported, but support is planned
for a future release.

[the -n alignment mode]: [#the--n-alignment-mode]
[the -v alignment mode]: [#the--v-alignment-mode]
[High Performance Tips]: [#high-performance-tips]
[Maq]:                   http://maq.sf.net

The `-n` alignment mode
=======================

When the `-n` option is specified (and it is by default), `bowtie`
determines which alignments are valid according to the following
policy, which is similar to [Maq]'s default policy.

  1. Alignments may have no more than `N` mismatches (where `N` is a
     number 0-3, set with `-n`) in the first `L` bases (where `L` is a
     number 5 or greater, set with `-l`) on the high-quality (left) end
     of the read.  The first `L` bases are called the "seed".

  2. The sum of the [Phred quality] values at all mismatched positions
     may not exceed `E` (set with `-e`).  Where qualities are
     unavailable (e.g. if the reads are from a FASTA file), the Phred
     quality defaults to 40.

The `-n` option is mutually exclusive with the `-v` option.

If there are many possible alignments satisfying these criteria, Bowtie
gives preference to alignments with fewer mismatches and where the sum
from criterion 2 is smaller.  When the `--best` option is specified,
Bowtie guarantees the reported alignment(s) are "best" in terms of
these criteria (criterion 1 has priority), and that the alignments are
reported in best-to-worst order.  Bowtie is somewhat slower when
`--best` is specified.

Note that [Maq] internally rounds base qualities to the nearest 10 and
rounds qualities greater than 30 to 30.  To maintain compatibility,
Bowtie does the same.  Rounding can be suppressed with the
`--nomaqround` option.
 
Bowtie is not fully sensitive in `-n 2` and `-n 3` modes by default.
In these modes Bowtie imposes a "backtracking limit" to limit effort
spent trying to find valid alignments for low-quality reads unlikely to
have any.  This may cause bowtie to miss some legal 2- and 3-mismatch
alignments.  The limit is set to a reasonable default (125 without
`--best`, 800 with `--best`), but the user may decrease or increase the
limit using the `--maxbts` and/or `-y` options.  `-y` mode is
relatively slow but guarantees full sensitivity.

[Maq]:           http://maq.sf.net
[Phred quality]: http://en.wikipedia.org/wiki/FASTQ_format#Variations

The `-v` alignment mode
=======================

In `-v` mode, alignments may have no more than `V` mismatches, where
`V` may be a number from 0 through 3 set using the `-v` option.
Quality values are ignored.  The `-v` option is mutually exclusive with
the `-n` option.

If there are many legal alignments, Bowtie gives preference to
alignments with fewer mismatches.  When the `--best` option is
specified, Bowtie guarantees the reported alignment(s) are "best" in
terms of the number of mismatches, and that the alignments are reported
in best-to-worst order.  Bowtie is somewhat slower when `--best` is
specified.

Reporting Modes
---------------

With the `-k`, `-a`, `-m`, `--best` and `--strata` options, the user
can flexibily select which alignments are reported.  Below we
demonstrate a few ways in which these options can be combined.  All
examples are using the `e_coli` index packaged with Bowtie.  The
--suppress option is used to keep the output concise and some
output is elided for clarity.

### Example 1: `-a`

> $ ./bowtie -a -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A
> -	gi|110640213|ref|NC_008253.1|	4930433	4:G>T,6:C>G
> -	gi|110640213|ref|NC_008253.1|	905664	6:A>G,7:G>T
> +	gi|110640213|ref|NC_008253.1|	1093035	2:T>G,15:A>T

Specifying `-a` instructs bowtie to report *all* valid alignments,
subject to the alignment policy: `-v 2`.  In this case, bowtie finds
5 inexact hits in the E. coli genome; 1 hit (the 2nd one listed)
has 1 mismatch, and the other 4 hits have 2 mismatches.  Four are on
the reverse reference strand and one is on the forward strand.  Note
that they are not listed in best-to-worst order.

### Example 2: `-k 3`

> $ ./bowtie -k 3 -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A
> -	gi|110640213|ref|NC_008253.1|	4930433	4:G>T,6:C>G

Specifying `-k 3` instructs bowtie to report up to 3 valid alignments.
In this case, a total of 5 valid alignments exist (see Example 1);
`bowtie` reports 3 out of those 5.  `-k` can be set to any integer
greater than 0.

### Example 3: `-k 6`

> $ ./bowtie -k 6 -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A
> -	gi|110640213|ref|NC_008253.1|	4930433	4:G>T,6:C>G
> -	gi|110640213|ref|NC_008253.1|	905664	6:A>G,7:G>T
> +	gi|110640213|ref|NC_008253.1|	1093035	2:T>G,15:A>T

Specifying `-k 6` instructs bowtie to report up to 6 valid alignments.
In this case, a total of 5 valid alignments exist, so bowtie reports
all 5.

### Example 4: default (`-k 1`)

> $ ./bowtie -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G

Leaving the reporting options at their defaults causes `bowtie` to
report the first valid alignment it encounters.  Because `--best` was
not specified, we are not guaranteed that bowtie will report the best
alignment, and in this case it does not (the 1-mismatch alignment from
the previous example would have been better).  The default reporting
mode is equivalent to `-k 1`.

### Example 5: `-a --best`

> $ ./bowtie -a --best -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A
> +	gi|110640213|ref|NC_008253.1|	1093035	2:T>G,15:A>T
> -	gi|110640213|ref|NC_008253.1|	905664	6:A>G,7:G>T
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G
> -	gi|110640213|ref|NC_008253.1|	4930433	4:G>T,6:C>G

Specifying `-a --best` results in the same alignments being printed as
if just `-a` had been specified, but they are guaranteed to be reported
in best-to-worst order.

### Example 6: `-a --best --strata`

> $ ./bowtie -a --best --strata -v 2 --suppress 1,5,6,7 e_coli -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A

Specifying `--strata` in addition to `-a` and `--best` causes `bowtie`
to report only those alignments in the best alignment "stratum".  The
alignments in the best stratum are those having the least number of
mismatches (or mismatches just in the "seed" portion of the alignment
in the case of `-n` mode).  Note that if `--strata` is specified,
`--best` must also be specified.

### Example 7: `-a -m 3`

> $ ./bowtie -a -m 3 -v 2 e_coli -c ATGCATCATGCGCCAT
> No alignments

Specifying `-m 3` instructs bowtie to refrain from reporting any
alignments for reads having more than 3 reportable alignments.  The
`-m` option is useful when the user would like to guarantee that
reported alignments are "unique", for some definition of unique.

Example 1 showed that the read has 5 reportable alignments when `-a`
and `-v 2` are specified, so the `-m 3` limit causes bowtie to output
no alignments.

### Example 8: `-a -m 5`

> $ ./bowtie -a -m 5 -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	148810	10:A>G,13:C>G
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A
> -	gi|110640213|ref|NC_008253.1|	4930433	4:G>T,6:C>G
> -	gi|110640213|ref|NC_008253.1|	905664	6:A>G,7:G>T
> +	gi|110640213|ref|NC_008253.1|	1093035	2:T>G,15:A>T

Specifying `-m 5` instructs bowtie to refrain from reporting any
alignments for reads having more than 5 reportable alignments.  Since
the read has exactly 5 reportable alignments, the `-m 5` limit allows
`bowtie` to print them as usual. 

### Example 9: `-a -m 3 --best --strata`

> $ ./bowtie -a -m 3 --best --strata -v 2 e_coli --suppress 1,5,6,7 -c ATGCATCATGCGCCAT
> -	gi|110640213|ref|NC_008253.1|	2852852	8:T>A

Specifying `-m 3` instructs bowtie to refrain from reporting any
alignments for reads having more than 3 reportable alignments.  As we
saw in Example 6, the read has only 1 reportable alignment when `-a`,
`--best` and `--strata` are specified, so the `-m` 3 limit allows
`bowtie` to print that alignment as usual.

Intuitively, the `-m` option, when combined with the `--best` and
`--strata` options, guarantees a principled, though weaker form of
"uniqueness."  A stronger form of uniqueness is enforced when `-m` is
specified but `--best` and `--strata` are not.

Paired-end Alignment
====================

`bowtie` can align paired-end reads when properly paired read files are
specified using the `-1` and `-2` options (for pairs of raw, FASTA, or
FASTQ read files), or using the `--12` option (for Tab-delimited read
files).  A valid paired-end alignment satisfies these criteria:

    1. Both mates have a valid alignment according to the alignment
       policy defined by the `-v`/`-n`/`-e`/`-l` options.
    2. The relative orientation and position of the mates satisfy the
       constraints defined by the `-I`/`-X`/`--fr`/`--rf`/`--ff`
       options. 

Policies governing which paired-end alignments are reported for a
given read are specified using the `-k`, `-a` and `-m` options as
usual.  The `--strata` and `--best` options do not apply in paired-end
mode.

A paired-end alignment is reported as a pair of mate alignments, both
on a separate line, where the alignment for each mate is formatted the
same as an unpaired (singleton) alignment.  The alignment for the mate
that occurs closest to the beginning of the reference sequence (the
"upstream" mate) is always printed before the alignment for the
downstream mate.  Reads files containing paired-end reads will
sometimes name the reads according to whether they are the #1 or #2
mates by appending a `/1` or `/2` suffix to the read name.  If no such
suffix is present in Bowtie's input, the suffix will be added when
Bowtie prints read names in alignments (except in `-S` "SAM" mode,
where mate information is encoded in the FLAGS field instead).

Finding a valid paired-end alignment where both mates align to
repetitive regions of the reference can be very time-consuming.  By
default, Bowtie avoids much of this cost by imposing a limit on the
number of "tries" it makes to match an alignment for one mate with a
nearby alignment for the other.  The default limit is 100.  This causes
`bowtie` to miss some valid paired-end alignments where both mates lie
in repetitive regions, but the user may use the `--pairtries` or `-y`
options to increase Bowtie's sensitivity as desired.

Paired-end alignments where one mate's alignment is entirely contained
within the other's are considered invalid.

Because Bowtie uses an in-memory representation of the original
reference string when finding paired-end alignments, its memory
footprint is larger when aligning paired-end reads.  For example, the
human index has a memory footprint of about 2.2 GB in single-end mode
and 2.9 GB in paired-end mode.

Performance Tuning
==================

    1. Use 64-bit bowtie if possible

    The 64-bit version of Bowtie is substantially faster (usually more
    than 50% faster) than the 32-bit version, due to Bowtie's use of
    64-bit arithmetic when searching both in the index and in the
    reference.  If possible, download the 64-bit binaries for Bowtie
    and on a 64-bit machine.  If you are building Bowtie from sources,
    you may need to pass the `-m64` option to `g++` to compile the
    64-bit version; you can do this by including `BITS=64` in the
    arguments  to the `make` command; e.g.: `make BITS=64 bowtie`.  To
    determine whether your version of bowtie is 64-bit or 32-bit, run
    `bowtie --version`.
  
    2. If your computer has multiple processors/cores, use `-p`
   
    The `-p <int>` option causes Bowtie to launch <int> parallel search
    threads.  Each thread runs on a different processor/core and all
    threads find alignments in parallel, increasing alignment
    throughput by approximately a multiple of `<int>`.
  
    3. If reporting many alignments per read, try tweaking
       `bowtie-build --offrate`
   
    If you are using the `-k`, `-a` or `-m` options and Bowtie is
    reporting many alignments per read (an average of more than about
    10 per read) and you have some memory to spare, using an index with
    a denser SA sample can speed things up considerably.

    To do this, specify a smaller-than-default `--offrate` value when
    running 'bowtie-build'.  A denser SA sample yields a larger index,
    but is also particularly effective at speeding up alignment when
    many alignments are reported per read.  For example, decreasing the
    index's --offrate by 1 could as much as double alignment
    performance, and decreasing by 2 could quadruple alignment
    performance, etc.
  
    On the other hand, decreasing `--offrate` increases the size of the
    Bowtie index, both on disk and in memory when aligning reads.  At
    the default `--offrate` of 5, the SA sample for the human genome
    occupies about 375 MB of memory when aligning reads.  Decreasing
    the `--offrate` by 1 doubles the memory taken by the SA sample, and
    decreasing by 2 quadruples the memory taken, etc.
  
    4. If bowtie "thrashes", try increasing 'bowtie --offrate'
  
    If `bowtie` is beign run on a relatively low-memory machine is very
    slow and consistently triggers more than a few page faults per
    second (as observed via `top` or `vmstat` on Mac/Linux, or via a
    tool like `Process Explorer` on Windows), then try setting
    `--offrate <int>` to a *larger* `<int>` value than the value used
    when building the index.  For example, bowtie-build's default
    `--offrate` is 5 and all pre-built indexes available from the
    Bowtie website are built with `--offrate 5`; so if bowtie thrashes
    when querying such an index, try using `bowtie --offrate 6`.  If
    bowtie still thrashes, try `bowtie --offrate 7`, etc.  A higher
    `--offrate` causes bowtie to use a sparser sample of the suffix-
    array than is stored in the index; this saves memory but makes
    alignment reporting slower (which is especially slow when using
    `-a` or large `-k` or `-m`).

Command Line
============

The following options control `bowtie`'s behavior:

    Usage: bowtie [options]* <ebwt> {-1 <m1> -2 <m2> | --12 <r> | <s>} [<hit>]

  <ebwt>             The basename of the index to be searched.  The
                     basename is the name of any of the index files up
                     to but not including the final .1.ebwt /
                     .rev.1.ebwt / etc.  bowtie looks for the specified
                     index first in the current directory, then in the
                     'indexes' subdirectory under the directory where
                     the currently-running 'bowtie' executable is
                     located, then looks in the directory specified in
                     the BOWTIE_INDEXES environment variable.

  <m1>               Comma-separated list of files containing the #1
                     mates (filename usually includes "_1"), or, if -c
                     is specified, the mate sequences themselves.
                     E.g., this might be "flyA_1.fq,flyB_1.fq", or, if
                     -c is given, this might be "GGTCATCCT,ACGGGTCGT".
                     Sequences specified with this option must
                     correspond file-for-file and read-for-read with
                     those specified in <m2>.  Reads may be a mix of
                     different lengths.  If "-" is specified, Bowtie
                     will read the #1 mates from stdin.

  <m2>               Comma-separated list of files containing the #2
                     mates (filename usually includes "_2"), or, if -c
                     is specified, the mate sequences themselves.
                     E.g., this might be "flyA_2.fq,flyB_2.fq", or, if
                     -c is given, this might be "GTATGCTG,AATTCAGGCTG".
                     Sequences specified with this option must
                     correspond file-for-file and read-for-read with
                     those specified in <m1>.  Reads may be a mix of
                     different lengths.  If "-" is specified, Bowtie
                     will read the #2 mates from stdin.

  <r>                Comma-separated list of files containing a mix of
                     unpaired and paired-end reads in Tab-delimited
                     format.  Tab-delimited format is a 1-read-per-line
                     format where unpaired reads consist of a read
                     name, sequence and quality string each separated
                     by tabs.  A paired-end read consists of a read
                     name, sequnce of the /1 mate, quality values of
                     the /1 mate, sequence of the /2 mate, and quality
                     values of the /2 mate separated by tabs.  Quality
                     values can be expressed using any of the scales
                     supported in FASTQ files.  Reads may be a mix of
                     different lengths and paired-end and unpaired
                     reads may be intermingled in the same file.  If
                     "-" is specified, Bowtie will read the Tab-
                     delimited reads from stdin.

  <s>                A comma-separated list of files containing
                     unpaired reads to be aligned, or, if -c is
                     specified, the unpaired read sequences themselves.
                     E.g., this might be
                     "lane1.fq,lane2.fq,lane3.fq,lane4.fq", or, if -c
                     is specified, this might be "GGTCATCCT,ACGGGTCGT".
                     Reads may be a mix of different lengths.  If "-"
                     is specified, Bowtie gets the reads from stdin.

  <hit>              File to write alignments to.  By default,
                     alignments are written to stdout (the console),
                     but a <hits> file must be specified if the
                     -b/--binout option is also specified.

 Options:
 ========

   Input:
   ------

  -q                 The query input files (specified either as <m1>
                     and <m2>, or as <s>) are FASTQ files (usually
                     having extension .fq or .fastq).  This is the
                     default.  See also: --solexa-quals and
                     --integer-quals.

  -f                 The query input files (specified either as <m1>
                     and <m2>, or as <s>) are FASTA files (usually
                     having extension .fa, .mfa, .fna or similar).  All
                     quality values are assumed to be 40 on the Phred
                     scale.

  -r                 The query input files (specified either as <m1>
                     and <m2>, or as <s>) are Raw files: one sequence
                     per line, without quality values or names.  All
                     quality values are assumed to be 40 on the Phred
                     scale.

  -c                 The query sequences are given on command line.
                     I.e. <m1>, <m2> and <singles> are comma-separated
                     lists of reads rather than lists of read files.

  -s/--skip <int>    Skip (i.e. do not align) the first <int> reads or
                     pairs in the input.

  -u/--qupto <int>   Only align the first <int> reads or read pairs
                     from the input (after the -s/--skip reads or pairs
                     have been skipped).  Default: no limit.

  -5/--trim5 <int>   Trim <int> bases from high-quality (left) end of
                     each read before alignment (default: 0).

  -3/--trim3 <int>   Trim <int> bases from low-quality (right) end of
                     each read before alignment (default: 0).

  --phred33-quals    Input qualities are ASCII chars equal to the Phred
                     quality plus 33.  Default: on.

  --phred64-quals    Input qualities are ASCII chars equal to the Phred
                     quality plus 64.  Default: off.

  --solexa-quals     Convert input qualities from solexa-scaled (which
                     can be negative) to phred-scaled (which can't).
                     The formula for conversion is phred-qual =
                     10 * log(1 + 10 ** (solexa-qual/10.0)) / log(10).
                     This is usually the right option for use with
                     (unconverted) reads emitted by GA Pipeline
                     versions prior to 1.3.  Default: off.

  --solexa1.3-quals  Same as --phred64-quals.  This is usually the
                     right option for use with (unconverted) reads
                     emitted by GA Pipeline version 1.3 or later. 
                     Default: off.

  --integer-quals    Quality values are represented in the read input
                     file as space-separated ASCII integers, e.g.,
                     "40 40 30 40...", rather than ASCII characters,
                     e.g., "II?I...".  Integers are treated as being on
                     the Phred scale unless --solexa-quals is also
                     specified.  Default: off.

   Alignment:
   ----------

  -n/--seedmms <int> The maximum number of mismatches permitted in the
                     "seed", which is the first 28 base pairs of the
                     read by default (see -l/--seedlen).  This may be
                     0, 1, 2 or 3 and the default is 2.  This option is
                     mutually exclusive with the -v option.
 
  -e/--maqerr <int>  The maximum permitted total of quality values at
                     mismatched read positions.  This total is also
                     called the "quality-weighted hamming distance" or
                     "Q-distance."  This is analogous to the -e option
                     for "maq map".  The default is 70.  Note that,
                     like Maq, Bowtie rounds quality values to the
                     nearest 10 and saturates at 30.
  
  -l/--seedlen <int> The "seed length"; i.e., the number of bases on
                     the high-quality end of the read to which the -n
                     ceiling applies.  The lowest permitted value for
                     this parameter is 5.  The default is 28.
                     Depending on the length of the reference genome,
                     Bowtie can become extremely slow as this parameter
                     is adjusted downward.

  --nomaqround       Maq accepts quality values in the Phred scale, but
                     internally rounds quality values to the nearest 10
                     saturating at 30.  By default, Bowtie imitates
                     this behavior.  Use --nomaqround to prevent this
                     type of rounding in Bowtie.

  -v <int>           Forego the Maq-like alignment policy and use a
                     SOAP-like alignment policy.  I.e., report end-to-
                     end alignments with at most <int> mismatches.  If
                     -v is specified, base quality values and the -e
                     and -l options are ignored.  -v is mutually
                     exclusive with -n.

  -I/--minins <int>  The minimum insert size for valid paired-end
                     alignments.  E.g. if -I 60 is specified and a
                     paired-end alignment consists of two 20-bp
                     alignments in the appropriate orientation with a
                     20-bp gap between them, that alignment is
                     considered valid (as long as -X is also
                     satisfied).  A 19-bp gap would not be valid in
                     that case.  If trimming options -3 or -5 are also
                     used, the -I constraint is applied with respect to
                     the untrimmed mates, not the trimmed mates.
                     Default: 0.

  -X/--maxins <int>  The maximum insert size for valid paired-end
                     alignments.  E.g. if -X 100 is specified and a
                     paired-end alignment consists of two 20-bp
                     alignments in the proper orientation with a 60-bp
                     gap between them, that alignment is considered
                     valid (as long as -I is also satisfied).  A 61-bp
                     gap would not be valid in that case.  If trimming
                     options -3 or -5 are also used, the -X constraint
                     is applied with respect to the untrimmed mates,
                     not the trimmed mates.  Default: 250.

  --fr/--rf/--ff     The upstream/downstream mate orientations for a
                     valid paired-end alignment against the forward
                     reference strand.  E.g., if --fr is specified and
                     there is a candidate paired-end alignment where
                     mate1 appears upstream of the reverse complement
                     of mate2 and the insert length constraints are
                     met, that alignment is valid.  Also, if mate2
                     appears upstream of the reverse complement of
                     mate1 and all other constraints are met, that too
                     is valid.  --rf likewise requires that an upstream
                     mate1 be reverse-complemented and a downstream
                     mate2 be forward-oriented.  --ff requires both an
                     upstream mate1 and a downstream mate2 to be
                     forward-oriented.  Default: --fr (appropriate for
                     the Illumina short insert library).

  --nofw/--norc      If --nofw is specified, Bowtie will not attempt to
                     align against the forward reference strand.  If
                     --norc is specified, Bowtie will not attempt to
                     align against the reverse-complement reference
                     strand.  For paired-end reads using --fr or --rf
                     modes, --nofw and --norc apply to the forward and
                     reverse-complement pair orientations.  I.e.
                     specifying --nofw and --fr will only find reads in
                     the R/F orientation where mate 2 occurs upstream
                     of mate 1 with respect to the forward reference
                     strand.

  --maxbts           The maximum number of backtracks permitted when
                     aligning a read in -n 2 or -n 3 mode (default:
                     125 without --best, 800 with --best).  A
                     "backtrack" is the introduction of a speculative
                     substitution into the alignment.  Without this
                     limit, the default parameters will sometimes
                     require that 'bowtie' try 100s or 1,000s of
                     backtracks to align a read, especially if the read
                     has many low-quality bases and/or has no valid
                     alignments, slowing bowtie down significantly.
                     However, this limit may cause some valid
                     alignments to be missed.  Higher limits yield
                     greater sensitivity at the expensive of longer
                     running times.  See also: -y/--tryhard.

  --pairtries <int>  For paired-end alignment, this is the maximum
                     number of attempts Bowtie will make to match an
                     alignment for one mate up with an alignment for
                     the opposite mate.  Most paired-end alignments
                     require only a few such attempts, but pairs where
                     both mates occur in highly repetitive regions of
                     the reference can require significantly more.
                     Setting this to a higher number allows Bowtie to
                     find more paired-end alignments for repetitive
                     pairs at the expense of speed.  The default is
                     100.  See also: -y/--tryhard.

  -y/--tryhard       Try as hard as possible to find valid alignments
                     when they exist, including paired-end alignments.
                     This is equivalent to specifying very high values
                     for the --maxbts and --pairtries options.  This
                     mode is generally MUCH SLOWER than the default
                     settings, but can be useful for certain research
                     problems.  This mode is slower when (a) the
                     reference is very repetitive, (b) the reads are
                     low quality, or (c) not many reads have valid
                     alignments.

  --chunkmbs <int>   The number of megabytes of memory a given thread
                     is given to store path descriptors in --best mode.
                     Best-first search must keep track of many paths at
                     once to ensure it is always extending the path
                     with the lowest cumulative cost.  Bowtie tries to
                     minimize the memory impact of the descriptors, but
                     they can still grow very large in some cases.  If
                     you receive an error message saying that chunk
                     memory has been exhausted in --best mode, try
                     adjusting this parameter up to dedicate more
                     memory to the descriptors.  Default: 32.

   Reporting:
   ----------

  -k <int>           Report up to <int> valid alignments per read or
                     pair (default: 1).  Validity of alignments is
                     determined by the alignment policy (combined
                     effects of -n, -v, -l, and -e).  If more than one
                     valid alignment exists and the --best and --strata
                     options are specified, then only those alignments
                     belonging to the best alignment "stratum" (i.e.
                     those with the fewest mismatches) will be
                     reported.  Bowtie is designed to be very fast for
                     small -k but bowtie can become significantly
                     slower as -k increases.  If you would like to use
                     Bowtie for larger values of -k, consider building
                     an index with a denser suffix-array sample, i.e.
                     specify a smaller '--offrate' when invoking
                     'bowtie-build' for the relevant index (see
                     Performance Tips section for details).

  -a/--all           Report all valid alignments per read or pair
                     (default: off).  Validity of alignments is
                     determined by the alignment policy (combined
                     effects of -n, -v, -l, and -e).  If more than one
                     valid alignment exists and the --best and --strata
                     options are specified, then only those alignments
                     belonging to the best alignment "stratum" (i.e.
                     those with the fewest mismatches) will be
                     reported.  Bowtie is designed to be very
                     fast for small -k but bowtie can become
                     significantly slower if -a/--all is specified.  If
                     you would like to use Bowtie with -a, consider
                     building an index with a denser suffix-array
                     sample, i.e. specify a smaller '--offrate' when
                     invoking 'bowtie-build' for the relevant index
                     (see Performance Tips section for details).

  -m <int>           Suppress all alignments for a particular read or
                     pair if more than <int> reportable alignments
                     exist for it.  Reportable alignments are those
                     that would be reported given the -n, -v, -l, -e,
                     -k, -a, --best, and --strata options.  Default:
                     no limit.  Bowtie is designed to be very fast for
                     small -m but bowtie can become significantly
                     slower for larger values of -m.    If you would
                     like to use Bowtie for larger values of -k,
                     consider building an index with a denser suffix-
                     array sample, i.e. specify a smaller '--offrate'
                     when invoking 'bowtie-build' for the relevant
                     index (see Performance Tips section for details).

  --best             Make Bowtie guarantee that reported singleton
                     alignments are "best" in terms of stratum (i.e.
                     number of mismatches, or mismatches in the seed in
                     the case of -n mode) and in terms of the quality
                     values at the mismatched position(s).  Stratum
                     always trumps quality; e.g. a 1-mismatch alignment
                     where the mismatched position has Phred quality 40
                     is preferred over a 2-mismatch alignment where the
                     mismatched positions both have Phred quality 10.
                     When --best is not specified, Bowtie may report
                     alignments that are sub-optimal in terms of
                     stratum and/or quality (though an effort is made
                     to report the best alignment).  --best mode also
                     removes all strand bias.  Note that --best does
                     not affect which alignments are considered "valid"
                     by Bowtie, only which valid alignments are
                     reported by Bowtie.  When --best is specified and
                     multiple hits are allowed (via -k or -a), the
                     alignments for a given read are guaranteed to
                     appear in best-to-worst order in Bowtie's output.
                     Bowtie is about 1-2.5 times slower when --best is
                     specified.

  --strata           If many valid alignments exist and are reportable
                     (e.g. are not disallowed via the -k option) and
                     they fall into more than one alignment "stratum",
                     report only those alignments that fall into the
                     best stratum.  By default, Bowtie reports all
                     reportable alignments regardless of whether they
                     fall into multiple strata.  When --strata is
                     specified, --best must also be specified. 

   Output:
   -------

  -S/--sam           Print alignments in SAM format.  See the SAM
                     format spec (http://samtools.sf.net/) and the "SAM
                     output" section of the manual for details.  To
                     suppress all SAM headers, use --sam-nohead.  To
                     suppress just the @SQ headers (e.g. if the
                     alignment is against a very large number of
                     reference sequences), use --sam-nosq.  Bowtie
                     does not write BAM files directly, but SAM output
                     can be converted to BAM on the fly by piping
                     Bowtie's output to 'samtools view'.  -S/--sam is
                     not compatible with --refout.

  --concise          Print alignments in a concise format. Each line
                     has format 'read_idx{-|+}:<ref_idx,ref_off,mms>',
                     where read_idx is the index of the read mapped,
                     {-|+} is the orientation of the read, ref_idx is
                     the index of the reference sequence aligned to,
                     ref_off is the offset into the reference sequence,
                     and mms is the number of mismatches in the
                     alignment.  Each alignment appears on a separate
                     line.

  -t/--time          Print the amount of wall-clock time taken by each
                     phase.

  -B/--offbase <int> When outputting alignments, number the first base
                     of a reference sequence as <int>.  Default: 0.
                     (Default is likely to change to 1 in Bowtie 1.0.)

  --quiet            Print nothing besides alignments.

  --refout           Write alignments to a set of files named
                     refXXXXX.map, where XXXXX is the 0-padded index of
                     the reference sequence aligned to.  This can be a
                     useful way to break up work for downstream
                     analyses when dealing with, for example, large
                     numbers of reads aligned to the assembled human
                     genome.  If <hits> is also specified, it will be
                     ignored.

  --refidx           When a reference sequence is referred to in a
                     reported alignment, refer to it by 0-based index
                     (its offset into the list of references that were
                     indexed) rather than by name.

  --al <filename>    Write all reads for which at least one alignment
                     was reported to a file with name <filename>.
                     Written reads will appear as they did in the
                     input, without any of the trimming or translation
                     of quality values that may have taken place within
                     Bowtie.  Paired-end reads will be written to two
                     parallel files with "_1" and "_2" inserted in the
                     filename, e.g., if <filename> is aligned.fq, the
                     #1 and #2 mates that fail to align will be written
                     to aligned_1.fq and aligned_2.fq respectively.

  --un <filename>    Write all reads that could not be aligned to a
                     file with name <filename>.  Written reads will
                     appear as they did in the input, without any of
                     the trimming or translation of quality values that
                     may have taken place within Bowtie.  Paired-end
                     reads will be written to two parallel files with
                     "_1" and "_2" inserted in the filename, e.g., if
                     <filename> is unaligned.fq, the #1 and #2 mates
                     that fail to align will be written to
                     unaligned_1.fq and unaligned_2.fq respectively.
                     Unless --max is also specified, reads with a
                     number of valid alignments exceeding the limit set
                     with the -m option are also written to <filename>.

  --max <filename>   Write all reads with a number of valid alignments
                     exceeding the limit set with the -m option to a
                     file with name <filename>.  Written reads will
                     appear as they did in the input, without any of
                     the trimming or translation of quality values that
                     may have taken place within Bowtie.  Paired-end
                     reads will be written to two parallel files with
                     "_1" and "_2" inserted in the filename, e.g., if
                     <filename> is max.fq, the #1 and #2 mates
                     that fail to align will be written to
                     max_1.fq and max_2.fq respectively.  These reads
                     are not written to the file specified with --un.

  --sam-nohead       Suppress header lines (starting with @) when
                     output is SAM.

  --sam-nosq         Suppress @SQ header lines when output is SAM.

  --fullref          Print the full refernce sequence name, including
                     whitespace, in alignment output.  By default
                     bowtie prints everything up to but not including
                     the first whitespace.
   Performance:
   ------------

  -p/--threads <int> Launch <int> parallel search threads (default: 1).
                     Threads will run on separate processors/cores and
                     synchronize when grabbing reads and outputting
                     alignments.  Searching for alignments is highly
                     parallel, and speedup is fairly close to linear.
                     This option is only available if bowtie is linked
                     with the pthreads library (i.e. if
                     BOWTIE_PTHREADS=0 is not specified at build time).

  -o/--offrate <int> Override the offrate of the index with <int>.  If
                     <int> is greater than the offrate used to build
                     the index, then some row markings are discarded
                     when the index is read into memory.  This reduces
                     the memory footprint of the aligner but requires
                     more time to calculate text offsets.  <int> must
                     be greater than the value used to build the index.

  --mm               Use memory-mapped I/O to load the index, rather
                     than normal POSIX/C file I/O.  Memory-mapping the
                     index allows many concurrent bowtie processes on
                     the same computer to share the same memory image
                     of the index (i.e. you pay the memory overhead
                     just once).  This facilitates memory-efficient
                     parallelization of Bowtie in situations where
                     using -p is not desirable.

  --shmem            Use shared memory to load the index, rather than
                     normal POSIX/C file I/O.  Using shared memory
                     allows many concurrent bowtie processes on
                     the same computer to share the same memory image
                     of the index (i.e. you pay the memory overhead
                     just once).  This facilitates memory-efficient
                     parallelization of Bowtie in situations where
                     using -p is not desirable.  Unlike --mm, --shmem
                     installs the index into shared memory permanently,
                     or until the user deletes the shared memory chunks
                     manually.  See your operating system documentation
                     for details on how to manually list and remove
                     shared memory chunks (on Linux and Mac OS X, these
                     commands are ipcs and ipcrm).  You may also need
                     to increase your OS's maximum shared-memory chunk
                     size to accomodate larger indexes; see your OS
                     documentation.

   Other:
   ------

  --seed <int>       Use <int> as the seed for pseudo-random number
                     generator.

  --verbose          Print verbose output (for debugging).

  --version          Print version information and quit.

  -h/--help          Print detailed description of tool and its options
                     (from MANUAL).

Default output
==============

`bowtie` outputs one alignment per line.  Each line is a collection of
8 fields separated by tabs; from left to right, the fields are:

    1. Name of read that aligned

    2. Reference strand aligned to, `+` for forward strand, `-` for
       reverse

    3. Name of reference sequence where alignment occurs, or numeric ID
       if no name was provided

    4. 0-based offset into the forward reference strand where leftmost
       character of the alignment occurs

    5. Read sequence (reverse-complemented if orientation is `-`)

    6. ASCII-encoded read qualities (reversed if orientation is `-`).
       The encoded quality values are on the Phred scale and the
       encoding is ASCII-offset by 33 (ASCII char `!`). 

    7. Number of other instances where the same read aligns against the
       same reference characters as were aligned against in this
       alignment.  This is *not* the number of other places the read
       aligns with the same number of mismatches.  The number in this
       column is generally not a good proxy for that number (e.g., the
       number in this column may be '0' while the number of other
       alignments with the same number of mismatches might be large).
       This column was previously described as "Reserved".

    8. Comma-separated list of mismatch descriptors.  If there are no
       mismatches in the alignment, this field is empty.  A single
       descriptor has the format offset:reference-base>read-base.  The
       offset is expressed as a 0-based offset from the high-quality
       (5') end of the read. 

SAM output
==========

Following is a brief description of the SAM format as output by
`bowtie` when the `-S`/`--sam` option is specified.  For more details,
see the [SAM format specification].

When `-S`/`--sam` is specified, `bowtie` prints a SAM header with
`@HD`, `@SQ` and `@PG` lines.  When one or more `--sam-RG` arguments
are specified, `bowtie` will also print an appropriately-formatted
`@RG` line.

Each subsequnt line corresponds to a read or an alignment.  Each line
is a collection of at least 12 fields separated by tabs; from left to
right, the fields are:

    1. Name of read that aligned

    2. Sum of all applicable flags.  Flags relevant to Bowtie are:

        1. The read is one of a pair
        2. The alignment is one end of a proper paired-end alignment
        4. The read has no reported alignments
        8. The read is one of a pair and has no reported alignments
       16. The alignment is to the reverse reference strand
       32. The other mate in the paired-end alignment is aligned to the
           reverse reference strand
       64. The read is the first (#1) mate in a pair
      128. The read is the second (#2) mate in a pair

      Thus, an unpaired read that aligns to the reverse reference
      strand will have flag 16.  A paired-end read that aligns and is
      the first mate in the pair will have flag 83 (= 64 + 16 + 2 + 1).

   3. Name of reference sequence where alignment occurs, or ordinal ID
      if no name was provided

   4. 1-based offset into the forward reference strand where leftmost
      character of the alignment occurs

   5. Mapping quality (always 255)

   6. CIGAR string representation of alignment

   7. Name of reference sequence where mate's alignment occurs.  Set to
      `=` if the mate's reference sequence is the same as this
      alignment's, or `*` if there is no mate.

   8. 1-based offset into the forward reference strand where leftmost
      character of the mate's alignment occurs.  Offset is 0 if there
      is no mate.

   9. Inferred insert size.  Size is negative if the mate's alignment
      occurs upstream of this alignment.  Size is 0 if there is no
      mate.

  10. Read sequence (reverse-complemented if aligned to the reverse
      strand)

  11. ASCII-encoded read qualities (reversed if aligned to the reverse
      strand).  The encoded quality values are on the Phred scale and
      the encoding is ASCII-offset by 33 (ASCII char `!`). 

 12+. Optional fields.  Fields are tab-separated.  For descriptions of
      all possible optional fields, see the SAM format specification.
      Bowtie outputs one or more of these optional fields for each
      alignment, depending on the type of the alignment:

      NM:i:<N> : Aligned read has an edit distance of <N>
      MD:Z:<S> : For aligned reads, <S> is a string representation of
                 the mismatched reference bases in the alignment.  See
                 SAM format specification for details.
      XA:i:<N> : Aligned read belongs to stratum <N>
      XM:i:<N> : For a read with no reported alignments, <N> is 0 if
                 the read had no alignments, or 1 if the read had
                 alignments that were suppressed by the -m option.

[SAM format specification]: http://samtools.sf.net/SAM1.pdf

 Using the 'bowtie-build' Indexer
 --------------------------------

 Use 'bowtie-build' to build a Bowtie index from a set of DNA
 sequences.  bowtie-build outputs a set of 6 files with suffixes
 .1.ebwt, .2.ebwt, .3.ebwt, .4.ebwt, .rev.1.ebwt, and .rev.2.ebwt,
 where the prefix is the <ebwt_outfile_base> parameter supplied by the
 user on the command line.  These files together constitute the index:
 they are all that is needed to align reads to the reference sequences.
 The original sequence files are no longer used by Bowtie once the
 index is built.  

 Use of Karkkainen's blockwise algorithm (see reference #4 below)
 allows bowtie-build to trade off between running time and memory
 usage. bowtie-build has three options governing how it makes this
 trade: -p/--packed, --bmax/--bmaxdivn, and --dcv.  By default, bowtie-
 build will automatically search for the settings that yield the best
 running time without exhausting memory.  This behavior can be disabled
 using the -a/--noauto option.

 The indexer provides options pertaining to the "shape" of the index,
 e.g. --offrate governs the fraction of Burrows-Wheeler rows that are
 "marked" (i.e., the "density" of the suffix-array sample; see
 reference #2).  All of these options are potentially profitable trade-
 offs depending on the application.  They have been set to defaults
 that are reasonable for most cases according to our experiments.  See
 "High Performance Tips" in the "Using the 'bowtie' Aligner" section
 for additional details.

 Because bowtie-build uses 32-bit pointers internally, it can handle up
 to a maximum of 2^32-1 (somewhat more than 4 billion) characters in an
 index.  If your reference exceeds 2^32-1 characters, bowtie-build will
 print an error message and abort.  To resolve this, divide your
 reference sequences into smaller batches and/or chunks and build a
 separate index for each.
 
 If your computer has more than 3-4 GB of memory and you would like to
 exploit that fact to make index building faster, you must use a 64-bit
 version of the bowtie-build binary.  The 32-bit version of the binary
 is restricted to using less than 4 GB of memory.  If a 64-bit pre-
 built binary does not yet exist for your platform on the sourceforge
 download site, you will need to build one from source.

 The Bowtie index is based on the FM Index of Ferragina and Manzini,
 which in turn is based on the Burrows-Wheeler transform.  The
 algorithm used to build the index is based on the blockwise algorithm
 of Karkkainen.  For more information on these techniques, see these
 references:

 1. Burrows M, Wheeler DJ: A block sorting lossless data compression
    algorithm. Digital Equipment Corporation, Palo Alto, CA 1994,
    Technical Report 124.
 2. Ferragina, P. and Manzini, G. 2000. Opportunistic data structures
    with applications. In Proceedings of the 41st Annual Symposium on
    Foundations of Computer Science (November 12 - 14, 2000). FOCS
 3. Ferragina, P. and Manzini, G. 2001. An experimental study of an
    opportunistic index. In Proceedings of the Twelfth Annual ACM-SIAM
    Symposium on Discrete Algorithms (Washington, D.C., United States,
    January 07 - 09, 2001). 269-278.
 4. Karkkainen, J. 2007. Fast BWT in small space by blockwise suffix
    sorting. Theor. Comput. Sci. 387, 3 (Nov. 2007), 249-257

  Command Line
  ------------

 Usage: bowtie-build [options]* <reference_in> <ebwt_outfile_base>

    <reference_in>          A comma-separated list of FASTA files
                            containing the reference sequences to be
                            aligned to, or, if -c is specified, the
                            sequences themselves. E.g., this might be
                            "chr1.fa,chr2.fa,chrX.fa,chrY.fa", or, if
                            -c is specified, this might be
                            "GGTCATCCT,ACGGGTCGT,CCGTTCTATGCGGCTTA".

    <ebwt_outfile_base>     The basename of the index files to write.
                            By default, bowtie-build writes files named
                            NAME.1.ebwt, NAME.2.ebwt, NAME.3.ebwt,
                            NAME.4.ebwt, NAME.rev.1.ebwt, and
                            NAME.rev.2.ebwt, where NAME is the
                            basename.

 Options:

    -f                      The reference input files (specified as
                            <reference_in>) are FASTA files (usually
                            having extension .fa, .mfa, .fna or
                            similar).

    -c                      The reference sequences are given on the
                            command line.  I.e. <reference_in> is a
                            comma-separated list of sequences rather
                            than a list of FASTA files.

    -a/--noauto             Disable the default behavior whereby
                            bowtie-build automatically selects values
                            for --bmax/--dcv/--packed parameters
                            according to the memory available.  User
                            may specify values for those parameters.
                            If memory is exhausted during indexing, an
                            error message will be printed; it is up to
                            the user to try new parameters.

    -p/--packed             Use a packed (2-bits-per-nucleotide)
                            representation for DNA strings.  This saves
                            memory but makes indexing 2-3 times slower.
                            Default: off.  This is configured
                            automatically by default; use -a/--noauto
                            to configure manually.

    --bmax <int>            The maximum number of suffixes allowed in a
                            block.  Allowing more suffixes per block
                            makes indexing faster, but increases memory
                            overhead.  Overrides any previous
                            specification of --bmax, --bmaxmultsqrt or
                            --bmaxdivn.  Default: --bmaxdivn 4.  This
                            is configured automatically by default; use
                            -a/--noauto to configure manually.

    --bmaxdivn <int>        The maximum number of suffixes allowed in a
                            block, expressed as a fraction of the
                            length of the reference.  Overrides any
                            previous specification of --bmax,
                            --bmaxmultsqrt or --bmaxdivn. Default:
                            --bmaxdivn 4.  This is configured
                            automatically by default; use -a/--noauto
                            to configure manually.

    --dcv <int>             Use <int> as the period for the difference-
                            cover sample.  A larger period yields less
                            memory overhead, but may make suffix
                            sorting slower, especially if repeats are
                            present.  Must be a power of 2 no greater
                            than 4096.  Default: 1024.  This is
                            configured automatically by default; use
                            -a/--noauto to configure manually.

    --nodc                  Disable use of the difference-cover sample.
                            Suffix sorting becomes quadratic-time in
                            the worst case (where the worst case is an
                            extremely repetitive reference).  Default:
                            off.

    -r/--noref              Do not build the NAME.3.ebwt and
                            NAME.4.ebwt portions of the index, which
                            contain a bitpacked version of the
                            reference sequences and are (currently)
                            only used for paired-end alignment.

    -3/--justref            Build *only* the NAME.3.ebwt and
                            NAME.4.ebwt portions of the index, which
                            contain a bitpacked version of the
                            reference sequences and are (currently)
                            only used for paired-end alignment.

    -o/--offrate <int>      To map alignments back to positions on the
                            reference sequences, it's necessary to
                            annotate ("mark") some or all of the
                            Burrows-Wheeler rows with their
                            corresponding location on the genome.  The
                            offrate governs how many rows get marked:
                            the indexer will mark every 2^<int> rows.
                            Marking more rows makes reference-position
                            lookups faster, but requires more memory to
                            hold the annotations at runtime.  The
                            default is 5 (every 32nd row is marked; for 
                            human genome, annotations occupy about 340
                            megabytes).  

    -t/--ftabchars <int>    The ftab is the lookup table used to
                            calculate an initial Burrows-Wheeler range
                            with respect to the first <int> characters
                            of the query.  A larger <int> yields a
                            larger lookup table but faster query times.
                            The ftab has size 4^(<int>+1) bytes.  The
                            default is 10 (ftab is 4MB).

    --ntoa                  Convert Ns in the reference sequence to As
                            before building the index.  By default, Ns
                            are simply excluded from the index and
                            'bowtie' will not find alignments that
                            overlap them.

    --big --little          Endianness to use when serializing integers
                            to the index file.  Default: little-endian
                            (recommended for Intel- and AMD-based
                            architectures).

    --seed <int>            Use <int> as the seed for pseudo-random
                            number generator.

    --cutoff <int>          Index only the first <int> bases of the
                            reference sequences (cumulative across
                            sequences) and ignore the rest.

    -q/--quiet              bowtie-build is verbose by default.  With
                            this option ebwt-build will print only
                            error messages.

    -h/--help               Print detailed description of tool and its
                            options (from MANUAL).

    --version               Print version information and quit.

 Using the 'bowtie-inspect' Index Inspector
 ------------------------------------------

 'bowtie-inspect' extracts information from a Bowtie index about the
 original reference sequences used to build it.  By default, the tool
 will output a FASTA file containing the sequences of the original
 references (with all non-A/C/G/T characters converted to Ns).  It can
 also be used to extract just the reference sequence names using the -n
 option.

  Command Line
  ------------

 Usage: bowtie-inspect [options]* <ebwt_base>

  <ebwt_base>        The basename of the index to be inspected.  The
                     basename is the name of any of the four index
                     files up to but not including the first period.
                     bowtie first looks in the current directory for
                     the index files, then looks in the 'indexes'
                     subdirectory under the directory where the
                     currently-running 'bowtie' executable is located,
                     then looks in the directory specified in the
                     BOWTIE_INDEXES environment variable.

 Options:

  -a/--across <int>  When printing FASTA output, output a newline
                     character every <int> bases (default: 60).

  -n/--names         Print reference sequence names only; ignore
                     sequence.

  -v/--verbose       Print verbose output (for debugging).

  --version          Print version information and quit.

  -h/--help          Print detailed description of tool and its options
                     (from MANUAL).