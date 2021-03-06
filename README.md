## Starcode: Sequence clustering based on all-pairs search ##
---
## Contents: ##
    1. What is starcode?
    2. Source file list.
    3. Compilation and installation.
    4. Running starcode.
    5. File formats.
    6. License.
    7. References.

---
## I. What is starcode?   ##


Starcode is a DNA sequence clustering software. Sequence clustering is
performed by finding all pairs below a Levenshtein distance metric [1].
Typically, a file containing a set of related DNA sequences is passed as
input, jointly with a parameter specifying the desired cluster distance.
Starcode aligns and computes the distance between all the sequence pairs
[2] and prints a line for each cluster containing: canonical DNA sequence,
sequence count and the list of sequences that belong to the cluster.

Starcode has many applications in the field of biology, such as DNA/RNA
motif recovery, barcode clustering, sequencing error recovery, etc.


II. Source file list
--------------------

* **main-starcode.c**        Starcode main file (parameter parsing).
* **starcode.c**             Main starcode algorithm.
* **starcode.h**             Main starcode algorithm public header file.
* **starcode-private.h**     Main starcode algorithm private header file.
* **trie.c**                 Trie search and construction functions.
* **trie.h**                 Trie public header file.
* **trie-private.h**         Trie private header file.
* **Makefile**               Make instruction file.


III. Compilation and installation
---------------------------------

To install starcode you first need to clone or manually download the 
repository content from github:

 > git clone git://github.com/gui11aume/starcode.git

the files should be downloaded in a folder named 'starcode'. To compile
just change the directory to 'starcode' and run make (Mac users require
'xcode', available at the Mac Appstore):

 > cd starcode

 > make

a binary file 'starcode' will be created. You can optionally make a
symbolic link to execute starcode from any directory:

 > sudo ln -s ./starcode /usr/bin/starcode


IV. Running starcode
--------------------

Starcode runs on Linux and Mac. It has not been tested on Windows.

List of arguments:

  > starcode [options] {[-i] INPUT_FILE | -1 PAIRED_END_FILE1 -2 PAIRED_END_FILE2} [-o OUTPUT_FILE]
  
  **-d or --distance** *distance*

     Defines the maximum Levenshtein distance for clustering.
     When not set it is automatically computed as:
     min(8, 2 + [median seq length]/30)

  **-t or --threads** *threads*

     Defines the maximum number of parallel threads.
     Default is 1.

  **-s or --spheres**

     When specified, sphere clustering algorithm is performed in the
     clustering phase, instead of the default message passing algorithm.

  **-r or --cluster-ratio** *ratio*

     Specifies the minimum sequence count ratio to cluster two matching
     sequences, i.e. the matching sequences A and B will only be
     clustered together if count(A) > ratio * count(B), assuming that
     count(A) > count(B).
     Note that this option only applies to message passing algorithm and
     ratio must be set to 1 to cluster unique input sequences together.
     Default is 5.


  **-q or --quiet**

     Non verbose. By default, starcode prints verbose information to
     the standard error channel.

  **-h or --help**

     Prints usage information.

  **-o or --output** *file*

     Specifies output file. When not set, standard output is used instead.

  **--non-redundant**
  
     Removes redundant sequences from the output. Only the canonical sequence
     of each cluster is returned.

  **--print-clusters**
  
     Adds a third column to the starcode output, containing the sequences
     associated with each cluster. By default, the output contains only
     the centroid and the counts.

Single-file mode:

  **-i or --input** *file*

     Specifies input file.

Paired-end fastq files:
   
  **-1** *file1* **-2** *file2*

     Specifies two paired-end FASTQ files for paired-end clustering mode.

Standard input is used when neither **-i** nor **-1/-2** are set.

V. File formats
---------------

### V.I. Supported input file formats: ###

####  V.I.I. Plain text: ####

  Consists of a file containing one sequence per line. Only the standard
  DNA-base characters are supported ('A', 'C', 'G', 'T'). The sequences
  may not contain empty spaces at the beginning or the end of the string,
  as these will be counted as alignment characters. The file may not
  contain empty lines as these will be considered as zero-length sequences.
  The sequences do not need to be sorted and may be repeated.
  
  Example:

    TTACTATCGATCATCATCGACTGACTACG
    ACTGCATCGACTAGCTACGACTACGCTACCATCAG
    TTACTATCGATCATCATCGACTGACTAGC
    ACTACGACTACGACTCAGCTCACTATCAGC
    GCATCGACCGCTACTACGCATACTACGACATC


####  V.I.II. Plain text with sequence count: ####

  If the count of the sequences is known, it may be specified in the input
  file using the following format:

  > [SEQUENCE]\t[COUNT]\n

  Where '\t' denotes the TAB character and '\n' the NEWLINE character.
  The sequences do not need to be sorted and may be repeated as well. If
  a repeated sequence is found, their counts will be addded together. As
  before, the sequences may not contain any additional characters and the
  file may not contain empty lines.

  Example:

    TATCGACTCTATCTATCGCTGATGCGTAC       200
    CGAGCCGCCGGCACGTCACGACGCATCAA       1
    TAGCACCTACGCATCTCGACTATCACG         234
    CGAGCCGCCGGCACGTCACGACGCATCAA       17
    TGACTCTATCAGCTAC                    39


####  V.I.III. FASTA/FASTQ ####

  Starcode supports FASTA and FASTQ files as well. Note, however, that
  starcode does not use the quality factors and the only relevant
  information is the sequence itself. The FASTA/FASTQ labels will not
  be used to identify the sequences in the output file. The sequences do
  not need to be sorted and may be repeated.

  Example FASTA:

    > FASTA sequence 1 label
    ATGCATCGATCACTCATCAGCTACAG
    > FASTA sequence 2 label
    TATCGACTATCTACGACTACATCA
    > FASTA sequence 3 label
    ATCATCACTCTAGCAGCGTACTCGCA
    > FASTA sequence 4 label
    ATGCATCGATTACTCATCAGCTACAG

  Example FASTQ:

    @ FASTQ sequence 1 label
    CATCGAGCAGCTATGCAGCTACGAGT
    +
    -$#'%-#.&)%#)"".)--'*()$)%
    @ FASTQ sequence 2 label
    TACTGCTGATATTCAGCTCACACC
    +
    ,*#%+#&*$-#,''+*)'&.,).,


### V.II. Output formats: ###

#### V.II.I Standard output format: ####

  Starcode prints a line for each detected cluster with the following
  format:

  > [CANONICAL SEQUENCE]\t[CLUSTER SIZE]\t[CLUSTER SEQUENCES]\n
  
  Where '\t' denotes the TAB character and '\n' the NEWLINE character.
  'CANONICAL SEQUENCE' is the sequence of the cluster that has more
  counts, 'CLUSTER SIZE' is the aggregated count of all the sequences
  that form the cluster, and 'CLUSTER SEQUENCES' is a list of all the
  cluster sequences separated by commas and in arbitrary order. The
  lines are printed sorted by 'CLUSTER SIZE' in descending order.

  For instance, an execution with the following input and clustering
  distance of 3 (-d3):

    TAGCTAGACGTA   250
    TAGCTAGCCGTA   10
    TAAGCTAGGGGT   16
    ACGCGAGCGGAA   155
    ACTTTAGCGGAA   1

  would produce the following output:

    TAGCTAGACGTA    276       TAGCTAGACGTA,TAGCTAGCCGTA,TAAGCTAGGGGT
    ACGCGAGCGGAA    156       ACGCGAGCGGAA,ACTTTAGCGGAA

  The same example executed with a more restrictive distance -d2 would
  produce the following output:

    TAGCTAGACGTA    260       TAGCTAGACGTA,TAGCTAGCCGTA
    ACGCGAGCGGAA    155       ACGCGAGCGGAA
    TAAGCTAGGGGT    16        TAAGCTAGGGGT
    ACTTTAGCGGAA    1         ACTTTAGCGGAA

#### V.II.II Non-redundant output format: ####

  In non-redundant output mode, starcode only prints the canonical
  sequence of each cluster, one per line. Following the example from
  the previous section, the output with distance 3 (-d3) would be:

      TAGCTAGACGTA
      ACGCGAGCGGAA
    
  whereas for -d2:

      TAGCTAGACGTA
      ACGCGAGCGGAA
      TAAGCTAGGGGT
      ACTTTAGCGGAA


VI. License
-----------

Starcode is licensed under the GNU General Public License, version 3
(GPLv3), for more information read the LICENSE file or refer to:

  http://www.gnu.org/licenses/


VII. References
---------------

[1] Levenshtein, V. (1966), 'Binary Codes Capable of Correcting Deletions,
    Insertions and Reversals', Soviet Physics Doklady 10, 707.

[2] Needleman, S.B. and Wunsch, C.D. (1970), 'A general method applicable
    to the search for similarities in the amino acid sequence of two
    proteins' J. Mol. Biol., 48 (3), 443-53.
