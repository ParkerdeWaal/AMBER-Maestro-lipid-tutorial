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

**Save the PDB as desmondOut.pdb, This is important for the script below to run automatically!**

##Step 5: reformat PDB
Copy POPC.csv, found in the includes folder, to the working folder and run the following terminal commands:

```
awk '{if ($4 == "SPC") printf("%4s %6d %-4s %-4s %5d %10.3f %7.3f %7.3f  1.00  0.00\n"), $1, $2, $3, $4, $5,$6,$7,$8; else print $0}' desmondOut.pdb > SPCfix.pdb
sed 's/SPC/WAT/' SPCfix.pdb > WATfix.pdb
awk '{if ($4 == "ACE" && $3 != "C") next; else print $0}' WATfix.pdb > ACEfix.pdb
awk '{if ($4 == "NMA" && $3 != "N") next; else print $0}'  ACEfix.pdb | sed 's/NMA/NME/' > NMEfix.pdb
charmmlipid2amber.py -c POPC.csv -i NMEfix.pdb -o LipidFix.pdb
sed 's/CL/Cl-/' LipidFix.pdb | sed 's/NA/Na+/' | sed 's/  CL/Cl-/' | sed 's/  NA/Na+/' > SaltFix.pdb
awk '/NME/{print $0 ;print "Ter";next}1' SaltFix.pdb > TerFIX.pdb
```
Next, open TerFIX.pdb and add CYX residues, if applicaplbe, and renumber NME residues by +1 (NME A 348A -> NME A 349)

![Renumb](/images/terminal.png)

##Step 6: TLeap input
Extract the box dimensions from *desmond_setup_1-out.cms* 

```
  69.378655
  0
  0
  0
  64.033248
  0
  0
  0
  95.385489

```

To set TLeap system dimensions add the following line in your leap.in:

`
set YOUR_NAME_HERE box { 69.378655 64.033248 95.385489}
`