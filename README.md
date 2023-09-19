# PaleoOceanStarter
Procedural generation of idealised ocean temperatures to initialise paleoclimate model simulations. The Jupyter Notebook contains the Python implementation of the equations below
and was used to produce the initial ocean temperature and salinity distributions for the HadGEM3 Eocene simulations at multiple $CO_2$ levels (1x-4x). It can be easily adapted
to be used with other ocean grids.

![initial_ocean_profiles](https://github.com/sebsteinig/PaleoOceanStarter/assets/80361118/e62f3f29-313c-4354-ae59-d3def7f1ff12)
**Figure 1:** Global mean profiles of initial Eocene ocean temperatures generated with this method compared to reference data sets.

## idealised temperature distribution
The DeepMIP-Eocene experimental design paper (Lunt et al., 2017), suggests the following equation to generate the initial horizontal and vertical temperature distribution:

$$   T[^{\circ}C] =
\begin{cases}
  \left( \frac{5000-z}{5000} \times 25 \times cos \:\phi \right) + 15 & \text{if z $\leq$ 5000 m}\\
  15 & \text{if z > 5000 m} \\
\end{cases} 
\tag{1}
$$

Experience from the first DeepMIP-Eocene phase shows that this overestimnates the subsurface ocean heat, which resulted in extended spin-up durations to slowly remove this excess heat from the ocean (several thousand years). I therefore test a modified version of the default equation which is given by:

$$   T[^{\circ}C] =
\begin{cases}
  \left( \left(\frac{2500-z}{2500}\right)^6 \times T_{upper}  \times cos^2 \:\phi \right) + T_{deep} & \text{if z $\leq$ 2500 m}\\
  T_{deep} & \text{if z > 2500 m}\\
\end{cases} 
\tag{2}
$$

where $\phi$ is latitude and z is depth below the surface. $T_{upper}$ and $T_{deep}$ are scaling parameters that only depend on the global mean surface temperature (GMST). The relation between the GMST and the deep ocean temperatures is taken from a large ensemble of HadCM3BL simulations across the Phanerozoic (Fig. 11 in Valdes et al., 2021) and can be expressed as:

$$
T_{deep} = \frac{GMST - 15.4}{0.76}
\tag{3}
$$

With this, we can numerically solve for $T_{upper}$. 

Assumption:  We can calculate $T_{deep}$ and need to chose $T_{upper}$ so that at the surface (i.e. z=0), Eq. (2) reproduces the initial GMST value when averaged over the globe, i.e. given (for z=0 and weights=$cos \:\phi $):

$$
T[^{\circ}C] = T_{upper} \times weights^2 + T_{deep}
\tag{4}
$$

$$
GMST = \frac{\sum weights \times T}{\sum weights}
\tag{5}
$$

will give:

$$
T_{upper} =\frac{GMST - T_{deep}}{\frac{\sum weights \times weights^2}{\sum weights}} \approx \frac{GMST - T_{deep}}{0.67}
\tag{6}
$$

![T_deep_and_T_upper_vs_GMST](https://github.com/sebsteinig/PaleoOceanStarter/assets/80361118/f9b58422-33df-4bcf-b780-0da891a287b3)
**Figure 2:** Dependency of $T_{deep}$ and $T_{upper}$ parameters on global mean surface temperature (GMST).

## NEMO implementation
NEMO needs monthly mean 3D fields of temperature and salinity to initilise the ocean. Eocene temperatures are produced in the following way:
1. Copy modern reference file (for correct metadata).
2. Loop over each latitude and longitude and calculate the local vertical temperature profile. Necesseary due to curvilinear grid in NEMO. This could be simpliefied to save some computation time for more regular grids.
3. Salinities are set to a single global value of 34.7.
4. Save new temperature and salinity field to disk


