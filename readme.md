Things to do:

- Why does the final selection for sufficiently sensitive rest-optical colors drop such a large number of objects from the southwest region of GOODS-S?
- Compare the photometry of my JADES selection with that of Endsley et al. (2024)
- Double check aperture characteristics of the convolved JADES KRON_S photometry compared to Endsley et al. (2024)

# Why aren't the two selections identical?

## Different photometry

One insightful exercise to explore differences between JADES photometry and that of Endsley et al. (2024) is to directly compare the two for the same set of objects. I made this comparison with the function `compare_catalog_endsley2024()` in `compare_phot.ipynb` by plotting the two sets of photometry against each other. 

Of the 278 F775W dropouts in Endsley et al. (2024), 268 have a corresponding source in the JADES catalog (i.e., an object within 0.1 arcsec of a coordinate in the other catalog). Summarizing the results of those figures:

- **Photometry.** The width of the photometric distributions may be slightly wider in the JADES catalog. But more importantly, **every** filter's photometry in JADES is $\sim1$ mag fainter.
- **SNR**. The SNR in the F435W and F606W filters appear consistent, and the SNR in the F410M and F444W filters may be a few higher in the JADES catalog. Otherwise, the SNR may be a few higher in the Endsley et al. (2024) catalog, though the discrepancy becomes much stronger at the high-SNR end.
- **Colors.** The JADES catalog seems to prefer redder F775W - F090W and F606W - F090W colors, but the F090W - F150W color is extremely consistent between the two catalogs. It may be notable that the two colors involving HST filters are discrepant, but not the color with just JWST filters. Though it may also be downstream of the fact that the F775W and F606W filters are shortward of the Lyman break at the targeted redshift.

### Open questions

- Why are 10 of the F775W dropouts of Endsley et al. (2024) missing in the JADES catalog?
- What are the reasons why some of the F775W dropouts from Endsley et al. (2024) do not have a well-defined color? And how important is each?