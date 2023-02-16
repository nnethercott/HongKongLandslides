# HongKongLandslides

<p align="center">
    <img src="./media/pcna.png" height="350" alt="pcna_nate"/>
    <img src="./media/reg1a.png" height="350" alt="reg1a_luca"/>
</p>

|**Languages** | **Libraries** |
| -----| ---- |
|![R](https://img.shields.io/badge/R-ff1111) |![sf](https://img.shields.io/badge/sf-1.0.9-ff1111) ![rgdal](https://img.shields.io/badge/rgdal-1.6.2-ff1111) ![sp](https://img.shields.io/badge/sp-1.4.6-ff1111) ![spatstat](https://img.shields.io/badge/spatstat-3.0.2-ff1111) <br> ![rstan](https://img.shields.io/badge/rstan-2.21.7-ff1111) ![maptools](https://img.shields.io/badge/maptools-1.1.3-ff1111) ![MASS](https://img.shields.io/badge/MASS-7.3.54-ff1111)

<a name="description"/>

## Description
Modelling historical landslide occurrences in Hong Kong through Bayesian statistical analysis.  Spatial covariance between observation sites is modelled and leveraged for informative kriging in a predictive phase. 

Relevant datasets for the project can be found through the [Enhanced Natural Terrain Landslide Inventory](https://data.gov.hk/en-data/dataset/hk-cedd-csu-cedd-entli) repository provided by the government of Hong Kong.  

The two main scripts driving this project are `mask_generation.R` which extracts the grid cells corresponding to hong kong from the bounding box, and `kriging.R` in which model fitting and prediction are performed. Hyperparameters in the former with respect to the level of grid refinement can allow for faster model fitting times with rstan but at the expense of potential non-informativeness of the geostatistical covariates aggregated over the larger grid cells. 

<a name="motivation"/>

## Motivation
In this project we attempt to uncover the relationship between geostatistical covariates and the frequency of landslides in a given region, whilst also considering the influence each region exerts on one another due to spatial proximity. 

Through relevant Bayesian modelling which uncovers influential covariates in our generalized linear model (GLM) stands the potential for informing government spending to implement relevant infrastructures for reinforcing identified at-risk regions, which consequently could save lives in the process.

<a name="dataset"/>

## The Dataset

|**Covariate** | **Description** |
| -----| ---- |
| SLIDE_TYPE | categorical variable classifying the observations as relict landslide("R”),  landslide with recent channelized debris flow ("C”), landslide with open hillslope ("O”), recent coastal landslide ("S”) |
| M_WIDTH | width of the landslide main scarp (in metres) |
| S_LENGTH | length of the landslide source area (in metres) |
| SLOPE | ground slope angle across the landslide head |
| COVER | categorical variable classifying the landslides based on the vegetation cover. Values fall into the categories of totally bare of vegetation ("A"), partially bare of vegetation ("B"), completely covered by grass ("C"), covered in shrubs and/or trees ("D") |
| YEAR_1 | year of the serial photograph on which the landslide was first observed |
| HEADELEV | elevation of the landslide’s crown (in mPD, i.e. metres above principal datum) |
| TAILELEV | elevation of landslide toe (in mPD) |
| ELEV_DIFF | elevation difference between landslide crown and toe in metres |
| GULLY | categorical variable classifying landslides belonging to a previously recorded area of gully erosion ("Y”) and landslides belonging outside of this area ("N”) |
| NORTHING & EASTING | coordinates of the landslides |

<a name="model"/>

## Model 
We consider a Cox generalized linear model with link function given by the exponential to model the intensity surface which drives landslide occurences in each grid cell, $s_{i}\in D\subset \mathbb{R}^{2}$. Essentially this intensity surfce takes into account underlying geospatial covariates along with assumed noise given by measurement errors and influences due to spatial proximity, and maps them to a scalar denoting the rate of landslide occurences in the given cell. 

We assume an exponential covariance function, $\rho$, though one can certainly consider a CAR formulation in which the sparsity of the proximity matrix allows for faster computations at higher levels of grid refinement. 

$$\begin{align*}Y(s_{i}) | \lambda &\sim Po(\lambda(s_{i}))\\\\\\
\log \lambda(s_{i}) | \beta, w, \epsilon &= X(s_{i})\beta + w(s_{i}) + \epsilon(s_{i})\\\\\\
\beta | \nu &\sim \mathcal{N}(\textbf{0}, \nu^{2} I_{k+1})\\\\\\
\epsilon(s_{1}), \epsilon(s_{2}), ... \epsilon(s_{n}) | \tau^{2}&\overset{iid}{\sim} \mathcal{N}(0, \tau^{2})\\\\\\
\textbf{w} | \sigma^{2}, \phi &\sim \mathcal{N}_{n}(\textbf{0}, \sigma^{2}\rho(\cdot;\phi))\\\\\\
\tau^{2} &\sim inv-gamma(1,1)\\\\\\
\nu^{2} &\sim inv-gamma(1,1)\\\\\\
\sigma^{2} &\sim \mathcal{HC}(0,1)\\\\\\
\phi &\sim inv-gamma(4,1)\end{align*}$$

Data used to train the model consists of aggregated landslide counts in the grid cells defining hong kong.  These labels - and the underlying features - are highly subjective to the hyperparameter which controls the grid refinement.  To choose grid dimensions which provide an adequate trade-off between covariate inhomogeneity and cell granularity we plot elbow curves of the within sum of squares for covariates in the cells and make an informed choice. 

PICTURES OF THE GRIDS, ALSO OF THE ELBOW


