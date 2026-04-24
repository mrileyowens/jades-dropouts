Things to do:

- Why does the final selection for sufficiently sensitive rest-optical colors drop such a large number of objects from the southwest region of GOODS-S?
- Compare the photometry of my JADES selection with that of E24
- Double check aperture characteristics of the convolved JADES KRON_S photometry compared to E24
- Directly compare the bulk properties of both selections
- Confirm if $\sim$ 90% of the brightest galaxies in my dropout selection are also recovered by E24
- Look through the Bagpipes fits to the updated catalog (using the adjusted low-SNR method)
- Add figures to compare_phot.ipynb that summarize the distributions in a single figure, so it's clear if there are any peristent systemic differences
- Combine compare_endsley2024() and compare_jades() into a loop
- Confirm the number of objects from the E24 catalog that have no coordinate match in the full JADES catalog
- Add descriptions of the purpose of each selection criteria / color
- Finish the initial condition table, including those I added compared to E24
- Adjust the far-UV flux density estimation to instead use the median flux density between 1450-1550 angstroms in the rest frame, instead of interpolating at 1500 angstroms

# Description of the code

This repository is my attempt to replicate the F775W dropout selection by [Endsley et al. (2024)](https://doi.org/10.1093/mnras/stae1857) (hereafter E24), meant to select $z\sim6$ in the JADES fields. An explanation of the repository's content follows.

First, this project does not attempt to reproduce the source identification and extraction of E24. Instead, I start from the JADES photometric catalogs, found at [the JADES HLSP page](https://archive.stsci.edu/hlsp/jades) in the "Direct Download" section. Specifically, the catalogs labeled "NIRCam Photometry (GOODS-S-Deep v2.0)" and "NIRCam Photometry (GOODS-N v1.0)". These store, for the GOODS-S and GOODS-N fields, respectively, the photometric object catalog from the JADES team. The individual catalogs exist in `data/`, named like `hlsp_jades_jwst_nircam_{field}_photometry_v{version number}_catalog.fits`.

The Jupyter notebook `select.ipynb` merges these catalogs together and then applies the selection criteria of E24 on the joint catalog.

The function `merge()` in `select.ipynb` merges the two catalogs, appropriately combining their headers and several (but not all) of their extensions. Namely just the extensions concerning the filter properties, data quality control flags, detection image extraction properties, and the Kron photometry of the PSF-convolved images to F444W resolution. The result is a single catalog file containing all the GOODS-N and GOODS-S photometric objects, saved in `results/catalogs/` as `hlsp_jades_jwst_nircam_goods-n_v1.0_goods-s-deep_v2.0_photometry_catalog.fits`. This joined catalog, as well as the constituent catalogs, have large on-disk sizes, so they are not synchronized with GitHub.

The F775W dropout selection then begins on the joint catalog with `drop()`, which performs an initial criteria enforcement. First, I adjust low-SNR photometry in F606W and F775W, two key filters we will use to calculate Lyman break colors that lie shortward of the Lyman break location we want to select for, by directly adopting the uncertainties in those filters as the observed photometry when SNR < 1. Crucially to later steps, though, I calculated the F606W SNR before this adjustment.

The remainder of this function calculates boolean lists to apply jointly to the catalog, each list corresponding to one of a set of initial criteria to enforce against the joint JADES photometric catalog, summarized in the table below.

>| Condition | Purpose | E24 |
>| - | - | - |
>| F775W - F090W > 1.2 | - | Y |
>| F090W - F150W < 1.0 | - | Y |
>| F775W - F090W > F090W - F150W + 1.2 | - | Y |
>| (SNR(F435W) < 2 and F606W - F090W > X) or F775W - F090W > 2.5 | - | Y |
>| Any NIRCam filter with SNR > 5 | - | Y |
>| At least 3 NIRCam filters with SNR > 3 | - | Y |
>| F814W or F850LP SNR > 3 | - | Y |
>
>**Table:** Initial photometric criteria.

After applying these conditions to the joint JADES photometric catalog, so that only the objects satisfying all the criteria remain, the function saves the resulting catalog to `results/catalogs/` as `hlsp_jades_jwst_nircam_goods-n_v1.0_goods-s-deep_v2.0_photometry_catalog_f775w_dropouts_init.fits`.

The next selecting function, `fit()`, finalizes the dropout selection with one more condition: $f(\text{FUV}) / \sigma\left(X\right) > 3$ , for $X\in\{\text{F335M}, \text{F356W}, \text{F410M}, \text{F444W}\}$, where $f\text{FUV})$ is the far-UV flux density in the rest frame. <!--E24 state that $f(\text{FUV})$ corresponds to the inferred $M_\text{UV}$ from the BEAGLE CSFH SED fits, though they are not clear about how they measured $M_\text{UV}$ from those SEDs. In a parallel project to reproduce the inferred EW distributions of E24, I measured $M_\text{UV}$ based on the median flux density of the SEDs in the wavelength range $1450-1550$ $\textrm{\AA}$ in the rest frame; this work adopts the interpolated flux density at 1500 $\textrm{\AA}$ in the rest frame.A-->

This guarantees...

Seeing as we have already selected for the $z\sim6$ character of the objects, one could crudely estimate this quantity from the observed photometry, perhaps by directly adopting the flux density of a rest-UV filter at this redshift, or correcting to 1500 $\textrm{\AA}$ with a UV slope, based on the color between two rest-UV filters. Neither approach is likely to be very accurate, as the specific redshift is unknown, and the photometric selection for $z\sim6$ can permit a wide redshift range.

Forgoing collecting spectra, that means we need to perform SED fits to the photometry to estimate the exact redshift, and correspondingly the spectrum's far-UV flux density. I used Bagpipes to perform this SED fitting. The table below lists the adopted Bagpipes parameters.

>| Parameter | Prior |
>| - | - |
>| Cosmology | ??? |
>| IMF | ??? |
>| SFH | CSFH |
>| Redshift | $5 < z < 7$ |
>| Time since SF began | $0 < \text{log}_{10}(t/\text{Gyr}) < 0.5$ |
>| Time since SF ended | $0 < \text{log}_{10}(t/\text{Gyr}) < 0.5$ |
>| Stellar mass formed | $6 < \text{log}_{10}(M_\star/\text{M}_\odot) < 13$ |
>| $A_\text{V}$ | $0 < A_\text{V} < 2$ |
>| Ionization parameter | $\text{log}_{10}(U)=-2$ |
>| Metallicity | $0 < Z/\text{Z}_\odot < 0.5$ |
>| Dust law | [Calzetti et al. (2000)](https://doi.org/10.1086/308692) |
>
>**Table:** Bagpipes parameters adopted for the SED fitting.

The `pipes/posterior/hlsp_jades_jwst_nircam_goods-n_v1.0_goods-s-deep_v2.0_photometry_catalog_f775w_dropouts_init.fits/` folder contains the posteriors of the SED fits, each labeled by their ID in the catalog. `fit()` also plots those fits against the observed photometry, stored in `figs/bagpipes_fits/`.

E24 state that $f(\text{FUV})$ corresponds to the inferred $M_\text{UV}$ from the BEAGLE CSFH SED fits, though they are not clear about how they measured $M_\text{UV}$ from those SEDs. In a parallel project to reproduce the inferred EW distributions of E24, I measured $M_\text{UV}$ based on the median flux density of the SEDs in the wavelength range $1450-1550$ $\textrm{\AA}$ in the rest frame; this work adopts the interpolated flux density at 1500 $\textrm{\AA}$ in the rest frame. `fit()` measures and saves the far-UV flux density calculated in this fashion from the SEDs in `hlsp_jades_jwst_nircam_goods-n_v1.0_goods-s-deep_v2.0_photometry_catalog_f775w_dropouts_init_f1500.h5` in `results/`.

After applying this second set of conditions to the pared-down catalog, the code saves the resulting catalog in `results/catalogs/` as `hlsp_jades_jwst_nircam_goods-n_v1.0_goods-s-deep_v2.0_photometry_catalog_f775w_dropouts_final.fits`.

<!--
<p float="left" align="middle">
    <img src="figs/bagpipes_fits/compare_ew_errors.png" width=33%/>
</p>
-->
















This repository attempts to replicate the selection of F775W dropout galaxies described by Endsley et al. (2024) (herafter E24) in the GOODS-N and GOODS-S fields by using the JADES photometric catalog.

The notebook `select.ipynb` performs the actual selection from the photometry, first in `drop()`, which makes all but the last selection from E24. I also added two adjustments, compared to the stated methodology of E24: 

1. **Reassign negative flux densities in the ACS F606W and F775W filters.** E24 adopted the $1\sigma$ upper limit for low-SNR photometry in Lyman break-shortward filters used to calculate colors (namely ACS F606W and F775W), which is a common practice. When repeating that for the JADES catalog, some objects still have negative flux densities, which means the corresponding magnitude (and any dependent colors) are undefined. This felt like a somewhat arbitrary data artifact, particularly for objects that seemed unfairly disqualified over this. So, for objects which still had negative flux densities after adopting the $1\sigma$ upper limit, I assigned a tiny, positive flux density to those filters: $10^{-3}$ nJy. This should still capture what I think is the point here: that these objects have flux densities basically consistent with 0 in these filters.

2. **Discard objects with undefined uncertainties.** Some objects have photometry with undefined uncertainties. I don't think it's appropriate to include those objects when one of the relevant filters to the selection criteria has an undefined uncertainty, so I also enforced that a handful of the most important filters must have a finite uncertainty.

I also made use of the JADES catalog's flags to remove foreground stars and objects contaminated by bright stars or other neighbors.

After making the initial selection, `drop()` saves the resulting catalog. This is necessary because the final selection requires making SED fits to estimate $f_\text{1500}$, and it would be cost prohibitive to do so for the entire JADES catalog before paring it down. E24 performed this SED fitting with BEAGLE, but I chose to use Bagpipes instead, since it is much quicker, and for the purposes of calculating $f_\text{1500}$, all that is necessary is to accurately reproduce the overall SED shape, and not necessarily accurate masses, metallicities, etc.

The function `fit()` contains the Bagpipes fitting, which uses an identical filter set to E24, even though more filters are available in the JADES catalog. Using the resulting Bagpipes fits, `fit()` makes the final selection of E24 and saves the remaining objects to a new catalog.

The remaining code analyzes the results of this selection.

# How do the two selections compare?

The two selections are not identical. Only 114 objects are common to both selections.

# Why aren't the two selections identical?

## Different photometry

### Comparing photometry of the Endsley et al. (2024) galaxies with the corresponding photometry in JADES

### Comparing photometry of the E24 galaxies with the corresponding photometry in JADES

One insightful exercise to explore differences between JADES photometry and that of E24 is to directly compare the two for the same set of objects. I made this comparison with the function `compare_catalog_endsley2024()` in `compare_phot.ipynb` by plotting the two sets of photometry against each other. 

Of the 278 F775W dropouts in E24, 268 have a corresponding source in the JADES catalog (i.e., an object within 0.1 arcsec of a coordinate in the other catalog). Summarizing the results of those figures:

- **Photometry.** The width of the photometric distributions may be slightly wider in the JADES catalog. But more importantly, **every** filter's photometry in JADES is $\sim1$ mag fainter.
- **SNR**. The SNR in the F435W and F606W filters appear consistent, and the SNR in the F410M and F444W filters may be a few higher in the JADES catalog. Otherwise, the SNR may be a few higher in the E24 catalog, though the discrepancy becomes much stronger at the high-SNR end.
- **Colors.** The JADES catalog seems to prefer redder F775W - F090W and F606W - F090W colors, but the F090W - F150W color is extremely consistent between the two catalogs. It may be notable that the two colors involving HST filters are discrepant, but not the color with just JWST filters. Though it may also be downstream of the fact that the F775W and F606W filters are shortward of the Lyman break at the targeted redshift.

### Comparing photometry of the E24 galaxies based on membership in the JADES dropout sample

Using `compare_endsley2024()` in `compare_phot.ipynb`, I compared the photometry of the F775W dropout catalog based on membership in my own JADES dropout sample. Of the 278 galaxies, 114 are also in my JADES dropout sample, and they tend to be brighter, higher SNR, and possibly have redder colors than those that are not also in my JADES dropout sample.

### Comparing photometry of the JADES dropout sample galaxies based on membership in the E24 dropout sample

The function `compare_jades()` in `compare_phot.ipynb` compares the photometry of the F775W dropout sample I made from the JADES photometric catalog, based on membership in E24's F775W dropout catalog. Of the 290 galaxies, 114 are also in the E24 sample, and 176 are not. Those that are also in the other sample seemed to trend slightly toward being brighter (tenths of a magnitude, probably) in Lyman break longward filters, slightly fainter in Lyman break shortward filters, higher SNR, and slightly redder colors.

### Open questions

- Why are 10 of the F775W dropouts of E24 missing in the JADES catalog?
- What are the reasons why some of the F775W dropouts from E24 do not have a well-defined color? And how important is each?

## What criteria do unrecovered E24 galaxies fail?

The function `compare_conds_fails()` in `compare_phot.ipynb` investigates the specific points of failure for the E24 F775W dropout galaxies not recovered by this work's F775W dropout selection process. The table below summarizes the exact conditions of the selection that these galaxies fail.

>| Condition | Count |
>| - | - |
>| F775W - F090W > 1.2 | 20 |
>| F090W - F150W < 1.0 | 9 |
>| F775W - F090W > F090W - F150W + 1.2 | 31 |
>| (SNR(F435W) < 2 and F606W - F090W > X) or F775W - F090W > 2.5 | - |
>| Any NIRCam filter with SNR > 5 | - |
>| At least 3 NIRCam filters with SNR > 3 | - |
>| F814W or F850LP SNR > 3 | 55 |
>| $f(FUV) / \sigma(X)>3\text{ }\forall X\in\{\text{F335M}, \text{F356W}, \text{F410M}, \text{F444W}\}$ | 39 |
>
>**Table:** The conditions failed by unrecovered E24 galaxies. Galaxies may fail more than one condition, so the sums of the counts do not total the number of unrecovered galaxies. The selection process only tests the second set of conditions for galaxies that pass the first set, so in those cases that a galaxy fails during the first set of conditions, the second does not count as a failure.

The additional conditions I set, which E24 do not explicitly mention, are not important reasons for nonrecovery. Most of those conditions (3/4) only fail single digits of the E24 galaxies. At most, 17 of the unrecovered E24 galaxies fail one of the additional conditions.

Instead, there are a few of the criteria explicitly mentioned by E24 that are the dominant failure modes. 

### An insufficient F775W break

First is the F775W break requirement (F775W - F090W > 1.2), which fails 20 of the unrecovered E24 galaxies. As noted in another section, there are systematic differences in the photometry between the two catalogs. If it were consistent between filters, then *colors* should be preserved. This does not appear to be the case. Confusingly, the below figure suggests that the JADES photometry actually predicts slightly *redder* F775W breaks. That, however, is balanced by the large number of non-finite colors (probably due to undefined or negative filter photometry in either reduction).

<!--
From visually analyzing the graph, only a few galaxies appear to have an insufficient F775W break in JADES but sufficient F775W break in E24, so most of the failures of this condition are probably due to 

<p float="left" align="middle">
    <img src="figs/photometry_comparison/endsley2024_f775w_dropouts_vs_jades_ACS_F775W_NRC_F090W_color.png" width=48%/>
</p>
-->

### A Lyman break weaker than the UV slope

The ... (F775W - F090W > F090W - F150W + 1.2) requirement fails 31 of the E24 galaxies.

### No clear detection in F814W or F850LP

Another important reason for failure is requiring a clear (SNR > 3) detection of the galaxy in either F814W or F850LP. This requirement fails 55 of the E24 galaxies, or about one third of the total discrepancy; more than any other requirement. The first figure below demonstrates that most of these galaxies have SNR(F814W) < 3, and many have SNR(F850LP) < 3. This makes sense in the context that the JADES F814W and F850LP photometry is simply fainter (see the second figure below); by about 1 magnitude. Thus the SNR should also be much lower, and it clears that we expect some E24 galaxies to fail this condition.

> 
><p float="left" align="middle">
>    <img src="figs/photometry_comparison/fails/f814w_f850lp_snr_gtr_3.png" width=50%/>
></p>
>
>**Figure:** The SNR of the ACS/F814W and ACS/F850LP photometry measured by JADES for the E24 F775W dropout galaxies not recovered by this work's F775W dropout selection. The dashed black line indicates SNR = 3, which is the critical threshold for the corresponding requirement on the F814W and F850LP SNR.

><p float="left" align="middle">
>   <img src="figs/photometry_comparison/endsley2024_f775w_dropouts_vs_jades_ACS_F814W.png" width=48%/>
>   <img src="figs/photometry_comparison/endsley2024_f775w_dropouts_vs_jades_ACS_F850LP.png" width=48%/>
></p>
>
> **Figure:** The ACS/F814W and ACS/F850LP photometry of the E24 F775W dropout galaxies in the E24 and JADES catalogs. In both filters, the JADES photometry is fainter by about 1 magnitude.

The sufficiently sensitive rest-optical photometry (...) requirement fails 39 of the E24 galaxies.