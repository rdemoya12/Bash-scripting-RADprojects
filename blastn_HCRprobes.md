Blast+ RAD
================
Ricardo A DeMoya
12/04/2024

# BLAST+ command line interface

Created by NCBI this command line interface for blast allows you to
access blast programs easily and run local blast searches. Recent
updates added the -remote option which allows you to use NCBI servers to
do you blast searches instead of cloud or local versions! We hope to use
this to hasten our generation of HCR probes.

## Blastn your probes from linux bash script

We have code that creates probes for us and then can output a fasta file
for submission to blast+. This report reveals the work in progress to
achieve this. We then need to parse the blast XML output and determine a
good way to select probes based on this data. The last part being the
most difficult.

First we will add the bash shebang to our file which means the code will
be run as a bash script in most cases. Allowing the program loader to
set the interpreter for the code in the file. Also we will add a
identifier for the person making the code.

``` bash
#!/usr/bin/env bash
# Author: Ricardo A DeMoya, PhD
```

Now when you are writing bash scripts it can be easier to declare number
variable first and update them in the code below. So we will declare
what we need for now. This section will change as more code is written
and more variables are needed downstream.

``` bash
declare -i two=2
declare -i wow=0
declare -i wow2=0
declare -i splt=35
```

Also these variable names will be updated to represent the data they
contain or process they are needed for. Again just good practice, but
while trying to get code to work it will be acceptable for now.

## Getting the fasta file from the user

Next we will get the fasta file name from the user to plug into our
blast+ program.

``` bash
printf "%s" "Enter the fasta file name: "
read fasta_name
```

Now the variable fasta_name contains what the user input after they hit
enter. The fasta file does need to be in the same directory as the bash
script, until code can be written to allow for directory choice.

Since we know we will need information about the gene we are blasting
probes for we capture the genename from the fasta file name below.

``` bash
genename=$(echo ${fasta_name} | cut -d '_' -f 2)
```

## Next we could allow for choice of organism to blast against

This is a great option, but has some limitations because it will time
out the submission if too much CPU space is used at NCBI for more than
an hour. Known as the SIGXCPU (24) error. For our purposes using Danio
rerio has allowed us to get up to 34 probe pairs, so 68 sequences to run
on blastn and return the XML file without hitting this limit.

``` bash
# Read in the users desired organism name
printf "%s" "Enter the scientific name of your organism of choice: "
read orgn_name

orgn_name+=" [organism]"
```

So we will ignore the above code for our uses but can implement later if
cloud databases are incorporated. It is a good idea but expensive,
everything done here cost nothing.

## What if you have more than 34 probes to screen?

Now as is the case with probes for longer genes more than 34 probes may
exist and we need to screen them all to see which 34 we would like to
keep, those most specific to our gene of interest and not other genes
similar in sequence.

To do this we need to keep track of our fasta file line number and we
can then split the file in half if it exceeds 34 probes. This part still
needs more development but will count the lines in the fasta file and
divide it by two.

``` bash
wow=($(wc -l < $fasta_name))
lines=${wow[0]}
wow2=$(bc <<< $wow/$two)
echo "$wow"
echo "$wow2"
```

## How do we get the file split and submit the pieces

This is where the code gets a little more complicated. We will need to
conditionally check if wow2 is larger than we want and then split the
file in two equal halves.

``` bash
if ((wow2 > splt)); then
    # split file in two and run one after another
    split -l $wow2 $fasta_name tmp/tmp
    # Try to find a number to set for range to iterate thru if more than files
    filnum=($(ls "tmp/"))
    for i in $filnum
    do
        name=$i
        blast_out="blast_result_"$genename"_fmt5_"$name".xml"
    
        #echo $blast_out
        echo $i
    done
    #echo $fasta1
    blastn -query "$fasta1" -db nt -remote -task blastn-short -word_size 7 -evalue 10 -perc_identity 95 -entrez_query "Danio rerio [organism]" -outfmt 5 -out "$blast_out" -max_target_seqs 10 -max_hsps 5
    sleep 10s # cannot submit again to NCBI for 10s
    blastn -query "$fasta2" -db nt -remote -task blastn-short -word_size 7 -evalue 10 -perc_identity 95 -entrez_query "${orgn_name}" -outfmt 5 -out blast_result_gadd45ba_fmt5_35.xml -max_target_seqs 10 -max_hsps 5
else echo "It did not work or is not more than 68 lines"

fi
```

## Blastn works well

The blastn process we have been using is above in the for loop but will
need more attention before it is outputting an XML file like we would
like. But below I have place a simple copy of the blastn program and its
parameters. Automating this will save hours of time and provide us a
quick efficient way to make probes and judge them properly.

``` bash
# Using other organism names doesn't always work because of the SIGXCPU memory error
# Danio rerio works well being a third the size of the human genome

# xml output
blastn -query "$fasta_name" -db nt -remote -task blastn-short -word_size 7 -evalue 10 -perc_identity 95 -entrez_query "${orgn_name}" -outfmt 5 -out blast_result_gadd45ba_fmt5_35.xml -max_target_seqs 10 -max_hsps 5

#Tabular output doesn't include <hit_def> needed for downstream analysis
blastn -query B2_gadd45ba_PP17.fasta -db nt -remote -task blastn-short -word_size 7 -evalue 10 -perc_identity 95 -entrez_query "Danio rerio [organism]" -outfmt 5 -out blast_result_gadd45ba_testoutfmt7_eval10.xml -max_target_seqs 10 -max_hsps 5
```

## Next steps

The next steps for this code will include getting the for loop to work
and output two XML files or maybe one combined file. This file would
then need to be parse for useful information from the blast and mapped
back to the input probes sequences. Next we can get the probes filtered
by their specificity to the gene of interest. Finally we can make probes
using this filtering method and test them for similar staining as Molc.
Instru. probes.
