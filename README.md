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

(While Maestro can automically assign CYX disulfides I prefer to perfer to assign them manually.)

Optimize side chain protonation states and the hydrogen bonding network using PROPKA by clicking 'optimize'

![PROPKA](/images/refine.png)

**Always manually verify protonation states and side chain orentations manually by selecting 'Interactive Optimizer'**

##Step 3: System Building
Select Tasks > Molecular Dynamics > System Setup. Under 'Edit Membrane...' select POPC (300K) and click 'Place on Prealigned Structure'. Under 'Ions' select 'Neutralize by adding' and 'Add salt' to 0.15 molar concentration

![Salt](/images/salts.png)

Click Run. Note: For larger systems this may take up to several minutes.

![DesRaw](/images/desmondRaw.png)

##Step 4: Removing Hydrogens
Select Tools > View In PyMOL > Workspace. Remove hydrogens on all protein and ligand residues (ACE -> NMA caps) *NOT* on lipids! 

![HydroRemove](/images/removeHydrogens.png)





