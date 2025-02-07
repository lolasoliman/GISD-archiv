---
title: "Karte des GISD Scores"
author: "Marvin Reis"
date: "20 01 2022"
output:
  bookdown::html_document2:
    keep_md: true
    code_folding: hide
    number_sections: false
    fig_caption: true
    theme: cerulean
    highlight: tango
---




```r
GISD_data_Kreis <- read.csv("Outfiles/2022/Bund/Kreis/Kreis.csv") %>% mutate(Kreis = Kreiskennziffer) %>% select(Kreis, GISD_Score, GISD_5, GISD_10) %>% distinct(Kreis, .keep_all = TRUE) %>% unique() %>% lazy_dt()

Kreise_data <- readRDS("Data/SHP/kreise_bkg.rds") %>% lazy_dt() %>% mutate(Kreis = as.numeric(id)) %>% select(-id) %>% left_join(GISD_data_Kreis, by = "Kreis") %>% lazy_dt()

Kreise_data <- as_tibble(Kreise_data)

GISD_data_Gem <- read.csv("Outfiles/2022/Bund/Gemeinde/Gemeinde.csv") %>% select(Gemeindekennziffer, GISD_Score, GISD_5, GISD_10) %>% distinct(Gemeindekennziffer, .keep_all = TRUE) %>% unique() %>% lazy_dt()

Gemeinden_data <- readRDS("Data/SHP/BRD_Gemeinden.rds") %>% lazy_dt() %>% mutate(Gemeindekennziffer = as.numeric(id)) %>% select(-id) %>% left_join(GISD_data_Gem, by = "Gemeindekennziffer") %>% lazy_dt()

Gemeinden_data <- Gemeinden_data %>% mutate(Kreis = floor(Gemeindekennziffer / 1000)) %>% left_join(GISD_data_Kreis, by = "Kreis")

Gemeinden_data <- as_tibble(Gemeinden_data)

Gemeinden_data <- Gemeinden_data %>% mutate(GISD_Score = ifelse(is.na(GISD_Score.x) == TRUE, GISD_Score.y, GISD_Score.x), GISD_5 = ifelse(is.na(GISD_5.x) == TRUE, GISD_5.y, GISD_5.x), GISD_10 = ifelse(is.na(GISD_10.x) == TRUE, GISD_10.y, GISD_10.x)) %>% select(-GISD_Score.x, -GISD_Score.y, -GISD_5.x, -GISD_5.y, -GISD_10.x, -GISD_10.y)

GISD_data_Lander <- read.csv("Outfiles/2022/Bund/Raumordnungsregion/Raumordnungsregion.csv") %>% mutate(ROR_id = Raumordnungsregion.Nr) %>%  select(ROR_id, GISD_Score, GISD_5, GISD_10) %>% distinct(ROR_id, .keep_all = TRUE) %>% unique() %>% lazy_dt()

Lander_data <- readRDS("Data/SHP/ROR_map.rds") %>% lazy_dt() %>% mutate(ROR_id = as.numeric(id)) %>% select(-id) %>% left_join(GISD_data_Lander, by = "ROR_id") %>% lazy_dt()

Lander_data <- as_tibble(Lander_data)

BuLa_data <- readRDS("Data/SHP/BRD_BuLa.rds") %>% rename(long_bula = long, lat_bula = lat, group_bula = group) %>% mutate(Gemeindekennziffer = as.numeric(id)) %>% select(Gemeindekennziffer, long_bula, lat_bula, group_bula)

Gemeinden_data <- Gemeinden_data %>% left_join(BuLa_data, by = "Gemeindekennziffer")
```


## GISD-Score auf Gemeindeebene

```r
ggplot(Gemeinden_data) +
  geom_polygon(aes(long, lat, group = group, fill = GISD_Score)) +
  scale_fill_rki(palette = "main", name = "GISD-Score", discrete = FALSE) +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
#which(is.na(Gemeinden_data$GISD_Score))
```


```r
ggplot(Gemeinden_data, aes(long, lat, group = group, fill = as.factor(GISD_5))) +
  geom_polygon() +
  scale_fill_rki(palette = "main", name = "GISD-Score (Quintile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-3-1.png)<!-- -->


```r
ggplot(Gemeinden_data, aes(long, lat, group = group, fill = as.factor(GISD_10))) +
  geom_polygon() +
  scale_fill_rki(palette = "main", name = "GISD-Score (Dezile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## GISD-Score auf Kreisebene

```r
ggplot(Kreise_data, aes(long, lat, group = group, fill = GISD_Score)) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score", discrete = FALSE) +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


```r
ggplot(Kreise_data, aes(long, lat, group = group, fill = as.factor(GISD_5))) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score (Quintile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


```r
ggplot(Kreise_data, aes(long, lat, group = group, fill = as.factor(GISD_10))) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score (Dezile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

## GISD-Score nach Raumordnungsregion

```r
ggplot(Lander_data, aes(long, lat, group = group, fill = GISD_Score)) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score", discrete = FALSE) +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


```r
ggplot(Lander_data, aes(long, lat, group = group, fill = as.factor(GISD_5))) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score (Quintile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-9-1.png)<!-- -->


```r
ggplot(Lander_data, aes(long, lat, group = group, fill = as.factor(GISD_10))) +
  geom_polygon(color = "black") +
  scale_fill_rki(palette = "main", name = "GISD-Score (Dezile)") +
  coord_equal() +
  theme_rki_void()
```

![](Score_Karte_2019_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
