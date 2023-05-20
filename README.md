# City Trees: Seattle
## Predicting the Health of Seattle's Urban Tree Cover
![Photo of downtown Seattle framed by trees with Mt. Hood in the background.](./resources/readme_photos/seattle-cover.png)
\
\
Trees play important roles in urban areas by improving air quality, regulating temperature, limiting stormwater damage, reducing noise pollution, and promoting biodiversity. They can also add beauty and provide space for community gatherings in areas often surrounded by seas of concrete.
\
\
Maintaining what researchers call our "urban tree canopy" is very important for the health of our urban communities. As we are facing climate change, the question becomes: what environmental factors impact the health of our urban tree canopy? And can we predict what that health will be in years to come?

## Data
Our primary data source comes from DataDryad and was published by a group of university students (citation below). It has over 5 million records accross the U.S., but I trimmed down to focus on Seattle, Washington. I also pulled in climate data from NOAA and a crowd-sourced website for Washington precipitation to get temperature, rainfall, and snowfall for weather stations in the greater Seattle metropolitan area.
> * [DataDryad.org](https://datadryad.org/stash/dataset/doi:10.5061/dryad.2jm63xsrf)
> * [NOAA Climate Normals](https://www.ncei.noaa.gov/data/normals-annualseasonal/2006-2020/)
> * [CoCoRaHS Crowd-Sourced Precipitation](https://merbgai.cocorahs.org/ViewData/TotalPrecipSummary.aspx)

## Data Cleaning
Our second climate source came from CoCoRaHS, which standard for the Community Collaborative Rain, Hail, & Snow Network. It is a crowd-sourced website for reporting daily data, with varying levels of completeness. This wasn't my ideal source but created a great opportunity to learn how to work with incomplete data.
\
\
I broke this down into a few steps, first calculating the number of reports, applying an adjustment factor for the possibility of non- reported days having rain or not, and filling missing values with daily averages from close weather stations. I was then able to map in climate data to each tree based on the nearest station from which the data was sourced.
\
![Mapping individual trees to their nearest weather station.](./resources/readme_photos/trees-mapped.png)

## EDA & Outliers
Our tree diameter field had me worried about outliers due to it's wide right tail so I took a look at the boxplot of all the trees and then drilled into some details.
\
\
**Boxplot of Diameters - All Trees**
\
![Boxplot of Diameter - All Trees showing a small IQR and many outliers.](./resources/readme_photos/all-trees-box.png)
\
\
**Boxplot of Diameters - Eddie's Dogwood**
\
![Boxplot of Diameter - Eddie's Dogwood showing a 0 IQR and many outliers.](./resources/readme_photos/eddies-dogwood-box.png)
\
\
The aggregate boxplot gave me a more powerful representation of our tailed data. But it was getting into the detail by species that I realized many records had identical diameters. It's possible the large scale of the data required some generalization/binning over precise measurements.
I like to use a 1.5X IQR definition for flagging outliers, but this binning made many of the IQRs equal to 0. To address this, I used the 1.5X IQR flag if for IQRs > 0 and > 3 standard deviations from the mean for IQRs = 0. I then dropped what this function flagged as an outlier, dropping 4,076 records (~2.5% of records).

### Correlation Matrix
To close out my EDA, I plotted a correlation matrix. Unfortunately I did not find an exciting correlation. Instead, just the covariance from some of the climate data I pulled into the analysis. I dropped the duplicative values, leaving me with a final tally of 158,004 rows and 11 columns.
\
\
![Corrleation Matrix showing no much correlation, but some covariance that needed to be addressed.](./resources/readme_photos/corr-matrix.png)

## Preprocessing & Training
Having this many records proved to be slow when fitting models, so I sampled down to 10,000 records to start. After some trial and error I settled to a few consistent pre-processing steps:
- SimpleImputer using median values for numerical features 
- SimpleImputer using 'missing' for categoricalfeatures 
- StandardScaler on my tree age and climate fields (more normally distributed)
- PowerTransformer on my other numerical fields (more tailed distribution)
- OneHotEncoder on my categorical values, ignoring unknowns
\
\
The categorical encoding became my biggest challenge to transform only my two categorical features, but still scale to new data. I ended up using a custom process using OneHotEncoder, but saving the results to a new dataframe. It works well for the
project but will need some tweaking before scaling more.

### Balancing Data


### Picking the Right Model
After doing some practice training with a logistic regression model, I created a loop that tested cookie cutters of a few different classification models to see which had the most promising results.

### Citations
McCoy, Dakota et al. (2022), A dataset of 5 million city trees from 63 US cities: species, location, nativity status, health, and more., Dryad, Dataset, https://doi.org/10.5061/dryad.2jm63xsrf
