# HiCPhylogenyWorkflow
This tutorial is an adaptation to the workflow described in
 **Graham, E. D., J. F. Heidelberg, and B. J. Tully. 'Potential for primary productivity in a globally-distributed bacterial phototroph.' The ISME journal (2018)**
 
The procedure described below was used to build a phylogenetic tree using 16 ribosomal proteins described in [Hug et al. 2016](https://doi.org/10.1038/nmicrobiol.2016.48) of metagenomic-assembled genomes (MAGs) from soil samples produced using ProxiMeta Deconvolution against [RefSoil+](https://doi.org/10.1128/mSystems.00349-18) genomes, a database of 922 soil microorganisms from RefSeq, for phylogenetic context.
In this repository you can run through the procedure using the example HiC MAGs fasta files and ribosomal protein calls from the RefSoil genomes present in the ref_soil folder to build a phylogenetic tree.
If you run this with your own data, this proecedure requires you to have metagenomic-assembled genomes ready for analysis.

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
