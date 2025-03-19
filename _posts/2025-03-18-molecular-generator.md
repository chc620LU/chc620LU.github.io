---
title: 'Molecular generator using RDKit'
date: 2025-03-17
permalink: /posts/2025/03/Molecular_generator_using_RDKit/
excerpt: ' '
tags:
  - RDKit
  - SMARTS
---

For my first couple of research projects, I used the Rule-input Network Generator [RING](https://conservancy.umn.edu/items/08233e5e-d6da-4a01-bdbb-85fa09c5a95b) to generate comprehensive reaction networks for the chemical problems we studied. RING is extremely useful—given elementary reaction rules and reactants, it automatically enumerates all possible reactions and products within specified constraints. The post-processing step allows users to easily extract reaction pathways and/or product molecules for further studies. Perhaps in a future post, I'll demonstrate how RING works using one of my completed projects.

In late 2022, for this particular project, I decided to try RDKit and built a small molecule generator since our goal was simply to produce diverse molecules for subsequent virtual screening.

Every time I need to work with SMARTS strings, I check out this [Daylight Theory Manual](https://www.daylight.com/dayhtml/doc/theory/theory.smarts.html) or [SMARTS Examples](https://www.daylight.com/dayhtml_tutorials/languages/smarts/smarts_examples.html). Unfortunately, I haven't found other structured or well-documented resources (a cheating sheet, basically) on SMARTS syntax, so these two websites have become my go-to references. When I first started writing SMARTS reactions, it was quite painful—the reaction products were not always what I expected. It took a lot of trial-and-error, constantly flipping back and forth between my code and the documentation until I finally achieved the desired results. Now that we're finally wrapping up this project and finishing the manuscript, I've decided to document the generator component and share it on my blog. I see this as my way of giving back—a toast to all the amazing researchers who generously shared their insights (and code!) with the community, from whom I’ve greatly benefited and drawn inspiration.


### Attach functional groups to aromatic scaffolds using RDKit SMARTS reactions

The purpose of this molecule generator is to create fictitious molecules by attaching various functional groups to pre-selected molecular scaffolds.

To achieve this, I use RDKit's function, <code>ReactionFromSmarts</code>. A detailed demonstration written by the RDKit developer on generating new molecules with SMARTS reactions can be found [Here](https://github.com/rdkit/rdkit-tutorials/blob/master/notebooks/003_SMARTS_ReactionsExamples.ipynb)


In the following example, I extract a scaffold from the scaffolds list and create a hydroxyl group (-OH) fragment.


```python
from rdkit import Chem
from rdkit.Chem import AllChem
from rdkit.Chem import Draw

# use IPythonConsole for pretty drawings
from rdkit.Chem.Draw import IPythonConsole
IPythonConsole.ipython_useSVG=False

# list of aromatics sacffolds in SMILES string
scaffolds = ['C1=CC=CC=C1', 'C12=CC=CC=C1C=CC=C2']

#poping out the first scaffold in the list
m_React = scaffolds.pop(0)
m_React = Chem.MolFromSmiles(m_React)

# hydroxyl group
OH_fragment = Chem.MolFromSmarts('[OX1H1]')

```

Since I want to attach two hydroxyl groups to the scaffold in an ortho position, creating a diol functionality, I use two hydroxyl groups on the reactant side. The aromatic carbons to which the hydroxyl groups will attach are specified using connected aromatic carbon atoms (<code>[c:1]:[c:2]</code>).<br>

After defining the reaction, I execute it using the <code>RunReactants</code> function.<br>

The reaction products are returned as a list of RDKit molecule objects, so I convert them back into SMILES strings. Here, I specifically set <code>kekuleSmiles=True</code> only because I need to perform theoretical geometric analysis on the molecule. Otherwise, leaving it at the default value (false) should generally suffice (I think).


```python
rxn_diol = AllChem.ReactionFromSmarts('[cH:1]:[cH:2].[OH:3].[OH:4]>>[OH:3]-[c:1]:[c:2]-[OH:4]')
products_diol = rxn_diol.RunReactants((m_React, OH_fragment,OH_fragment))

# The returns is Mol obejct
print(products_diol)

# convert mol object to SMILES strings
uniq_SMILES_diol = [Chem.MolToSmiles(x[0], kekuleSmiles = True) for x in products_diol]
print(uniq_SMILES_diol)
print(len(uniq_SMILES_diol))

Draw.MolsToImage(products_diol[0])
```

    ((<rdkit.Chem.rdchem.Mol object at 0x000001DFF89D7290>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94BA0>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E949E0>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94820>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94C80>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94CF0>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94D60>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94DD0>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94E40>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94EB0>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94F20>,), (<rdkit.Chem.rdchem.Mol object at 0x000001DFF8E94F90>,))
    ['OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1', 'OC1=C(O)C=CC=C1']
    12
    




    
![png](2025-03-18-molecular-generator_files/2025-03-18-molecular-generator_5_1.png)
    






```python

```


