# Using machine learning to improve free topography data for flood modelling

As part of the requirements for the [Master of Disaster Risk & Resilience](https://www.canterbury.ac.nz/study/qualifications-and-courses/masters-degrees/master-of-disaster-risk-and-resilience/) programme at the [University of Canterbury](https://www.canterbury.ac.nz/), this research project explored the potential for machine learning models to make free Digital Surface Models (such as the widely-used SRTM) more applicable for flood modelling, by stripping away vertical biases relating to vegetation & built-up areas to get a "bare earth" Digital Terrain Model.

The image below visualises the performance of one of these models (a fully-convolutional neural network) in one of the three test zones considered (i.e. data unseen during model training & validation, used to assess the model's ability to generalise to new locations). A more detailed description is provided in the associated open-access journal article: [Meadows & Wilson 2021](https://www.mdpi.com/2072-4292/13/2/275).  

![graphical_abstract](/images/graphical_abstract_boxplots.png)
<br/>

## Python scripts

All Python code fragments used during this research are shared here (covering preparing input data, building & training three different ML models, and visualising the results), in the hope that they'll be useful for others doing related work or extending/improving this approach. Please note this code includes lots of exploratory steps & some dead ends, and is not a refined step-by-step template for applying this approach in a new location.

Scripts are stored in folders relating to the virtual environments within which they were run, along with a text file summarising all packages loaded in each environment:

- [geo](/scripts/geo/): geospatial processing & mapping
- [sklearn](/scripts/sklearn/): development of Random Forest model
- [tf2](/scripts/tf2/): development of neural network models
- [osm](/scripts/osm/): downloading OpenStreetMap data
<br/>

## Brief summary of datasets used

The data processed for use in this project comprised the feature data (free, global datasets relevant to the vertical bias in DSMs, to be used as inputs to the machine learning models), target data (the reference "bare earth" DTM from which the models learn to predict vertical bias), and some supplementary datasets (not essential to the modelling but used to explore/understand the results).  

### Feature data

A guiding principle for the project was that all feature (input) data should be available for free and with global (or near-global) coverage, so as to maximise applicability in low-income countries/contexts. While these datasets were too big to store here, all can be downloaded for free and relatively easily (some require signing up to the provider platform) based on the notes below.

**Digital Surface Models (DSMs)**
- SRTM: Downloaded from [EarthExplorer](https://earthexplorer.usgs.gov/) under *Digital Elevation > SRTM > SRTM 1 Arc-Second Global*
- ASTER: Downloaded from [EarthData Search](https://search.earthdata.nasa.gov/search) ("ASTER Global Digital Elevation Model V003")
- AW3D30: Downloaded from [Earth Observation Research Centre](https://www.eorc.jaxa.jp/ALOS/en/aw3d30/) (Version 2.2, the latest available at the time)

**Multi-spectral imagery**
- Landsat-7: Downloaded from [EarthExplorer](https://earthexplorer.usgs.gov/) under *Landsat > Landsat Collection 1 Level-2 (On-Demand) > Landsat 7 ETM+ C1 Level-2* (surface reflectance bands) and *Landsat > Landsat Collection 1 Level-1 > Landsat 7 ETM+ C1 Level-1* (thermal & panchromatic bands), limited to the *Tier 1* collection only and a 6-month period centred around the SRTM data collection period (11-22 Feb 2000)
- Landsat-8: Downloaded from [EarthExplorer](https://earthexplorer.usgs.gov/) under *Landsat -> Landsat Collection 1 Level-2 (On-Demand) -> Landsat 8 OLI/TIRS C1 Level-2* (surface reflectance bands) and *Landsat -> Landsat Collection 1 Level-1 -> Landsat 8 OLI/TIRS C1 Level-1* (thermal & panchromatic bands), limited to the *Tier 1* collection only and 6-month periods centred around each of the LiDAR survey dates (in 2016, 2017 & 2018)

**Night-time light**
- DMSP-OLS Nighttime Lights Time Series (annual composites): Downloaded from the NOAA [Earth Observation Group](https://ngdc.noaa.gov/eog/dmsp/downloadV4composites.html)
- VIIRS Day/Night Band Nighttime Lights (monthly composites): Downloaded from the CSM [Earth Observation Group](https://eogdata.mines.edu/download_dnb_composites.html)

**Others**
- Global forest canopy height: Developed by [Simard et al. 2011](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2011JG001708) and available for download [here](https://landscape.jpl.nasa.gov/)
- Global forest cover: Developed by [Hansen et al. 2013](https://science.sciencemag.org/content/342/6160/850) and available for download [here](https://earthenginepartners.appspot.com/science-2013-global-forest/download_v1.6.html)
- Global surface water: Developed by [Pekel et al. 2016](https://www.nature.com/articles/nature20584) and available for download [here](https://global-surface-water.appspot.com/download)
- OpenStreetMap layers: Downloaded using the [OSMnx](https://github.com/gboeing/osmnx) Python module developed by [Boeing 2017](https://www.sciencedirect.com/science/article/pii/S0198971516303970)  
  
### Target data

In order to learn how to predict (and then correct) the vertical biases present in DSMs, the models need reference data - "bare earth" DTMs assumed to be the "ground truth" that we're aiming for. For this project, we used three of the high-resolution LiDAR-derived DTMs published online by the New Zealand Government, accessible to all via the Land Information New Zealand (LINZ) [Data Service](https://data.linz.govt.nz/). The specific LiDAR surveys used are summarised below, from the Marlborough & Tasman Districts (in the north of Aotearoa New Zealand's South Island):

- Marlborough (May-Sep 2018): [DTM](https://data.linz.govt.nz/layer/103535-marlborough-lidar-1m-dem-2018/) and corresponding [index tiles](https://data.linz.govt.nz/layer/103538-marlborough-lidar-index-tiles-2018/)
- Tasman - Golden Bay (Nov-Dec 2017): [DTM](https://data.linz.govt.nz/layer/95503-tasman-golden-bay-lidar-1m-dem-2017/) and corresponding [index tiles](https://data.linz.govt.nz/layer/95627-goldenbaytilelayout/)
- Tasman - Abel Tasman & Golden Bay (Dec 2016): [DTM](https://data.linz.govt.nz/layer/95578-tasman-abel-tasman-and-golden-bay-lidar-1m-dem-2016/) and corresponding [index tiles](https://data.linz.govt.nz/layer/95581-tasman-abel-tasman-and-golden-bay-lidar-index-tiles-2016/)

To find similar target/reference DTM data in other parts of the world, the [OpenTopography](https://opentopography.org/) initiative maintains a catalogue of freely available sources.  
  
### Supplementary data

A few other datasets are referred to in the code, not as inputs to the machine learning models but just as references to better understand the results.

- MERIT DSM: Improved DSM developed by [Yamazaki et al. 2017](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2017GL072874), with a request form for the data available [here](http://hydro.iis.u-tokyo.ac.jp/~yamadai/MERIT_DEM/)
- New Zealand Land Cover Database (LCDB v5.0): Developed by [Manaaki Whenua - Landcare Research](https://www.landcareresearch.co.nz/), available from the [LRIS portal](https://lris.scinfo.org.nz/)
<br/>

## Brief summary of approach taken

The broad approach taken is summarised below as succinctly as possible, with further details provided as comments in the relevant scripts.

1. For each available LiDAR survey zone, process the DSMs and DTM in tandem: clipping each DSM ([SRTM](/scripts/geo/geo_process_LiDAR_SRTM.py), [ASTER](/scripts/geo/geo_process_ASTER.py) and [AW3D30](/scripts/geo/geo_process_AW3D30.py)) to the extent covered by the LiDAR survey, and resampling the [DTM](/scripts/geo/geo_process_LiDAR_SRTM.py) to the same resolution & grid alignment as each DSM. Various DSM derivatives (such as slope, aspect & topographical index products) are also prepared here.  

2. Based on a comparison of differences between each DSM and the DTM (resampled to match that particular DSM), the SRTM DSM was selected as the "base" for all further processing ([script](/scripts/geo/geo_visualise_DSMs.py)).  

3. Process all other input datasets - resampling to match the SRTM resolution & grid alignment, masking out clouds for the multi-spectral imagery, applying bounds where appropriate (e.g. for percentage variables):  
    - Landsat-7 multi-spectral imagery ([script](/scripts/geo/geo_process_Landsat7.py))
    - Landsat-8 multi-spectral imagery ([script](/scripts/geo/geo_process_Landsat8.py))
    - ASTER DEM ([script](/scripts/geo/geo_process_ASTER.py))
    - AW3D30 DEM ([script](/scripts/geo/geo_process_AW3D30.py))
    - Night-time light ([script](/scripts/geo/geo_process_NTL.py))
    - Global forest canopy height ([script](/scripts/geo/geo_process_GCH.py))
    - Global forest cover ([script](/scripts/geo/geo_process_GFC.py))
    - Global surface water ([script](/scripts/geo/geo_process_GSW.py))
    - OpenStreetMap layers ([script](/scripts/geo/geo_process_OSM.py))  

4. Divide all available data into training (90%), validation (5%) and testing (5%) subsets, and prepare for input to the pixel-based approaches (random forest & standard neural network) and patch-based approach (convolutional neural network) ([script](/scripts/geo/geo_process_ML_inputs.py)).  

5. Use step floating forward selection (SFFS) (with a random forest estimator) to select relevant features based on the training & validation datasets ([script](/scripts/sklearn/sklearn_random_forest.py))  

6. Train the random forest model, tuning hyperparameters with reference to the validation data subset ([script](/scripts/sklearn/sklearn_random_forest.py))  

7. Train the densely-connected neural network model, tuning hyperparameters with reference to the validation data subset ([script](/scripts/tf2/tf2_densenet.py))  

8. Train the fully-convolutional neural network model, tuning hyperparameters with reference to the validation data subset ([script](/scripts/tf2/tf2_convnet.py))  

9. Visualise results for the three zones of the testing data subset (unseen during model development) ([script](/scripts/geo/geo_visualise_results.py))  
