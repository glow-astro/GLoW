# GLoW: Gravitational Lensing of Waves

![GLoW logo](./sphinx_doc/diagrams/glow_logo.png#gh-light-mode-only)
![GLoW logo](./sphinx_doc/diagrams/glow_logo_dark.png#gh-dark-mode-only)

If you use this code, please cite this repository and the main GLoW paper
[Villarrubia-Rojo+ 24](https://inspirehep.net/literature/2826315).

The online documentation can be found
[here](https://miguelzuma.github.io/GLoW_public/index.html).

In addition, GLoW has been used in a number of other works, which are
[listed on the project site](https://glow-astro.org/glow-code.html#papers). That
list is maintained there rather than here, so it stays current: if your paper is
missing, or is listed and should not be, say so and it will be corrected.

## Installation

![GLoW-Light](./sphinx_doc/diagrams/diagram_simp.png#gh-light-mode-only)
![GLoW-Dark](./sphinx_doc/diagrams/diagram_simp_dark.png#gh-dark-mode-only)

The pure Python version of the code should work out of the box. It only requires standard scientific
packages like ``numpy`` and ``scipy``.

The C version requires an external library, the GNU Scientific Library (GSL), that can be easily
installed with your favorite package manager. Alternatively, if you are using Conda, you can install
and activate the environment that we provide
```console
conda env create --file glow_env.yml && conda activate glow_env
```
Once the previous requirements are met, the code can be easily installed by running
```console
pip install .
```
in the main GLoW directory. If any error occurs, the file ``configure.log`` will contain additional
information. Open MP is also used to run certain parts of the code in parallel, but it is not
mandatory. If it is not correctly set up, the installation will configure the code in serial mode.

More detailed installation instructions can be found in the
[online documentation](https://miguelzuma.github.io/GLoW_public/usage.html#installation).
