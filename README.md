# Predicting Molecular Properties
In this repository I am collecting my Jupyter notebooks for the [Predicting Molecular Properties](https://www.kaggle.com/c/champs-scalar-coupling) kaggle competition

Here I am collecting only my final results leaving out all things done during the understanding phase of the project and all things that did not work.

The Data availabe for the competition can be found [here](https://www.kaggle.com/c/champs-scalar-coupling/data)
Among all the data available I have used only train.csv, test.csv files and I extracted features from the structure.zip (with xyz atoms coordinates for each molecule.

# The problem
We are required to predic the coupling of nuclear magnetic moment of pairs of atoms in a set of molecules. More precicely, given the ID of the atoms pair within a certain molecule we need to provide a prediction of the scalar coupling constant.

# A bit of domain knowledge
The coupling between nuclear magnetic moments of two atoms (J-coupling) depends from atoms features (Z, electronegativity, charge etc) and from environmental features (atoms distance, bonds between atoms, substituent groups atoms are bounded with etc.) For more details check "Understanding NMR Spectroscopy" of James Keeler one of the reference books for NMR spectroscopy.

# My solution
The idea is to manually extract a series of features for all atoms pair in each molecule for all the molecules. I am going to use [rdkit](https://www.rdkit.org/) to process the moleculecular structure. In order to use rdkit I changed the molecule's file format from .xyz to .mol using [xyz2mol](https://github.com/jensengroup/xyz2mol) script. I had to further change some of the files to sdf format due to problems in extracting structure from mol. Afterwards create a gradient boosting model with all hand extracted and generated features to capture the most relevant factors affecting the J-coupling.

Features_Extraction.ipynb preforms the extraction of features
I have extracted the following features:
- type (string representing of J-coupling, already provided)
- atom_0_type (it is allways H),
- atom_1_type (it can be H,C,N),
- atom_1_hybridization,
- number of pi_bonds between atom 0 and 1 (double bonds count as 1, triple bonds count as 2, and aromatic bonds count as 1.5), 
- graph_distance (through bonds distance in the minimal path from atom 0 to 1),
- graph_smile (string with atoms symbols from atom 0 to 1),
- angle (between atom 0 and 1 if there is one and only one other atom between them, otherwise angle = 0.0),
- dihedral (between atom 0 and 1 if there are two and two only atoms between atom 0 and 1, otherwise dihedral = 0.0),
- sum_electronegativity_inbetwen (between atom 0 and 1),
- sum_electronegativity_neghb (of atom's 1 neighborhoods),
- donor_groups_in_neighb (whether there are or not donor groups around atom j. 0=no, 1=yes),
- aceptor_groups_in_neighb (whether there are or not aceptor groups around atom j. 0=no, 1=yes),
- posIonizable_groups_in_neighb (groups bonded to atom 1 which can be positively charged),
- aromatic_groups_in_neighb (groups bonded to atom 1 which can be negativelly charged),
- hydrophobe_groups_in_neighb,
- lumpedHydrophobe_groups_in_neighb (specific hydrophobe group),
- negIonizable_groups_in_neighb (groups bonded to atom 1 which can be negativelly charged), 
- sigma_bonds (number of sigma bonds between atom 0 and 1)

FeaturesEng_Model.ipynb preforms features encoding and the model training
I have target econded the following features with the smoothed mean scheme:
- atom_1_type, 
- atom_1_hybridization, 
- pi_bonds, 
- graph_smile, 
- donor_groups_in_neighb, 
- aceptor_groups_in_neighb, 
- posIonizable_groups_in_neighb, 
- aromatic_groups_in_neighb, 
- hydrophobe_groups_in_neighb, 
- lumpedHydrophobe_groups_in_neighb, 
- negIonizable_groups_in_neighb, 
- sigma_bonds,

I have label encoded the following features: 
- type 
- atom_1_type 
- atom_1_hybridization 
- graph_smile

training set size = 4065979
test set size = 2505542
I used a gradient boosting model with 5-folds cross validation scheme (CV).
My final mean CV score (group MAE) was 0.2230, which resulted in a Private Score of 0.19351
and public score of 0.18922 in the competition's leaderboard.
