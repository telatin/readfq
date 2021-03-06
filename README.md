# Readfq (by Heng Li)

Forked from https://github.com/lh3/readfq

## Changes to the Perl subroutine (see readfq.pl)
 
 * Removed 'defined @array': deprecated, now fatal error 
 * Added parsing of "comments" in FASTA/FASTQ file names (i.e. string after the first space)

## Original documentation
Readfq is a collection of routines for parsing the FASTA/FASTQ format. It
seamlessly parses both FASTA and multi-line FASTQ with a simple interface.

Readfq is first implemented in a single C header file and then ported to Lua,
Perl and Python as a single function less than 50 lines. For users of scripting
languages, I encourage to copy-and-paste the function instead of using readfq
as a library. It is always good to avoid unnecessary library dependencies.

Readfq also strives for efficiency. The C implementation is among the fastest
(if not the fastest). The Python and Perl implementations are several to tens
of times faster than the official Bio* implementations. If you can speed up
readfq further, please let me know. I am not good at optimizing programs in
scripting languages. Thank you.

As to licensing, the C implementation is distributed under the MIT license.
Implementations in other languages are released without a license. Just copy
and paste. You do not need to acknowledge me. The following shows a brief
example for each programming language:


### Perl
```perl
my @aux = undef; # this is for keeping intermediate data
while (my ($name, $seq, $qual) = readfq(\*STDIN, \@aux)) { print "$seq\n"; }
```


### Python: generator function
```
  for name, seq, qual in readfq(sys.stdin): print seq
```


### Lua: closure
```
  for name, seq, qual in readfq(io.stdin) do print seq end
```

### Go 
```
  package main

  import (
    "fmt"
    "bufio"
    "github.com/drio/drio.go/bio/fasta"
  )

  func main() {
    var fqr fasta.FqReader
    fqr.Reader = bufio.NewReader(os.Stdin)
    for r, done := fqr.Iter(); !done; r, done = fqr.Iter() {
      fmt.Println(r.Seq)
    }
  }
```

### C
```
  #include <zlib.h>
  #include <stdio.h>
  #include "kseq.h"
  KSEQ_INIT(gzFile, gzread)

  int main() {
    gzFile fp;
    kseq_t *seq;
    fp = gzdopen(fileno(stdin), "r");
    seq = kseq_init(fp);
    while (kseq_read(seq) >= 0) puts(seq->seq.s);
    kseq_destroy(seq);
    gzclose(fp);
    return 0;
  }
```


Some naive benchmarks. To convert a FASTQ containing 25 million 100bp reads to FASTA,
FASTX-Toolkit (parsing 4-line FASTQ only) takes 325.0 CPU seconds and EMBOSS' seqret
247.8 seconds. My seqtk, which uses the kseq.h library, finishes the task in 24.6
seconds, 10X faster. For retrieving 25k sequences by name from the same FASTQ,
BioPython takes 963 seconds, while readfq.py takes 136 seconds; BioPerl takes more
than 40 minutes (killed), while readfq.pl 273 seconds. Seqtk takes 29 seconds.

