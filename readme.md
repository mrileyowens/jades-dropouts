Things to do:

- Why does the final selection for sufficiently sensitive rest-optical colors drop such a large number of objects from the southwest region of GOODS-S?
- Compare the photometry of my JADES selection with that of Endsley et al. (2024)
- Double check aperture characteristics of the convolved JADES KRON_S photometry compared to Endsley et al. (2024)

# Description of the code

This repository attempts to replicate the selection of F775W dropout galaxies described by Endsley et al. (2024) (herafter E24) in the GOODS-N and GOODS-S fields by using the JADES photometric catalog.

The notebook `select.ipynb` performs the actual selection from the photometry, first in `drop()`, which makes all but the last selection from E24. I also added two adjustments, compared to the stated methodology of E24: 

1. **Reassign negative flux densities in the ACS F606W and F775W filters.** E24 adopted the $1\sigma$ upper limit for low-SNR photometry in Lyman break-shortward filters used to calculate colors (namely ACS F606W and F775W), which is a common practice. When repeating that for the JADES catalog, some objects still have negative flux densities, which means the corresponding magnitude (and any dependent colors) are undefined. This felt like a somewhat arbitrary data artifact, particularly for objects that seemed unfairly disqualified over this. So, for objects which still had negative flux densities after adopting the $1\sigma$ upper limit, I assigned a tiny, positive flux density to those filters: $10^{-3}$ nJy. This should still capture what I think is the point here: that these objects have flux densities basically consistent with 0 in these filters.

2. **Discard objects with undefined uncertainties.** Some objects have photometry with undefined uncertainties. I don't think it's appropriate to include those objects when one of the relevant filters to the selection criteria has an undefined uncertainty, so I also enforced that a handful of the most important filters must have a finite uncertainty.

I also made use of the JADES catalog's flags to remove foreground stars and objects contaminated by bright stars or other neighbors.

After making the initial selection, `drop()` saves the resulting catalog. This is necessary because the final selection requires making SED fits to estimate $f_\text{1500}$, and it would be cost prohibitive to do so for the entire JADES catalog before paring it down. E24 performed this SED fitting with BEAGLE, but I chose to use Bagpipes instead, since it is much quicker, and for the purposes of calculating $f_\text{1500}$, all that is necessary is to accurately reproduce the overall SED shape, and not necessarily accurate masses, metallicities, etc.

The function `fit()` contains the Bagpipes fitting, which uses an identical filter set to E24, even though more filters are available in the JADES catalog. Using the resulting Bagpipes fits, `fit()` makes the final selection of E24 and saves the remaining objects to a new catalog.

The remaining code analyzes the results of this selection.

# Why aren't the two selections identical?

## Different photometry

### Comparing photometry of the Endsley et al. (2024) galaxies with the corresponding photometry in JADES

One insightful exercise to explore differences between JADES photometry and that of Endsley et al. (2024) is to directly compare the two for the same set of objects. I made this comparison with the function `compare_catalog_endsley2024()` in `compare_phot.ipynb` by plotting the two sets of photometry against each other. 

Of the 278 F775W dropouts in Endsley et al. (2024), 268 have a corresponding source in the JADES catalog (i.e., an object within 0.1 arcsec of a coordinate in the other catalog). Summarizing the results of those figures:

- **Photometry.** The width of the photometric distributions may be slightly wider in the JADES catalog. But more importantly, **every** filter's photometry in JADES is $\sim1$ mag fainter.
- **SNR**. The SNR in the F435W and F606W filters appear consistent, and the SNR in the F410M and F444W filters may be a few higher in the JADES catalog. Otherwise, the SNR may be a few higher in the Endsley et al. (2024) catalog, though the discrepancy becomes much stronger at the high-SNR end.
- **Colors.** The JADES catalog seems to prefer redder F775W - F090W and F606W - F090W colors, but the F090W - F150W color is extremely consistent between the two catalogs. It may be notable that the two colors involving HST filters are discrepant, but not the color with just JWST filters. Though it may also be downstream of the fact that the F775W and F606W filters are shortward of the Lyman break at the targeted redshift.

### Comparing photometry of the Endsley et al. (2024) galaxies based on membership in the JADES dropout sample

Using `compare_endsley2024()` in `compare_phot.ipynb`, I compared the photometry of the F775W dropout catalog based on membership in my own JADES dropout sample. Of the 278 galaxies, 114 are also in my JADES dropout sample, and they tend to be brighter, higher SNR, and possibly have redder colors than those that are not also in my JADES dropout sample.

### Comparing photometry of the JADES dropout sample galaxies based on membership in the Endsley et al. (2024) dropout sample

The function `compare_jades()` in `compare_phot.ipynb` compares the photometry of the F775W dropout sample I made from the JADES photometric catalog, based on membership in Endsley et al. (2024)'s F775W dropout catalog. Of the 290 galaxies, 114 are also in the Endsley et al. (2024) sample, and 176 are not. Those that are also in the other sample seemed to trend slightly toward being brighter (tenths of a magnitude, probably) in Lyman break longward filters, slightly fainter in Lyman break shortward filters, higher SNR, and slightly redder colors.

### Open questions

- Why are 10 of the F775W dropouts of Endsley et al. (2024) missing in the JADES catalog?
- What are the reasons why some of the F775W dropouts from Endsley et al. (2024) do not have a well-defined color? And how important is each?