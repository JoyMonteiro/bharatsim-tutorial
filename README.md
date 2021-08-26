# BharatSim Tutorial

Tutorial Documentation for the BharatSim agent based modelling framework

Link: https://bharatsim.readthedocs.io/

## Setting up the Documentation Environment

- Clone the repository: `git clone git@github.com:JoyMonteiro/bharatsim-tutorial.git`
- Move into the repository: `cd bharatsim-tutorial`
- Install requirements: `pip install -r requirements.txt`

The documentation uses Sphinx with the Sphinx-RTD theme.

## Documenting BharatSim

Read about [Sphinx](https://www.sphinx-doc.org/en/master/). Write documentation pages inside the `source` directory.

### Building HTML Pages

- If you have `make` installed, simply do `make html`.
- Else, `sphinx-build -b html source build/html`.

This is only to check the final documentation. The build files are not supposed to be committed to version control. 
The final documentation site is built automatically from source files.

After that, submit a Pull Request to the main repository. They will be merged after a review.
