# Info4forest: Climate Areas Inspired by the Quitt Classification

## Description of the Visualisation and Calculation Methodology

**Document version:** 3.0  
**Revision date:** 2026-07-19  
**Purpose of the document:** a substantive description of the climate areas shown in the Info4forest application, an explanation of the displayed results, and a transparent methodology for their calculation.  
**Methodological basis:** the Quitt climate classification and its analysis in the publication issued by Palacký University Olomouc and the Czech Hydrometeorological Institute [1].  

---
## Contents

1. What Climate Areas Inspired by the Quitt Classification Are
2. What the Application Displays
3. How to Read the Main Results
4. Climate Data Sources and Monitored Locations
5. Relationship to the Quitt Classification
6. The Eleven Climate Characteristics Used
7. Overview of Climate Areas and Units
8. Methodology for Calculating the Climate Unit
9. Trend Methodology
10. Interpretation, Uncertainty, and Limitations
11. The API as the Data and Calculation Backend
12. Recommended Citation
13. Sources
14. Appendix A – Limit Intervals Used by the Implementation

---

## 1. What Climate Areas Inspired by the Quitt Classification Are

The **WF5 – Forest-related Climate Area Map** tab in the Info4forest application converts climate data into an understandable map of climate areas and related characteristics. The user selects a climate data source, location, and time period. The application then shows which type of climate each place most closely resembles during the selected period.

The result is not based on a single variable such as mean temperature. Eleven temperature- and precipitation-related characteristics are evaluated together for each location. These include the numbers of summer, frost, and ice days; temperatures in four representative months; the number of precipitation days; and seasonal precipitation totals.

In simplified terms, the calculation can be understood as **comparing the climate profile of a location with 23 known climate units**. The application selects the unit whose intervals most closely match the location’s profile. The units form a scale from the coldest type, `CH1`, to the warmest type, `T5`.

> **Brief explanation for application users:** Climate areas inspired by the Quitt classification show which type of climate the selected location most closely resembles in the chosen period. The calculation compares temperature, precipitation, and other climate indicators with 23 units ranging from cold to warm areas and displays the closest match.

The calculation uses 11 characteristics available in the Info4forest data system. It is an algorithmic and reproducible implementation inspired by the Quitt scheme; it is not a simple redrawing of the historical Quitt map.

---

## 2. What the Application Displays

### 2.1 Selection of Climate Data Source, Location, and Period

The application allows the user to select:

- a **climate data source**, such as the historical ERA5-Land reanalysis or a future climate model;
- a **location** for which the source is available;
- the **start and end year** of the analysed period.

The lay name of the source is intended for general users, for example, “Historical climate / ERA5-Land model / 10 × 10 km”. The technical name available in the information tooltip specifies the exact model, regional model, and scenario, for example, `EC-Earth / REMO2020 / SSP3-7.0` [T4].

### 2.2 Climate Area Map

The main map displays the best-matching climate unit for each mapped location. The colour scale runs from cold units through moderately warm units to warm units. The map provides a spatial overview and makes it possible to identify differences between lowlands, middle elevations, and mountain areas.

After clicking the map, details are displayed for the selected point:

- the climate unit and its verbal description;
- the internal Czech code, for example `T2`, and the English/atlas code, for example `W2`;
- the match score;
- the mean values of the eleven climate characteristics;
- the climate-unit trend and its statistical significance;
- the worst continuous adverse deviation from the trend line;
- trends in the individual climate characteristics.

### 2.3 Climate Area Trend Map

The **Climate Zone Trends** map shows the estimated shift along the climate-unit scale during the selected period. The visualised variable `estimated_total_change` expresses the estimated total change:

- a positive value indicates a shift towards warmer climate units;
- a negative value indicates a shift towards colder units;
- a value close to zero indicates little or no systematic shift.

This is a continuous estimated change in unit rank, not a simple subtraction of the first-year code from the last-year code. The calculation uses annual values for the entire period.

### 2.4 Eleven Climate Characteristic Maps

In addition to climate areas, all eleven characteristics can be displayed separately. Each map shows the mean value of the characteristic over the selected period. Its trend is also shown for a selected point; a trend that is not statistically significant is displayed as a dash.

### 2.5 Trend Information for a Selected Point

For a selected point, the application displays:

- **Total change** – the estimated change over the entire period;
- **Trend per decade** – the estimated change over ten years;
- **Significant trend: yes/no** – the result of the Mann–Kendall test;
- **Worst deviation from the trend** – the most severe continuous period of adverse deviations;
- **Duration** – the length of this episode in years;
- **Mean deterioration** – the mean magnitude of the adverse deviation during the episode.

In this context, “significant” means **statistically significant**, not automatically ecologically or economically important.

---

## 3. How to Read the Main Results

### 3.1 Climate Area and Climate Unit

The Quitt scheme distinguishes three broad areas:

- **CH – Cold area**: `CH1` to `CH7`;
- **MT – Moderately warm area**: `MT1` to `MT11`;
- **T – Warm area**: `T1` to `T5`.

The application returns a specific **climate unit**, not only one of the three broad areas. The unit is the closest climate type based on the joint evaluation of all characteristics.

The English version uses the atlas codes `C1–C7`, `MW1–MW11`, and `W1–W5`. The internal calculation codes remain `CH`, `MT`, and `T` [T3].

### 3.2 Match

The **match** value (`best_score`) ranges from 0 to 1. It expresses how well the eleven calculated values fit the intervals of the best climate unit.

- a value close to `1` indicates a very good similarity;
- an intermediate value indicates partial similarity or a position near the boundaries of several units;
- a low value indicates that even the best of the 23 units does not match the location particularly well.

The match score is **not a probability**, confidence interval, or proportion of an area. For example, a score of `0.69` must not be interpreted as a 69% probability that the location belongs to the unit.

### 3.3 Mean

For a climate characteristic, “Mean” denotes the arithmetic mean of annual values over the entire selected period. For example, the mean number of summer days for 1990–2009 is the mean of twenty annual values.

### 3.4 Trend and Statistical Significance

The trend is estimated from annual values using Sen’s slope. Statistical significance is assessed using the Mann–Kendall test. If the test does not establish a trend at the selected significance level, the application displays a dash for the trend of the individual characteristic. This does not mean that the value remained unchanged throughout the period; it means that the available series does not provide sufficiently strong evidence of a monotonic trend.

### 3.5 Worst Adverse Episode

The worst episode is the most severe continuous sequence of years in which values were on the adverse side of the robust trend line. The “adverse” direction depends on the characteristic. For example, an increase in summer days is considered an adverse increase by default, whereas a decrease in precipitation during the growing season is considered an adverse decrease [T2].

---

## 4. Climate Data Sources and Monitored Locations

### 4.1 Locations

The application catalogue contains the following locations [T4]:

| Code | Czech name | English name | Bounding box `[minLon, minLat, maxLon, maxLat]` |
|---|---|---|---|
| `cz` | Česko | Czechia | `[12.07, 48.48, 18.97, 51.08]` |
| `sk` | Slovensko | Slovakia | `[16.8174, 47.6296, 22.7629, 49.705]` |
| `maestrazgo_mount` | pohoří Maestrazgo | Maestrazgo Mountains | `[-1.09395, 39.8289, 0.297912, 40.8966]` |
| `southern_fi` | Jižní Finsko | Southern Finland | `[20.3719, 58.9769, 32.7637, 63.1035]` |

The bounding box defines the data coverage of the source. It does not automatically represent the administrative boundary of a country or pilot area.

### 4.2 Climate Data Sources

| Name displayed to users | Technical name | Available locations |
|---|---|---|
| Historical climate / ERA5-Land model / 10 × 10 km | `ERA5-Land` | `cz` |
| Future climate, pessimistic scenario / MPI model / 12 × 12 km | `MPI / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |
| Future climate, pessimistic scenario / EC-Earth model / 12 × 12 km | `EC-Earth / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |
| Future climate, pessimistic scenario / MIROC model / 12 × 12 km | `MIROC / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |

Historical reanalysis and a climate model are not the same:

- **reanalysis** combines meteorological observations with a numerical model and provides a consistent reconstruction of past climate;
- a **climate projection** simulates possible future development under a selected emissions/scenario assumption and model chain.

Results from different models should not be understood as exact forecasts for a specific year. They are intended for assessing possible long-term developments and model uncertainty.

---

## 5. Relationship to the Quitt Classification

### 5.1 Original Classification Scheme

The Quitt classification is an example of **complex climatology**: an area is not classified according to one variable, but according to a combination of value classes for several climatological characteristics. The UPOL/CHMI publication describes 23 climate units in three areas, defined by combinations of 14 characteristics [1, pp. 3–6; PDF pp. 4–7].

The original limits were designed for the climatic conditions of Czechoslovakia and were based on historical map sources. The UPOL/CHMI authors note that the intervals of individual units overlap and that practical implementation of the classification involves uncertainty and a degree of expert judgement [1, p. 4; PDF p. 5].

### 5.2 Assignment Uncertainty

The detailed UPOL/CHMI analysis showed that no evaluated grid square satisfied all 14 original criteria, and only 6 to 9 criteria were most commonly satisfied. The same maximum may also occur for more than one unit [1, pp. 13–14; PDF pp. 14–15]. The result should therefore be understood as the **closest climate type**, not as an absolute and error-free boundary.

UPOL/CHMI also describes a digital maximum-number-of-satisfied-criteria method in which the warmer unit is preferred in the event of a tie. At the same time, it notes that different methods can produce different map implementations from the same input data [1, p. 14; PDF p. 15].

### 5.3 What Info4forest Adopts and What It Adds

Info4forest adopts:

- the system of 23 units and three broad areas;
- the unit codes and verbal characteristics;
- the interval limits of the climate characteristics used;
- the orientation of the scale from colder to warmer units.

Info4forest adds its own algorithmic procedures:

- it uses eleven available characteristics;
- it evaluates similarity using a fuzzy score rather than a purely binary pass/fail rule;
- it interpolates point values using IDW;
- it supports any available multi-year period;
- it calculates robust trends, their statistical significance, and adverse episodes;
- it works with multiple climate data sources and locations.

The correct designation is therefore **“climate areas inspired by the Quitt classification”**. The output is not identical to the historical Quitt map or the UPOL/CHMI map for 1961–2000.

---

## 6. The Eleven Climate Characteristics Used

| Data index | Code | Name | Unit | Meaning |
|---:|---|---|---|---|
| 0 | `SUM25` | Annual number of summer days | days | Number of days on which the daily maximum temperature reaches at least 25 °C. |
| 1 | `VEG10` | Annual number of days with mean temperature ≥ 10 °C | days | An approximate indicator of the length of the warmer part of the year suitable for vegetation. |
| 2 | `FROST0` | Annual number of frost days | days | Number of days on which the daily minimum temperature falls below 0 °C. |
| 3 | `ICE0` | Annual number of ice days | days | Number of days on which the daily maximum temperature remains below 0 °C. |
| 4 | `T_JAN` | Mean January temperature | °C | Characteristic of mid-winter temperature conditions. |
| 5 | `T_APR` | Mean April temperature | °C | Characteristic of spring temperature conditions. |
| 6 | `T_JUL` | Mean July temperature | °C | Characteristic of mid-summer temperature conditions. |
| 7 | `T_OCT` | Mean October temperature | °C | Characteristic of autumn temperature conditions. |
| 8 | `RAIN1` | Annual number of precipitation days ≥ 1 mm | days | Number of days with a precipitation total of at least 1 mm. |
| 9 | `P_APR_SEP` | Precipitation total, April–September | mm | Total precipitation during the growing-season part of the year, from April to September. |
| 10 | `P_OCT_MAR` | Precipitation total, October–March | mm | Total precipitation during the cold part of the year, from October to March. |

The original Quitt scheme uses 14 characteristics [1, pp. 4–7; PDF pp. 5–8]. Info4forest uses eleven characteristics available in the harmonised WF5 data structure. Characteristics related to snow cover, cloudiness, and clear/overcast days are not included. This reduction is one reason why the output is described as inspired by the classification rather than as its complete reproduction.

---

## 7. Overview of Climate Areas and Units

### 7.1 Indicative Scale

```text
COLDEST                                                               WARMEST
CH1 → CH2 → CH3 → CH4 → CH5 → CH6 → CH7 → MT1 → … → MT11 → T1 → T2 → T3 → T4 → T5
│--------------- cold area ---------------││---- moderately warm area ----││-- warm --│
```

The order is an indicative climate scale used by the calculation. A difference of one rank cannot automatically be interpreted as a constant physical change in temperature or precipitation; neighbouring units are defined by a combination of several variables.

### 7.2 Overview of All 23 Units

The verbal characteristics in the following table are based on Table 1 of the UPOL/CHMI publication [1, pp. 4–5; PDF pp. 5–6] and are normalised according to the application metadata [T3].

| Internal code | English/atlas code | English name | Verbal characteristic |
|---|---|---|---|
| `CH1` | `C1` | Cold 1 | Very short, cold, very wet summer; very long transition period with a very cold spring and cold autumn; very long, very cold, very wet winter with a very long duration of snow cover. |
| `CH2` | `C2` | Cold 2 | Very short, cold, very wet summer, but with a lower precipitation total than C1; very long transition period with a very cold spring and cold autumn; very long, very cold, very wet winter, but with a lower precipitation total than C1, and a very long duration of snow cover. |
| `CH3` | `C3` | Cold 3 | Very short, cold, wet summer; very long transition period with a very cold to cold spring and cold autumn; very long, very cold, wet winter with a very long duration of snow cover. |
| `CH4` | `C4` | Cold 4 | Very short, cold, wet summer; very long transition period with a cold spring and moderately cold autumn; very long, very cold, wet winter with a very long duration of snow cover. |
| `CH5` | `C5` | Cold 5 | Very short to short, moderately cold, wet summer; long transition period with a cold spring and moderately cold autumn; very long, cold, moderately wet winter with a long duration of snow cover. |
| `CH6` | `C6` | Cold 6 | Very short to short, moderately cold, wet to very wet summer; long transition period with a cold spring and moderately cold autumn; very long, moderately cold, wet winter with a long duration of snow cover. |
| `CH7` | `C7` | Cold 7 | Very short to short, moderately cold, wet summer; long transition period with a moderately cold spring and mild autumn; long, mild, moderately wet winter with a long duration of snow cover. |
| `MT1` | `MW1` | Moderately Warm 1 | Short, moderately cold, wet summer; very long transition period with a moderately cold spring and mild autumn; normally long, cold, dry to moderately dry winter with a long duration of snow cover. |
| `MT2` | `MW2` | Moderately Warm 2 | Short, mild to moderately cold, moderately wet summer; short transition period with a mild spring and mild autumn; normally long winter with mild temperatures, dry conditions, and a normally long duration of snow cover. |
| `MT3` | `MW3` | Moderately Warm 3 | Short, mild to moderately cold, dry to moderately dry summer; normal to long transition period with a mild spring and mild autumn; normally long, mild to moderately cold, dry to moderately dry winter with a normal to short duration of snow cover. |
| `MT4` | `MW4` | Moderately Warm 4 | Short, mild, dry to moderately dry summer; short transition period with a mild spring and mild autumn; normally long, moderately warm, dry winter with a short duration of snow cover. |
| `MT5` | `MW5` | Moderately Warm 5 | Normally long to short, mild to moderately cold, dry to moderately dry summer; normal to long transition period with a mild spring and mild autumn; normally long, moderately cold, dry to moderately dry winter with a normal duration of snow cover. |
| `MT6` | `MW6` | Moderately Warm 6 | Normally long to long, mild, moderately wet summer; normal to long transition period with a mild to moderately warm spring and mild autumn; normally long, cold, dry to moderately dry winter with a normal duration of snow cover. |
| `MT7` | `MW7` | Moderately Warm 7 | Normally long, mild, moderately dry summer; short transition period with a mild spring and moderately warm autumn; normally long, moderately warm, dry to moderately dry winter with a short duration of snow cover. |
| `MT8` | `MW8` | Moderately Warm 8 | Long, warm, moderately wet summer; normally long transition period with a moderately warm spring and moderately warm autumn; normally long, mild to moderately cold, dry winter with a short duration of snow cover. |
| `MT9` | `MW9` | Moderately Warm 9 | Long, warm, dry to moderately dry summer; short transition period with a mild to moderately warm spring and moderately warm autumn; short, mild, dry winter with a short duration of snow cover. |
| `MT10` | `MW10` | Moderately Warm 10 | Long, warm, moderately dry summer; short transition period with a moderately warm spring and moderately warm autumn; short, moderately warm, very dry winter with a short duration of snow cover. |
| `MT11` | `MW11` | Moderately Warm 11 | Long, warm, dry summer; short transition period with a moderately warm spring and moderately warm autumn; short, warm, very dry winter with a short duration of snow cover. |
| `T1` | `W1` | Warm 1 | Long, warm, dry summer; short transition period with a moderately warm to warm spring and moderately warm to warm autumn; short, mild to moderately cold, dry to very dry winter with a short duration of snow cover. |
| `T2` | `W2` | Warm 2 | Long, warm, dry summer; very short transition period with a warm to moderately warm spring and moderately warm to warm autumn; short, moderately warm, dry to very dry winter with a very short duration of snow cover. |
| `T3` | `W3` | Warm 3 | Very long, very warm, dry summer; short transition period with a warm spring and warm autumn; short, mild, dry to very dry winter with a short duration of snow cover. |
| `T4` | `W4` | Warm 4 | Very long, very warm, very dry summer; very short transition period with a warm spring and warm autumn; short, moderately warm, dry to very dry winter with a very short duration of snow cover. |
| `T5` | `W5` | Warm 5 | Very long, very warm, very dry summer; very short transition period with a warm spring and warm autumn; very short, warm, dry to very dry winter with a very short duration of snow cover. |

---

## 8. Methodology for Calculating the Climate Unit

### 8.1 Input Data Structure

Annual climate characteristics are stored in a raster collection in the following order:

```text
[year, indicator, y, x]
```

Each spatial cell therefore contains eleven values for every year. The application first verifies that the selected period and coordinates fall within the extent of the selected data source [T1].

### 8.2 Multi-year Mean

For the static map, the mean across all years in the selected period is calculated for each indicator `i` and location `s`:

```text
mean_i(s) = (1 / n) × Σ I_i(year, s)
```

Both the start year and the end year are included. The calculation therefore does not compare only the first and last year.

### 8.3 Spatial Interpolation

Map output on the native data grid uses the source-cell values directly. For an exact selected point, which will usually not lie at the centre of a cell, **IDW – inverse distance weighting** is used. Nearby data points receive greater weight than more distant points:

```text
w_j = 1 / d_j^p
interpolated_value = Σ(w_j × value_j) / Σ(w_j)
```

The default exponent is `p = 2`. At most nine nearest valid points from the local neighbourhood are used. Distance is calculated along the Earth’s surface. If the requested coordinate exactly matches a source point, its value is returned without averaging [T1, T5].

IDW does not increase the actual spatial resolution of the source data. It creates a smooth estimate between the available points.

### 8.4 Partial Similarity to an Interval

Each climate unit contains a lower and upper limit for each indicator. Similarity equals `1` for a value inside the interval. Outside the interval, it decreases linearly to zero:

```text
μ = 1                                         for lower ≤ value ≤ upper
μ = max(0, 1 − (lower − value) / tolerance)   for value < lower
μ = max(0, 1 − (value − upper) / tolerance)   for value > upper
```

The following tolerance widths are used [T5]:

| Group | Characteristics | Tolerance |
|---|---|---:|
| Numbers of days | `SUM25`, `VEG10`, `FROST0`, `ICE0`, `RAIN1` | 10 days |
| Temperatures | `T_JAN`, `T_APR`, `T_JUL`, `T_OCT` | 1 °C |
| Precipitation totals | `P_APR_SEP`, `P_OCT_MAR` | 50 mm |

The tolerances are not adopted as a separate parameter from the original Quitt publication; they are part of the Info4forest calculation implementation.

### 8.5 Overall Score and Unit Selection

The overall score of a unit is the arithmetic mean of the partial similarities of all valid indicators:

```text
score(unit) = mean(μ_1, μ_2, …, μ_11)
best_unit   = unit with the highest score
```

If several units have exactly the same result, the unit occurring later in the order from `CH1` to `T5` is preferred, meaning the warmer unit [T5]. This principle is consistent with one of the digital implementations described by UPOL/CHMI, which also preferred the warmer unit when maxima were equal [1, p. 14; PDF p. 15].

---

## 9. Trend Methodology

### 9.1 Why Only the First and Last Year Are Not Used

A difference between two years can be strongly affected by an unusually warm, cold, dry, or wet year. Trends therefore use all valid annual values in the selected period.

### 9.2 Sen’s Slope

For every pair of years, the change in value divided by the time difference is calculated. Sen’s slope is the median of all these pairwise slopes:

```text
slope_per_year = median((value_j − value_i) / (year_j − year_i))
slope_per_decade = 10 × slope_per_year
estimated_total_change = slope_per_year × (max_year − min_year)
```

The median is less sensitive to individual extreme years than ordinary linear regression [T2].

### 9.3 Mann–Kendall Test

The Mann–Kendall test determines whether values exhibit a statistically significant monotonic tendency. The implementation accounts for tied values and uses a normal approximation. The application uses the default level `α = 0.05` [T2].

- `significant = true`: the trend is statistically significant at the given level;
- `significant = false`: statistical significance was not established.

Statistical significance alone does not indicate the magnitude or practical impact of a change. `estimated_total_change`, `slope_per_decade`, the length of the period, and the nature of the data source must therefore also be considered.

### 9.4 Climate Unit Trend

Each climate unit has a rank from the coldest to the warmest. For each year, a continuous expected position on this scale (`expected_fuzzy_rank`) is derived from the fuzzy scores. Sen’s slope is then applied to the annual series of this position [T2].

A positive trend indicates a shift towards warmer units; a negative trend indicates a shift towards colder units. The value may be decimal because it is the trend of a continuous expected position, not the number of discrete colour categories crossed.

### 9.5 Trend of an Individual Characteristic

For maps and details of individual characteristics, the trend is calculated in the physical unit of the variable:

- days per decade for counts of days;
- °C per decade for temperatures;
- mm per decade for precipitation totals.

Total change is the trend estimate for the entire period, not the difference between the first and last measured or modelled value.

### 9.6 Worst Adverse Episode

A robust trend line is first constructed using Sen’s slope and a median intercept. For each year, the deviation from this line in a predefined adverse direction is determined. The algorithm searches for continuous sequences of positive adverse deviation and selects the sequence with the greatest total severity [T2].

- `duration` – number of consecutive years;
- `severity` – sum of adverse deviations;
- `mean_severity` – mean deviation during the episode;
- `peak_anomaly` – largest individual adverse deviation.

This diagnostic describes episodes **relative to the long-term trend**, not automatically the worst climate period in absolute terms from the perspective of forest management.

---

## 10. Interpretation, Uncertainty, and Limitations

### 10.1 A Climate Unit Is the Closest Similarity

The boundaries of climate units overlap in the original scheme, and according to UPOL/CHMI their numerical limits are only indicative in relation to reality [1, p. 18; PDF p. 19]. The result should therefore not be interpreted as a precise natural boundary.

### 10.2 The Classification Was Developed for Different Historical Conditions

The original limits are based on climate values from historical Czechoslovakia. When applied to other regions, future scenarios, or values outside the historical range, some profiles may be distant from all units. This will be reflected in a lower match score or values near the ends of the scale.

### 10.3 Eleven Characteristics Are Used

Info4forest does not use all the original characteristics. The result must therefore not be presented as a complete reproduction of the 1971 classification or the UPOL/CHMI map.

### 10.4 Map Resolution Is Limited by the Source Data

Smooth rendering and IDW may appear more detailed than the actual model resolution. For example, a source with an approximate resolution of 10 × 10 km cannot reliably describe the microclimate of a slope, valley, or forest stand at the scale of individual hectares.

### 10.5 Model Sources Include Uncertainty

Future climate data depend on the global model, regional model, scenario, and natural climate variability. For decision-making, it is advisable to compare multiple sources and periods and, where appropriate, other expert evidence.

### 10.6 Climate Unit Trend Is a Derived Quantity

A unit combines several characteristics. The same shift on its trend scale may be caused by different combinations of temperature and precipitation changes at different locations. For substantive interpretation, the trends of the individual characteristics should therefore also be examined.

### 10.7 Appropriate and Inappropriate Uses

The outputs are particularly suitable for:

- an indicative description of the climate type of an area;
- comparison of periods, locations, or climate data sources;
- screening of long-term changes relevant to forest planning;
- identifying areas suitable for more detailed analysis.

The outputs alone are not a substitute for:

- local measurements and microclimate assessment;
- dendrological, soil, hydrological, or site assessment;
- operational weather forecasting;
- an unequivocal basis for major decisions without further expert interpretation.

---

## 11. The API as the Data and Calculation Backend

The Info4forest application obtains its source catalogue, map data, point characteristics, and trend diagnostics through the FWF5 Climate API. General users do not need to use the API directly, but its publication supports transparency, reproducibility, and integration of the results into other systems.

The API also provides more detailed functions than must always be visible simultaneously in the main interface, including:

- a complete catalogue of climate data sources and locations;
- metadata for climate characteristics and units;
- annual series of all eleven characteristics for a specific point;
- separate map and point trend outputs;
- technical information about the temporal and spatial extents of data collections;
- richer diagnostic information on trends, series quality, and shifts over periods of different lengths.

The current technical API documentation is available at:

**<https://app.swaggerhub.com/apis/JIRKAVALES_1/Info4forest/1.0.0>**

Examples of endpoints used as the application’s backend:

- `/climaticsourcecatalog` – climate data sources and their lay and technical names;
- `/climaticsourcecatalog/locations` – locations and data extents;
- `/climaticzones/grid` and `/climaticzones/point` – climate units;
- `/climaticzones/trend/grid` and `/climaticzones/trend/point` – climate-unit trends;
- `/climatecharacteristics/{characteristic}/grid` – maps of individual characteristics;
- `/climatecharacteristics/trend/{characteristic}/point` – trend of a characteristic at a point.

These names are included only to make the technical backend traceable; the main purpose of this document is the substantive interpretation of the visualisation and methodology.

---

## 12. Recommended Citation

When citing a map or result, it is advisable to state:

1. the application and workflow: **Info4forest, WF5 – Forest-related Climate Area Map**;
2. the climate data source/model and scenario;
3. the location and selected period;
4. the date on which the result was created or downloaded;
5. a link to this methodology;
6. the specialist methodological source [1] and, where appropriate, the original Quitt work [2].

Example:

> Info4forest (2026): WF5 – Forest-related Climate Area Map, ERA5-Land climate data source, Czechia, 1991–2020. Climate units inspired by the Quitt classification; methodology according to Květoň and Voženílek (2011) and the Info4forest implementation.

---

## 13. Sources

### Specialist and Methodological Sources

**[1]** KVĚTOŇ, Vít; VOŽENÍLEK, Vít. *Klimatické oblasti Česka: klasifikace podle Quitta za období 1961–2000 / Climatic Regions of Czechia: Quitt's Classification during Years 1961–2000*. Olomouc: Palacký University Olomouc, co-published with the Czech Hydrometeorological Institute, 2011. M.A.P.S., Num. 3. ISBN 978-80-244-2813-0; ISBN 978-80-86690-89-6.

**[2]** QUITT, Evžen. *Klimatické oblasti Československa* [Climate Areas of Czechoslovakia]. Studia Geographica, Vol. 16. Brno: Institute of Geography, Czechoslovak Academy of Sciences, 1971, 73 pp. Bibliographic information according to [1, p. 19; PDF p. 20].

### Technical Implementation Sources

**[T1]** `FWF5_API.py` – API layer, input validation, catalogue, calculation orchestration, and response format.  
**[T2]** `FWF5_trends.py` – Sen’s slope, Mann–Kendall test, climate-unit trends, and adverse episodes.  
**[T3]** `FWF5_climate_metadata.json` – Czech and English names, codes, and verbal descriptions of climate units and characteristics.  
**[T4]** `climate_source_catalog.json` – catalogue of climate data sources and locations, lay and technical names, and Rasdaman collection names.  
**[T5]** `FWF5_quitt_limits_11_rasdaman_index.json`, `FWF5_quitt_fuzzy_match_point.py`, and `FWF5_quitt_fuzzy_match_area_custom_grid.py` – limit intervals, fuzzy scoring, IDW, and the order of climate units.  
**[T6]** OpenAPI documentation for the FWF5 Climate API: <https://app.swaggerhub.com/apis/JIRKAVALES_1/Info4forest/1.0.0>.

---

## 14. Appendix A – Limit Intervals Used by the Implementation

The following table gives the exact intervals of the eleven characteristics used by the current implementation [T5]. Units: counts of days in days, temperatures in °C, and precipitation totals in mm.

| Unit | SUM25 | VEG10 | FROST0 | ICE0 | T_JAN | T_APR | T_JUL | T_OCT | RAIN1 | P_APR_SEP | P_OCT_MAR |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `CH1` | 0–10 | 0–80 | 160–180 | 60–80 | -8–-7 | 0–2 | 10–12 | 2–4 | 140–160 | 900–1000 | 600–700 |
| `CH2` | 0–10 | 0–80 | 160–180 | 60–70 | -8–-7 | 0–2 | 10–12 | 2–4 | 140–160 | 700–900 | 500–600 |
| `CH3` | 0–20 | 80–120 | 160–180 | 60–70 | -8–-7 | 0–2 | 12–14 | 2–4 | 120–140 | 600–700 | 400–500 |
| `CH4` | 0–20 | 80–120 | 160–180 | 60–70 | -7–-6 | 2–4 | 12–14 | 4–5 | 120–140 | 600–700 | 400–500 |
| `CH5` | 10–30 | 100–120 | 140–160 | 60–70 | -6–-5 | 2–4 | 14–15 | 5–6 | 120–140 | 500–600 | 350–400 |
| `CH6` | 10–30 | 120–140 | 140–160 | 60–70 | -5–-4 | 2–4 | 14–15 | 5–6 | 140–160 | 600–700 | 400–500 |
| `CH7` | 10–30 | 120–140 | 140–160 | 50–60 | -4–-3 | 4–6 | 15–16 | 6–7 | 120–130 | 500–600 | 350–400 |
| `MT1` | 20–30 | 120–140 | 160–180 | 40–50 | -6–-5 | 5–6 | 15–16 | 6–7 | 120–130 | 500–600 | 300–350 |
| `MT2` | 20–30 | 140–160 | 110–130 | 40–50 | -4–-3 | 6–7 | 16–17 | 6–7 | 120–130 | 450–500 | 250–300 |
| `MT3` | 20–30 | 120–140 | 130–160 | 40–50 | -4–-3 | 6–7 | 16–17 | 6–7 | 110–120 | 350–450 | 250–300 |
| `MT4` | 20–30 | 140–160 | 110–130 | 40–50 | -3–-2 | 6–7 | 16–17 | 6–7 | 110–120 | 350–450 | 250–300 |
| `MT5` | 30–40 | 140–160 | 130–140 | 40–50 | -5–-4 | 6–7 | 16–17 | 6–7 | 100–120 | 350–450 | 250–300 |
| `MT6` | 30–40 | 140–160 | 140–160 | 40–50 | -6–-5 | 6–7 | 16–17 | 6–7 | 100–120 | 450–500 | 250–300 |
| `MT7` | 30–40 | 140–160 | 110–130 | 40–50 | -3–-2 | 6–7 | 16–17 | 7–8 | 100–120 | 400–450 | 250–300 |
| `MT8` | 40–50 | 140–160 | 130–140 | 40–50 | -5–-4 | 7–8 | 17–18 | 7–8 | 100–120 | 400–450 | 250–300 |
| `MT9` | 40–50 | 140–160 | 110–130 | 30–40 | -4–-3 | 6–7 | 17–18 | 7–8 | 100–120 | 400–450 | 250–300 |
| `MT10` | 40–50 | 140–160 | 110–130 | 30–40 | -3–-2 | 7–8 | 17–18 | 7–8 | 100–120 | 400–450 | 200–250 |
| `MT11` | 40–50 | 140–160 | 110–130 | 30–40 | -3–-2 | 7–8 | 17–18 | 7–8 | 90–100 | 350–400 | 200–250 |
| `T1` | 50–60 | 160–170 | 120–130 | 30–40 | -5–-3 | 7–8 | 17–19 | 7–9 | 90–100 | 350–400 | 200–300 |
| `T2` | 50–60 | 160–170 | 100–110 | 30–40 | -3–-2 | 8–9 | 18–19 | 7–9 | 90–100 | 350–400 | 200–300 |
| `T3` | 60–70 | 170–180 | 110–120 | 30–40 | -4–-3 | 8–10 | 19–20 | 8–9 | 90–100 | 350–400 | 200–300 |
| `T4` | 60–70 | 170–180 | 100–110 | 30–40 | -3–-2 | 9–10 | 19–20 | 9–10 | 80–90 | 300–350 | 200–300 |
| `T5` | 60–70 | ≥ 180 | 90–100 | ≤ 30 | -2–-1 | 9–10 | 19–20 | 9–10 | 80–90 | 300–350 | 200–300 |

### Note on Using the Table

The intervals should not be used in isolation as a simple decision tree. The implementation evaluates all eleven characteristics jointly and uses a smoothly decreasing fuzzy similarity outside the intervals. UPOL/CHMI also notes that the original numerical limits are indicative and that different implementations of the classification may produce different maps [1, p. 18; PDF p. 19].
