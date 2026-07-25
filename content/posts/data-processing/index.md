---
title: "LiangJingCe - Development NotesData : Data Sources"
date: 2026-04-28
draft: false
tags: ["Data", "DEM", "OSM"]
summary: "The following table summarises the datasets employed in the study, including their specifications, analytical functions, and spatial coverage."
---

## Data Sources
The following table summarises the datasets employed in the study, including their specifications, analytical functions, and spatial coverage.
Topographic elevation (DEM): 30-metre resolution DEM. Used to derive slope, aspect, and cost distance. Coverage: Xi’an (Chang’an) and its surrounding area; Luoyang and its surrounding area.
Water system vectors: Line and polygon vector data for rivers and lakes. Used to calculate distance to water bodies and for buffer analysis. Coverage: same as above.
Administrative boundaries: Modern city and county boundaries (optional). Used to clip the study area and provide a cartographic background. Coverage: Shaanxi Province, Henan Province.
Historical site points: Locations of Tang-dynasty granaries and city coordinates. Used for model validation and result comparison. Coverage: vicinity of Chang’an and Luoyang.
## Detailed descriptions
Detailed descriptions of each dataset are given below.
1.	DEM (topographic elevation data)
Data source: Geospatial Data Cloud 
URL:(http://www.gscloud.cn/)
Resolution: 30 m
2.	Water system, road network, and other vector data (OSM)
The river and road network vector data were downloaded from OpenStreetMap (OSM), an open geospatial database maintained by a global community of volunteers and widely used in geographical and urban planning research worldwide. The data were obtained from the official OSM distribution platform GeoFabrik (download.geofabrik.de) in Shapefile format, clipped to the coordinates of the study area, and subjected to quality control.
URL: https://download.geofabrik.de/asia/china.html
3.	Historical site data
Data sources: historical documents including Tang Huiyao (Institutional History of the Tang), Yuanhe Junxian Tuzhi (Maps and Records of Prefectures and Counties in the Yuanhe Period), and Jiu Tangshu (Old Book of Tang).
Granary sites:
Chang’an Taicang (approx. present-day Xi’an city centre)
Dongweiqiao Granary (northeast of Xi’an, on the north bank of the Wei River)
Luoyang Hanjiacang Granary (north of the old city of Luoyang)
Luoyang Huiluo Granary (east of the Sui-Tang city of Luoyang)
Coordinates were recorded in a CSV file.
