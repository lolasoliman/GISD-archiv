---
title: "GISD Karten auf Gemeinde-, Kreis-, ROR-Ebene"
author: "Omar Soliman"
date: "2023-11-16"
output:
  bookdown::html_document2:
    keep_md: true
    code_folding: hide
    number_sections: false
    fig_caption: true
    theme: cerulean
    highlight: tango
---



## Pakete installieren/laden

```r
##Bei Bedarf auskommentieren:
# install.packages("tidyverse")
# install.packages("sf")
# devtools::install_github("lekroll/rkicolors")

library("tidyverse")
library("sf")
library("rkicolors")
```

## GISD einlesen und mit Shapefiles matchen

```r
# Basemaps laden
#src1 <- "https://daten.gdz.bkg.bund.de/produkte/vg/vg250_ebenen_0101/2020/vg250_01-01.gk3.shape.ebenen.zip" #Bundesländer
#src2 <- "https://daten.gdz.bkg.bund.de/produkte/sonstige/ge250/2021/ge250.gk3.shape.zip"                    #Raumordnungsregionen
#src3 <- "https://daten.gdz.bkg.bund.de/produkte/vg/vg2500/2020/vg2500_01-01.gk3.shape.zip"                  #Kreise und Gemeinden

invisible(capture.output({ #Output ist stur, so wird er stumm gemacht
basemap_bula <- st_read("Data/SHP/2020/vg2500_01-01.gk3.shape/vg2500/vg2500_lan.shp")
basemap_ror <- st_read("Data/SHP/2020/ge250.gk3.shape/ge250/ror/ROR250.shp")
basemap_krs <- st_read("Data/SHP/2020/vg2500_01-01.gk3.shape/vg2500/vg2500_krs.shp")
basemap_gem <- st_read("Data/SHP/2020/vg250_01-01.gk3.shape.ebenen/vg250_ebenen_0101/VG250_GEM.shp")
}))

# Gemeindedaten laden
gisd_gem <- read_csv("Outfiles/2023_v0/Bund/GISD_Bund_Gemeinde.csv") %>% 
  filter(year == 2020) #Nur das neueste Jahr behalten

# Kreisdaten laden
gisd_krs <- read_csv("Outfiles/2023_v0/Bund/GISD_Bund_Kreis.csv") %>% 
  filter(year == 2020) #Nur das neueste Jahr behalten

gisd_krs2 <- gisd_krs %>% # Für imputation später
  mutate(kkz = as.numeric(kreis_id)) %>%
  select(kkz, gisd_score, gisd_5, gisd_10)

# ROR-Daten laden
gisd_ror <- read_csv("Outfiles/2023_v0/Bund/GISD_Bund_Raumordnungsregion.csv") %>% 
  filter(year == 2020) #Nur das neueste Jahr behalten

# Matche Basemaps mit GISD-Daten
gisd_gem <- left_join(basemap_gem, gisd_gem,
            by = c("AGS" = "gemeinde_id"))

gisd_krs <- left_join(basemap_krs, gisd_krs,
            by = c("ARS" = "kreis_id"))

gisd_ror <- left_join(basemap_ror, gisd_ror,
            by = c("SN_ROR" = "ror_id"))

# Imputiere GISD-Score für gemeindefreie Gebiete durch Score des übergeordneten Kreises
gemeindenohnescore <- st_drop_geometry(gisd_gem) %>% 
  filter(is.na(gisd_score)) %>% 
  mutate(kkz = floor((as.numeric(AGS) / 1000))) %>% #Die ersten vier Stellen der Gemeindekennziffer ist die Kreiskennziffer
  select(AGS, kkz) %>% 
  left_join(., gisd_krs2,
            by = "kkz") %>% 
  select(-kkz)

gisd_gem <- left_join(gisd_gem,
                      gemeindenohnescore,
                      by = "AGS") %>%
  mutate(gisd_score = coalesce(gisd_score.x, gisd_score.y),
         gisd_5 = coalesce(gisd_5.x, gisd_5.y),
         gisd_10 = coalesce(gisd_10.x, gisd_10.y)) %>%
  select(-ends_with(".x"),
         -ends_with(".y"))
```

## GISD Score auf Gemeindeebene

```r
credits <- "Author: Niels Michalski / Omar Soliman\n
Projection: Gauss-Krüger 3 (EPSG:31467)\n
Scale: 1:2'500'000\n
© GeoBasis-DE/BKG 2023"

ggplot() +
  geom_sf(data = gisd_gem, aes(fill = gisd_score), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score"),
                 reverse = TRUE, discrete = FALSE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/gemeindeebene-1.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_gem, aes(fill = as.factor(gisd_5)), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Quintile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/gemeindeebene-2.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_gem, aes(fill = as.factor(gisd_10)), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Dezile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/gemeindeebene-3.png)<!-- -->

## GISD Score auf Kreisebene

```r
ggplot() +
  geom_sf(data = gisd_krs, aes(fill = gisd_score), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score"),
                 reverse = TRUE, discrete = FALSE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/kreisebene-1.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_krs, aes(fill = as.factor(gisd_5)), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Quintile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/kreisebene-2.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_krs, aes(fill = as.factor(gisd_10)), color = NA)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Dezile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/kreisebene-3.png)<!-- -->

## GISD Score auf Raumordnungsregionsebene

```r
ggplot() +
  geom_sf(data = gisd_ror, aes(fill = gisd_score), color = "grey50", lwd = 0.05)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score"),
                 reverse = TRUE, discrete = FALSE,) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/rorebene-1.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_ror, aes(fill = as.factor(gisd_5)), color = "grey50", lwd = 0.05)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Quintile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/rorebene-2.png)<!-- -->

```r
ggplot() +
  geom_sf(data = gisd_ror, aes(fill = as.factor(gisd_10)), color = "grey50", lwd = 0.05)  +
  geom_sf(data = basemap_bula, fill = NA, color = "grey30", lwd = 0.1) +
  scale_fill_rki(name = c("GISD-Score (Dezile)"),
                 reverse = TRUE) +
  theme_rki_void()
```

![](Score_Karte_2020_files/figure-html/rorebene-3.png)<!-- -->
