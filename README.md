# HiCPhylogenyWorkflow
This tutorial is an adaptation to the workflow described in
 **Graham, E. D., J. F. Heidelberg, and B. J. Tully. 'Potential for primary productivity in a globally-distributed bacterial phototroph.' The ISME journal (2018)**
 
The procedure described below was used to build a phylogenetic tree using 16 ribosomal proteins described in [Hug et al. 2016](https://doi.org/10.1038/nmicrobiol.2016.48) of metagenomic-assembled genomes (MAGs) from soil samples produced using ProxiMeta Deconvolution against [RefSoil+](https://doi.org/10.1128/mSystems.00349-18) genomes, a database of 922 soil microorganisms from RefSeq, for phylogenetic context.
In this repository you can run through the procedure using the example [HiC MAGs](HiCPhylogenyWorkflow/HiC_MAG/) fasta files and ribosomal protein calls from the RefSoil genomes present in the [ref_soil folder](HiCPhylogenyWorkflow/ref_soil/) to build a phylogenetic tree.
*If you run this with your own data, this proecedure requires you to have metagenomic-assembled genomes ready for analysis.*

To run through this example you will go through the following steps:

1. Generate gene predictions using prodigal for HiC MAGs

2. Identify ribosomal proteins using an hmm model

3. Align, trim, and concatenate proteins from MAGs and RefSoil genomes for a phylogenetic tree

4. Build a phylogenetic tree


## Prior to Procedure Install Required Dependencies ##

To run this protocol you will need the following programs:

[Python2.7](https://www.python.org/download/releases/2.7/)

[BioPython](http://biopython.org/)

[HMMER](http://hmmer.org/download.html)

[Prodigal](https://github.com/hyattpd/prodigal/wiki/Installation)

[BinSanity](https://github.com/edgraham/BinSanity/wiki/Installation)

[Muscle](https://www.drive5.com/muscle/manual/install.html)

[TrimAL](http://trimal.cgenomics.org/)

[FastTree](http://www.microbesonline.org/fasttree/)

Along with the [identifyHMM](https://github.com/edgraham/PhylogenomicsWorkflow/blob/master/identifyHMM) script which you can retrieve and make executable using the following commands:
```
$ cd ~/bin
$ wget https://raw.githubusercontent.com/edgraham/PhylogenomicsWorkflow/master/identifyHMM
$ chmod +x identifyHMM
```

### Step 1: Download Github Repo ###

After installing all dependencies run the following command to get all the files for the tutorial from this github repo.

```
$ git clone https://github.com/tjhaan/HiCPhylogenyWorkflow.git
$ cd HiCPhylogenyWorkflow/HiC_MAG
$ mv identifyHMM ~/bin && 
```

### Step 2: Run IdentifyHMM script on [HiC MAGs](HiCPhylogenyWorkflow/HiC_MAG/) to extract marker genes ###
The main script used is `identifyHMM` which does the following:

1. Runs prodigal on MAGs
2. Runs `hmmsearch` on these gene calls
3. Parses out genes matching one of the 16 Ribosomal protiens

The script from relies on the user providing the location of a file containing Hidden Markov Models (HMMs) for their genes of interest. In this tutorial we provided a file called `ribosomalmarkers.hmm`. This .hmm file contains HMM models for 16 Ribosomal proteins: RpL14, RpL15, RpL16 ,RpL18, RpL22, RpL24 ,RpL2, RpL3, RpL4, RpL5, RpL6, RpS10, RpS17, RpS19, RpS3, RpS8. 
In the following example we will use the [HiC MAGs](HiCPhylogenyWorkflow/HiC_MAG/) with this the identifyHMM script. If you are running this on your own data and need more information on how to use this script go to the [edgraham/PhylogenomicsWorkflow](https://github.com/edgraham/PhylogenomicsWorkflow) github page.

First change into the  directory containing HiC MAGs (in this case /HiCPhylogenyWorkflow/HiC_MAG) 

```
$ cd HiCPhylogenyWorkflow/HiC_MAG
```

Once in the `HiC_MAG` directory run `identifyHMM` as follows:

```
$ identifyHMM --markerdb ../ribosomalmarkers.hmm --performProdigal --outPrefix HiC .fasta

```

The outputs after running through this example will be 16 `.faa` files appended with the 'HiC' prefix spefied by `--outPrefix`, 10 `.faa` files corresponding to the prodigal gene calls for each HiC MAG, and 10 hmm reports (Stored in the `.tbl` files). Files that look like `HiC_RpL14.faa` are ribosomal proteins pulled from the HiC MAGs of interest.
 
 
 ## Step 3: Combine Reference Proteins from RefSoil genomes with proteins from the HiC MAGs ##
 
 Now that we have extracted marker genes from the HiC MAGs we can merge together the ribosomal protein calls we just made the MAGs with genomes from the RefSoil database in the folder '/HiCPhylogenyWorkflow/ref_soil'.

 To do this we will use a quick bash trick that uses the text file `marker_list.txt` which contains a list of the marker proteins we are searching for. Use the following Bash command:
 
```
 while read p; do cat ../ref_soil/REF_"$p".faa HUG_"$p".faa > Dataset1_"$p".faa; done < ../marker_list.txt
``` 
This bash script is iterating through the list given in `marker_list.txt` and concatenate appropriate reference and experimental protein sets. So for example it would concatenate `ExampleRefSet.RpL6.faa` and `Hug_RpL6.faa` into `Dataset1_RpL6.faa`. 


## Step 4: Align your Proteins ##

Now that we have 16 files containing Ribosomal proteins from our 5 unknown genomes and 100 references we can move on to aligning our proteins.
To align our proteins we can use muscle (You could also alternatively use [MAFFT](https://mafft.cbrc.jp/alignment/software/algorithms/algorithms.html) here):

```
while read p; do muscle -maxiters 16 -in Dataset1_"$p".faa -out Dataset1_"$p".aln; done < ../marker_list.txt
```
We will use the same trick as above where we feed the list of ribosomal markers in `marker_list.txt` into a bash loop thar runs muscle on head of the protein fasta files we generated. This will end in 16 alignment (`.aln`) files.


## Step 5: Trim your alignments ##

Now that we have our alignments we need to trim those alignments. There are many programs that do this and I implore you to try a couple and see how different ones work, or even try manual trimming. Here we will use Trimal because its automated and works quickly when running alignments with lots of sequences.

You can run the following to trim your alignments:

```
while read p;do trimal -automated1 -in Dataset1_"$p".aln -out Dataset1_"$p".trimmed.aln; done < ../marker_list.txt
```

Once your sequences are trimmed I advise manually inspecting the file to ensure everything looks right.

## Step 6: Concatenate proteins ##

Now that you have the trimmed and aligned sequences you can concatenate these 16 files. To do this we will use the `concat` script packaged with BinSanity. 

Usage is shown below:
```
usage: concat -f directory -e Alignment Extension --Prefix file linker -o output

    *****************************************************************************
    *********************************BinSanity***********************************
    **     The `concat` script is used to concatenate multiple sequence        **
    **     alignments for conducting a phylogenomic analysis. Note that you    **
    **     receive an error if there are any duplicate sequence ids in an      **
    **     alignment.
    *****************************************************************************

optional arguments:
  -h, --help  show this help message and exit
  -f          Specify directory where alignments are
  -e          Specify the extension for your alignments (must be in Fasta format)
  --Prefix    Specify the prefix that links your alignments (ex: if you have two alignments TOBG_RpL10, TOBG_RpL24, the --Prefix would be TOBG
  -o          Specify output file
  -N          Specify the minimum number of sequences needed to be included in concatenation
  ```
  
  Run it as follows:
  
```
  concat -f . -e .trimmed.aln --Prefix Dataset1 -o Dataset1.Ribosomal.trimmed.concat.aln -N 8

``` 
## Step 7: Build the Tree ##

You now have a concatenated alignment 'Dataset1.Ribosomal.trimmed.concat.aln' which can be used to build a phylogenetic tree.

We will be using FastTree with the `-gamma` and `-lg` parameters. These parameters are optional and just indicate what models we would like to  use for branch length calculation and amino acid evolution respectively. Feel free to adjust these for your own purposes after looking at the FastTree Manual.

```
FastTree -gamma -lg Dataset1.Ribosomal.trimmed.concat.aln > Dataset1.Ribosomal.trimmed.concat.newick

```

## Step 8: View Tree ##

Now you have a newick file which can be viewed in a variety of tree views and edited. One of my favorite tools for making publication quality trees is [ITOL](https://itol.embl.de/)!
