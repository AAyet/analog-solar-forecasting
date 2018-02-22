
# README #

This repository contains the code used to generate the figures for the following publication:
A.Ayet & P.Tandeo (2018) Nowcasting solar irradiance using an analog method and geostationary satellite images, *Solar Energy*.

The figures evaluate the performance of an Analog algorithm, used to forecast Global Horizontal Irradiance (GHI) on top of a solar energy source for up to 6 hours ahead. The algorithm uses as input satellite-derived irradiance maps post-processed by OSI SAF. The data is available free of charge [here](http://www.osi-saf.org/?q=content/meteosat-solar-surface-irradiance).


### Brief Description of the Analog Algorithm ###

The Algorithm is the intellectual property of [Elum Energy](http://elum-energy.com/). Access to the code can be granted for research purposes, on a case-by-case basis. This repository contains only the ouptuts of the algorithm.

The algorithm uses an Analog method.

* Given a satellite image of the current
state of the atmosphere (the *observation* or *truth*), we find in a historical
database one eighteen images that resemble the observation (the *analogs*).

* The analogs are found running a k-nearest neighbors algorithm on compressed
images (into four *features*).

* Then, for a lead time l, we look at what happened l hours after the
analogs (in the past), these are the *successors*.

* The analogs and the successors images are translated to optimally
match the truth

* The successors are then aggregated to produce a forecast. The operator
implemented here is the *local linear* aggregation operator. We fit a linear
regression between the analogs and the successors, and use the resulting
coefficients to propagate the observation and get a mean forecast and a
variance

The hypothesis behind the local linear operator is that the probability
distribution function of the truth is gaussian. One can however use directely
the successors as an ensemble of forecasts to estimate a different PDF.


### Structure of the repository data ###

The repository contains a 'data' folder containing the outputs the Analog prediction algorithm on the year 2016 over five BSRN stations (see the publication for further details). The algorithm is trained with the rest of the OSI SAF database.

Irradiance-maps results (section 6) are stored in 'data/0/results', and 'ground' results (section 7) in 'data/bsrn/results'. 

The outputs of the analog algorithm are saved in the following xarray format (see below for more details the xarray package)

```
#!python
point = xr.Dataset(({'forecast': (['time', 'size', 'lead'],
                                              forecasted[None, :, :]),
                                 'variance': (['time', 'size', 'lead'],
                                              variance[None, :, :]),
                                 'prop_ensemble': (['time', 'rank', 'lead'],
                                                   ensemble[None, :, :]),
                                 'ensemble': (['time', 'rank', 'lead'],
                                              n_ensemble[None, :, :]),
                                 'truth': (['time', 'lead'],
                                           truth[None, 1:]),
                                 'clim': (['time', 'lead'],
                                          clim[None, 1:])}),
                               {'time': [c_t],
                                'rank': range(ensemble.shape[0]),
                                'lead': np.arange(1, lead + 1),
                                'size': sizes})
```
Dimensions are the following:

- time: time of the observation 

- lead: forecast lead time from observation time (+1h, +2h , etc.)

- rank: labels the ensemble members (the analogs) before aggregation.

- size: number of analogs chosen for the aggregation (from 10 to 100, 10 by ten), for sensibility tests.

Variables are the following:

- forecast (time, size, lead): deterministic forecast

- variance (time, size lead): variance of the deterministic forecast (the forecast is gaussian)

- ensemble (time, rank, lead): the ensemble of forecasts (the translated successors), before aggregation.

- prop\_ensemble (time, rank, lead): the ensemble of forecasts obtained after propagating the translated analogs with the linear regression $B$ (corresponding to the ensemble in part 2 of Fig. 6).

- truth (time, lead): observed GHI value 

- clim (time lead): montly climatology

Note that the two last variables contain redudant information: 'truth' at (time: 2016-01-01T10:00, lead: 2) is identical to 'truth' at (time: 2016-01-01T11:00, lead: 1).

This particular storage format was used to make score computation easier, as in this case it suffices to compute
```
(point.forecast - point.truth)**2
```
to get the squared error, since xarray package matches dimensions.


### Notebooks contained in the repository ###

The repository contains two jupyter notebooks using the data contained in the data folder:
- 1\_Figure\_5-8-9 plots figures 5, 8 and 9. It also generates some of the data (associated to the ensemble scores)
- 2\_Figure\_10-13 plots figures 10 to 13. It also shows the results of the Diebold-Mariano test (recquires an external library).


### Quick introduction to xarray ###

The data has been stored using [xarrays](http://xarray.pydata.org/en/stable/) datasets.
An xarray dataset is a multidimentional collection of nummpy arrays, with
common dimensions that have values and a name. It is usefull to handle multiple
meteorological variables with latitude, longitude and time as dimensions for
instance. From a set of numpy arrays (var1, var2) of dimension three (dim1, dim2 and dim3 are 1D lists or arrays of the labels of the dimensions),
an xarray is created as

```
#!python

import xarray as xr

xr.Dataset({'variable1': (['dimension1', 'dimension2', 'dimension3'],
                          var1),
            'variable2': (['dimension1', 'dimension2', 'dimension3'],
                          var2)},
            {'dimension1': dim1,
             'dimension2': dim2})
```

*IMPORTANT*: the order of the dimensions when defining the variables  is *VERY* important:

```
#!python
xr.Dataset({'variable1': (['dimension1', 'dimension2', 'dimension3'],
                          var1)....
```

is different than

```
#!python
xr.Dataset({'variable1': (['dimension2', 'dimension3', 'dimension1'],
                          var1)....
```

### Contact ###

Alex Ayet
alex.ayet@ifremer.fr
https://www.eleves.ens.fr/home/ayet/

Current PhD subject
http://www.umr-lops.fr/Recherche/Theses-en-cours/Alex-Ayet
