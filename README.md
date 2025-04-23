# Spatial Modelling of Deer in BC

**DATA 589 Project**  
**Team:** Shiqi Zhang, Jiakun Li, Ernest Wong, Zefeng Pei  

### See our project presentation: [Google Slides](https://docs.google.com/presentation/d/1zm0EeMhrbMxSbx7flz2MLp8R3iqKiiuxFAKzXRSirMU/edit?usp=sharing)

<img src="https://github.com/user-attachments/assets/bd12e3e8-5b7e-4d70-a7c7-7935a9f8ad85" alt="Description" width="400"/>

---

## Overview

This project aims to model and understand how environmental covariates — including forest cover, elevation, and human population — influence the spatial distribution of deer in British Columbia. Our approach uses spatial point process models and generalized additive models (GAMs) to study intensity variation across the region.

### Research Questions

- How do forest cover, elevation, and human population affect deer presence?
- Can we build an accurate spatial point process model to explain deer intensity?

---

## Data Sources

- **Deer Occurrence Data (11,488 points)**: GBIF.org (2024).  
  [GBIF Download Link](https://www.gbif.org/occurrence/download?has_coordinate=true&has_geospatial_issue=false&taxon_key=2440974&gadm_gid=CAN.2_1)

- **Forest Cover and Elevation**: Extracted from raster sources via the BC Parks dataset.
- **Population Data**:  
  Bondarenko et al. (2020), [DOI:10.5258/SOTON/WP00684](https://doi.org/10.5258/SOTON/WP00684)

---

## Repository Structure

| File | Description |
|------|-------------|
| `Multivariate_Model.Rmd` | Main multivariate model script and report |
| `Multivariate_Model.html` | Rendered output with visuals |
| `Forest_vs_Deer.Rmd` / `.html` | Univariate Poisson model: Forest vs. Deer |
| `Elevation_vs_Deer.qmd` / `.html` | Univariate Poisson model: Elevation vs. Deer |
| `Population_vs_Deer.qmd` / `.html` | Univariate Poisson model: Population vs. Deer |
| `data/deer_data.Rdata` | Processed dataset including covariates and deer ppp |

To load the required data:
```r
load("data/deer_data.Rdata")
```

---

## Setup and Required Packages

```r
# Core spatial analysis
install.packages("spatstat.core")
install.packages("spatstat.geom")
install.packages("spatstat")
install.packages("spatstat.data")

# Raster and vector handling
install.packages("terra")
install.packages("raster")
install.packages("sf")

# Modeling
install.packages("mgcv")

# General tools
install.packages("tidyverse")
install.packages("viridis")

# Optional spatial support
install.packages("sp")
```

---

## Data Wrangling Workflow
![workflow](https://github.com/user-attachments/assets/15c99f2b-5773-4b7a-a17a-b6e08bb42245)

1. **Spatial Smoothing**  
   Applied kernel smoothing to the population raster using a Gaussian kernel:
   ```r
   pop_smoothed <- Smooth(pop_im, sigma = 2500)
   ```

2. **Handling Missing or Zero Values**  
   Replaced NA or small values with a positive floor:
   ```r
   pop_smoothed$v[is.na(pop_smoothed$v) | pop_smoothed$v <= 0] <- 1e-6
   ```

3. **Masking by Study Region**  
   Trimmed covariates to the spatial extent of the BC study window:
   ```r
   forest_masked <- forest_im[win_bc, drop=FALSE]
   elev_masked   <- elevation_im[win_bc, drop=FALSE]
   pop_masked    <- pop_smoothed[win_bc, drop=FALSE]
   ```

4. **Log Transformation**  
   Applied log transform to population after masking:
   ```r
   pop_log <- eval.im(log(pop_masked + 1e-3))
   ```

---

## Modelling Approach

We adopted a **stepwise modeling strategy**:

- **Univariate Poisson Models**  
  Explored each covariate independently to understand its isolated spatial effect.

- **Multivariate Poisson Model (Polynomial)**  
  Jointly modeled all covariates with polynomial terms.  
  Detected residual spatial autocorrelation → motivated GAM.

- **GAM (Generalized Additive Model)**  
  Final model using smooth terms for forest, elevation, and population.

---

## Key Visualizations

- Map: Observed deer points
![deer](https://github.com/user-attachments/assets/141bd635-704a-4e00-a545-3c76e08ad933)

- Covariate maps: forest, elevation, population
  
![forest](https://github.com/user-attachments/assets/6403a44f-766f-4ec3-bc26-c859cf787629)
![elev](https://github.com/user-attachments/assets/b2c45fb5-d507-400a-950e-21704bc9c535)
![pop](https://github.com/user-attachments/assets/90d280e6-1b06-4692-b52c-d95edc15aee3)


- Predicted intensity maps (Poisson vs. GAM)
![poisson intensity](https://github.com/user-attachments/assets/ad8f4328-9bc0-4e94-8b81-0cf3a175bfc5)
![gam intensity](https://github.com/user-attachments/assets/d74e23a3-380b-471b-80fe-450b43da8bfa)

- Spatial autocorrelation (variogram)
![variogram](https://github.com/user-attachments/assets/6fb74326-9c48-4dd3-9286-bb474f25da2e)

---

## Model Comparison Summary

| Metric | Polynomial Poisson (log pop) | GAM |
|--------|------------------------------|-----|
| AIC | 346,672 | −71,014,078 |
| Deviance Explained | — | 75.3% |
| Residual Moran’s I | 1.0 (high) | Lower (visually) |
| Interpretability | High | Moderate |
| Spatial Fit Quality | Moderate | High |

---

## Conclusions

- **Forest, elevation, and population all play significant roles** in explaining deer intensity.
- The **GAM achieved superior spatial fit**, explaining 75% of deviance and reducing residual clustering.
- **Poisson models remain valuable** for interpretation and spatial process grounding.
- Our **modeling progression from univariate → multivariate → GAM** was key to identifying the appropriate model complexity.

---

## Citation and References

- GBIF.org (2024). *Deer occurrence data (Odocoileus virginianus) in British Columbia*.  
- Bondarenko, M., Kerr, D., Sorichetta, A., & Tatem, A. J. (2020).  
  *WorldPop, University of Southampton.*  
  [DOI:10.5258/SOTON/WP00684](https://doi.org/10.5258/SOTON/WP00684)

