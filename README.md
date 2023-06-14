# BharatSim Documentation  

This repository contains the source code of the documentation of _BharatSim_, an agent based modelling framework for India. This documentation is hosted on ReadTheDocs [here](https://bharatsim.readthedocs.io/).


## Contributing

The documentation uses Sphinx with the Sphinx-RTD theme. Read about Sphinx [here](https://www.sphinx-doc.org/en/master/).

If you plan on contributing to this documentation, get in touch with us at bharatsim@ashoka.edu.in. Then, `fork` this repository on GitHub and make changes to your fork. Create and modify documentation pages inside the `source` directory. You can then submit a [pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) on GitHub. Your changes will then be merged after a review.

### Prerequisites

- Python 3 - Sphinx is based on Python, therefore you must have Python installed
- A Python package installer like `pip` or `conda`

### Setting up the documentation environment

The following commands will make a local copy of this repository, and install the required python packages.

- Clone the repository: `git clone git@github.com:bharatsim/documentation.git`
- Move into the repository: `cd documentation`
- Install requirements: `pip install -r requirements.txt`


### Building the Sphinx site locally

You can preview the final version of your local edits by building the HTML pages from your source. Note that this is only to check the final documentation. **The build files should not be committed to version control.** The final documentation site is built automatically from source files.

In order to build a local version of the HTML pages, navigate to within the repository and:

- If you have `make` installed, simply run `make html`
- If not, run `sphinx-build -b html source build/html`

You can now edit the source files and preview the results using any text editor. Sphinx uses [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html), a "a simple, unobtrusive markup language" that will not take you too long to learn if you have not already used it. 

If you wish, you can use a code editor like [VS Code](https://code.visualstudio.com) to edit the files, along with the [reStructuredText plugin](https://docs.restructuredtext.net) which includes many additional features, including a live preview.

## License

[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

