This repository contains the likelihood modules for the KiDS-1000 (in short: K1K) COSEBIs, Bandpowers, and 2-point correlation function measurements from [Asgari et al. 2020 (arXiv:2007.15633)](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A).
The module will be working 'out-of-the-box' within a KCAP setup and an additional install of [MontePython](https://github.com/brinckmann/montepython_public) and [CLASS](https://github.com/lesgourg/class_public) (version >= 2.8 including the HMcode module) setup. The required KiDS-1000 data files can be downloaded from the [KiDS science data webpage](http://kids.strw.leidenuniv.nl/sciencedata.php) and the parameter file for reproducing the fiducial run of [Asgari et al. 2020](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A) is supplied in the subfolder `INPUT`.

The '_2cosmos' subfolders contain the likelihood modules that were used in Section B.2 of [Asgari et al. 2020](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A) for a consistency analysis following the methodology developed in [Köhlinger et al. 2019 (MNRAS, 484, 3126)](http://adsabs.harvard.edu/abs/2019MNRAS.484.3126K).
These likelihoods can only be run with the modified 'Monte Python 2cosmos' version of Monte Python, i.e. [montepython_2cosmos_public](https://github.com/fkoehlin/montepython_2cosmos_public) which calls two independent instances of [CLASS](https://github.com/legourg/class_public) (version >= 2.8 including the HMcode module) and required KCAP modules and which also keeps the relevant cosmological parameter dictionaries separate, so that each subset of the mutually exclusive split of the main dataset is assigned its own copy of cosmological (and nuisance) parameters. The two subsets are still coupled through their joint covariance though to properly account for their correlations. To compare the split parameters then to the joint parameters and to compare the evidences for the Bayes factor a comparison run with the regular K1K likelihoods is required within a standard [MontePython](https://github.com/brinckmann/montepython_public) and [CLASS](https://github.com/lesgourg/class_public) (version >= 2.6 and including the HMcode module) setup. The required KiDS-1000 data files can be downloaded from the [KiDS science data webpage](http://kids.strw.leidenuniv.nl/sciencedata.php) and the parameter file for reproducing the split runs of [Asgari et al. 2020 (arXiv:2007.15633)](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A) is supplied in the subfolder `INPUT`.

Assuming that KCAP and MontePython (with CLASS version >= 2.8 including the HMcode module) are set up (we recommend to use nested sampling), please proceed as follows:

1) Copy `K1K_COSEBIs`, `K1K_BandPowers`, and `K1K_CorrelationFunctions` or their '2cosmos' equivalent from this folder into `/your/path/to/montepython_public/montepython/likelihoods/`.

(you can rename the folder to whatever you like, but you must use this name then consistently for the whole likelihood which implies to rename the `*.data`-file, including the prefixes of the parameters defined in there, the name of the likelihood in the `__init__.py`-file and also in the `*.param`-file.)

2) Set the path to the data folder (i.e. `KiDS-1000_DATA_RELEASE` from the tarball available from the [KiDS science data webpage](http://kids.strw.leidenuniv.nl/sciencedata.php') in `K1K_[COSEBIs, BandPowers, or CorrelationFunctions].data` and modify parameters as you please (note that everything is set up to reproduce the fiducial run with `K1K_[COSEBIs, BandPowers, or CorrelationFunctions].param`).

3) Start your runs using e.g. the `K1K.param` supplied in the subfolder `INPUT`.

4) If you publish your results based on using this likelihood, please cite [Asgari et al. 2020 (arXiv:2007.15633)](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A) and all further references for the KiDS-1000 data release (as listed on the [KiDS science data webpage](http://kids.strw.leidenuniv.nl/sciencedata.php)) and also all relevant references for KCAP, Monte Python and CLASS.

Refer to `mpirun_with_multinest.sh` or `mpirun_with_polychord.sh` within the subfolder `INPUT` for all MultiNest and PolyChord-related settings that were used for the fiducial runs.

As of version 3.3.0 of MontePython, S_8 is not implemented as a sampling parameter. To reproduce the fiducial run of [Asgari et al. 2020](https://ui.adsabs.harvard.edu/abs/2020arXiv200715633A) you need to enable S_8 sampling by adding the following lines to the for loop starting in line 789 of `/your/path/to/montepython_public/montepython/data.py`:
```python
if elem == 'S_8':
    h = self.cosmo_arguments['h']
    # infer sigma8 from S_8, h, omega_b, omega_cdm, and omega_nu (assuming one standard massive neutrino and omega_nu=m_nu/93.14)
    omega_b = self.cosmo_arguments['omega_b']
    omega_cdm = self.cosmo_arguments['omega_cdm']
    #
    try:
        omega_nu = self.cosmo_arguments['m_ncdm'] / 93.14
    except:
        omega_nu = 0.
    self.cosmo_arguments['sigma8'] = self.cosmo_arguments['S_8'] * math.sqrt((0.3*h**2) / (omega_b+omega_cdm+omega_nu))
    del self.cosmo_arguments[elem]
```
In HMcode you can specify a baryonic feedback model which depends on two parameters: the minimum concentration "c_min" from the and the halo bloating parameter "eta_0". These can be related via eta_0 = a_0 + a_1 * c_min  (eq.30 in [Mead et al. 2015](https://ui.adsabs.harvard.edu/abs/2015MNRAS.454.1958M)). For the fiducial KiDS-1000 analysis we use a_0 = 0.98 and a_1 = -0.12.
Class currently doesn't support a_0 and a_1 as input parameters, but instead uses some hardcoded values (see line 2906 in https://github.com/lesgourg/class_public/blob/master/source/input.c). These values are equivalent to the ones used in the fiducial KiDS-1000 analysis.

If you want to change the values of a_0 and a_1, you need to add the following lines to `/your/path/to/montepython_public/montepython/data.py` in order to calculate eta_0 from a_0, a_1, and c_min.
```python
if elem == 'c_min':
    c_min = self.cosmo_arguments['c_min']
    # if a_0 and a_1 are not defined use the standard values given in equation (30) in Mead et al. 2015
    try:
        a_0 = self.cosmo_arguments['a_0']
    except:
        a_0 = 1.03
    try:
        a_1 = self.cosmo_arguments['a_1']
    except:
        a_1 = -0.11
    self.cosmo_arguments['eta_0'] = a_0 + a_1 * c_min
    del self.cosmo_arguments['a_0']
    del self.cosmo_arguments['a_1']
```
You can then set the values of a_0 and a_1 by adding the following lines to the .param file:
```python
data.parameters['a_0']   = [0.98,   0.5,    1,       0,      1, 'cosmo']
data.parameters['a_1']   = [-0.12,  -0.3,   0.3,     0,      1, 'cosmo']
```
Note: a_0 and a_1 need to be defined defined in the parameter file (e.g.`INPUT/K1K.param`) as **fixed** cosmological parameters.

WARNING: This likelihood only produces valid results for `\Omega_k = 0`, i.e. flat cosmologies!
