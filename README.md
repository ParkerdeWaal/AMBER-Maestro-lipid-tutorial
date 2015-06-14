# AMBER Maestro Protein-Lipid Tutorial

This tutorial provides a step-by-step protocol to build protein-lipid systems for AMBER using the free version of Maestro bundled with Desmond. In contrast to that of the CHARMM membrane system web builder, Maestro's system builder provides a greater level of control for system size and orentation within the membrane to optimize simulation NS/day.

Before proceeding, it is highly recommened to familiarize yourself with the offical AMBER lipid tutorial found [here](http://ambermd.org/tutorials/advanced/tutorial16/) as a working level knowledge of AMBER membrane systems is assumed.

##Requirments
* Any modern linux distribution
* [Desmond/Maestro](https://www.deshawresearch.com/downloads/download_desmond.cgi)
* [PyMOL](http://sourceforge.net/projects/pymol)

##Step 0: PDB preparation
Open and prepare your structure for LEaP input as normal in PyMOL or other PDB editing software. 

##Step 1: Determine Protein Membrane Orientation
Submit the cleaned membrane protein generated in **step 0** to the [OPM server](http://opm.phar.umich.edu/server.php) for optimal membrane orientation determination. **Always verify the membrane orientation before proceeding.**

##Step 2: Maestro Prep Wiz
Open Maestro and load the OPM oriented PDB file. Select the *Prep Wiz* tool found in the top toolbar and check the following settings. To run click preprocess.

![Prep](/images/proteinPrepWizard.png)

Note: While Maestro can automically assign CYX disulfides I prefer to perfer to assign them manually prior to importing the structure into Maestro.

After preprocessing, select the *refine* tab and click *optimize* to determine optimal side chain protonation states and hydrogen boding networks using PROPKA.

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

```bash
awk '{if ($4 == "SPC") printf("%4s %6d %-4s %-4s %5d %10.3f %7.3f %7.3f  1.00  0.00\n"), $1, $2, $3, $4, $5,$6,$7,$8; else print $0}' desmondOut.pdb > SPCfix.pdb
sed 's/SPC/WAT/' SPCfix.pdb > WATfix.pdb
awk '{if ($4 == "ACE" && $3 != "C") next; else print $0}' WATfix.pdb > ACEfix.pdb
awk '{if ($4 == "NMA" && $3 != "N") next; else print $0}'  ACEfix.pdb | sed 's/NMA/NME/' > NMEfix.pdb
charmmlipid2amber.py -c POPC.csv -i NMEfix.pdb -o LipidFix.pdb
sed 's/CL/Cl-/' LipidFix.pdb | sed 's/NA/Na+/' | sed 's/  CL/Cl-/' | sed 's/  NA/Na+/' > SaltFix.pdb
awk '/NME/{print $0 ;print "Ter";next}1' SaltFix.pdb > TerFIX.pdb
```

Next, open TerFIX.pdb and add CYX residues, if applicable, and renumber NME residues by +1 (NME A 348A -> NME A 349)

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

Utilizing these dimensions, here is an example TLeap input file:

```
source leaprc.ff14SB
source leaprc.lipid14

system = loadpdb ./protein/POLY.pdb

bond system.110.SG system.187.SG

set system box { 69.378655 64.033248 95.385489 }

saveamberparm system ./system/POLY.prmtop ./system/POLY.inpcrd

quit
```

**Due to Maestros naming of ions, it is important to check the total system charge before runing simunations. Typically a handful of ions will be removed due to naming duplications and must be added back via 'addIonsRand'**
