# AMBER Maestro Protein-Lipid Tutorial

This tutorial provides a step-by-step protocol to build Protein-Lipid systems for AMBER with Maestro. Corresponding example

##Requirments
* [Desmond/Maestro](https://www.deshawresearch.com/downloads/download_desmond.cgi)
* [PyMOL](http://sourceforge.net/projects/pymol)

##Step 0: PDB preparation
Open and clean PDB using PyMOL. This can be done in Maestro, however the interface for PyMOL is much simpler.

##Step 1: Determine Protein Membrane Orientation
Submit membrane protein to the [OPM server](http://opm.phar.umich.edu/server.php) to determine optimal membrane orientation. 

##Step 2: Maestro Prep Wiz
Open the OPM oriented PDB file in Maestro and select the 'Prep Wiz' tool using the folloing settings:

![Prep](/images/proteinPrepWizard.png)

(I manually set CYX bonds in by editing the PDB with sublime)
