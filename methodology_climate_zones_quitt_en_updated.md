# Methodology for Calculating Climate Zones Inspired by Quitt’s Classification

**Purpose of this document:** a methodological description of the climate-zone calculation used by the API. The document is intended for publication in GitLab/GitLab Pages and for linking from OpenAPI/Swagger documentation using `externalDocs`.

**Document version:** 1.1  
**Date:** 2026-06-04  
**Methodological basis:** E. Quitt’s climate classification and the methodology described in the publication *Climatic Regions of Czechia: Quitt’s Classification During Years 1961–2000*, published by Palacký University Olomouc and the Czech Hydrometeorological Institute.  
**Publication link:** <https://www.cartography.upol.cz/MAPS/MAPS_Num3_brozura.pdf>

---

## 1. Plain-language summary

The climate-zone calculation works much like filling in a climate questionnaire for each location and then finding which known climate region that location most closely resembles.

For each location and selected period, for example 1991–2020, the calculation first derives 11 climate characteristics. These include, for example, the number of summer days, the number of frost days, mean temperature in January, April, July and October, and precipitation totals in the vegetation and winter periods. Together, these characteristics describe whether a location is warmer or colder, drier or wetter, and how pronounced its summer and winter conditions are.

The value of each characteristic is then averaged over the selected period. If the user requests a point that does not lie exactly at the centre of an input data cell, the values are interpolated from surrounding cells using IDW, i.e. inverse distance weighting. In this method, closer cells have a stronger influence than more distant cells.

The resulting 11 values are then compared with a table of Quitt climate units. Quitt’s classification distinguishes units `CH1` to `CH7` for the cold region, `MT1` to `MT11` for the moderately warm region, and `T1` to `T5` for the warm region. Each unit is defined by value ranges, for example how many summer days it should have or what its mean July temperature should be.

Because real-world data rarely fit exactly into only one unit, the calculation does not use a strict “matches / does not match” rule only. Instead, it uses a similarity score from 0 to 1. A value of 1 means that a characteristic lies inside the interval for the given climate unit. If the value is just outside the interval, it receives a partial score. If it is too far away, it receives a score of 0. The total score of a climate unit is the average of the scores across all 11 characteristics.

The resulting climate zone is the unit with the highest score. If two units receive the same score, the calculation prefers the warmer unit. This follows the principle described in the UPOL methodology for digital assignment according to the maximum number of fulfilled criteria.

The API output is GeoJSON. For each point, the output contains the code of the best-matching climate unit, for example `MT7`, and the value `best_score`, which indicates how well the location corresponds to the selected unit. The score is not a probability; it is a measure of similarity to the intervals in the Quitt table.


In addition to the static climate-zone calculation for a selected multi-year period, the API can also calculate trends. Trend endpoints do not compare only the first and last year. They use the annual values for all years in the selected period and estimate a robust Sen slope. This makes the result less sensitive to individual anomalous years. Trends can be calculated either for individual climate characteristics, such as July temperature or vegetation-period precipitation, or for climate zones. Climate-zone trends support two metrics: `expected_fuzzy_rank` and `exceedance_rank_equivalent`.

---

## 2. Relationship to Quitt’s classification and the UPOL methodology

Quitt’s climate classification is a complex climatological classification. A territory is not assigned to a class according to a single variable, for example mean temperature alone, but according to a combination of several climatological characteristics. The original Quitt scheme contains 23 climate units in three main regions:

- **CH** – cold region (`CH1` to `CH7`),
- **MT** – moderately warm region (`MT1` to `MT11`),
- **T** – warm region (`T1` to `T5`).

According to the UPOL/CHMI publication, the original Quitt classification distinguishes 23 units defined by combinations of values of 14 climatological characteristics. Unit boundaries are determined by changes in these characteristics, and the individual intervals may partly overlap. The UPOL publication notes that practical implementation of the classification involves uncertainty, because one location may satisfy the characteristics of several units at the same time.

This implementation is **inspired by Quitt’s classification and the UPOL methodology**, but it is not a direct reproduction of the original map or its manual cartographic interpretation. The calculation is algorithmic, repeatable, and based on precomputed gridded climate indicators. Compared with the original set of 14 characteristics, the implementation uses 11 available characteristics. The metadata of the limit table list `PDZAT`, `PDJAS`, `PDSCE` and `PDOBL` as characteristics not used in this implementation.

The main difference from a simple “maximum number of fulfilled criteria” method is that this implementation uses fuzzy scoring. Instead of assigning each characteristic only a value of 0 or 1, a value slightly outside the interval still receives a partial match. This makes the calculation more stable in cases where the values are close to the boundary between climate units.

---

## 3. Input data

The calculation uses a precomputed Rasdaman collection of annual climate indicators. The collection has four dimensions:

```text
[year, ind, y, x]
```

where:

- `year` is the calendar year,
- `ind` is the climate-characteristic index from 0 to 10,
- `y` is the row index of the spatial grid,
- `x` is the column index of the spatial grid.

Example of a production collection in the attached registry:

```text
era5l_cz_yindicators
```

This collection is described as annual precomputed climate indicators on the Czech grid, stored as a 4D array `[year, ind, y, x]`. Scenario or model data can use other collections of the same type, provided that they have the same indicator structure.

The spatial grid is defined in the file `rasdaman_registry.json`. The registry contains metadata for collections, time axes, spatial grids, coordinates, and Rasdaman connection settings.

---

## 4. The 11 climate characteristics used

| Index `ind` | Code | Meaning | Unit |
|---:|---|---|---|
| 0 | `SUM25` | Annual number of summer days, i.e. days with maximum temperature at least 25 °C | days |
| 1 | `VEG10` | Annual number of days with mean temperature at least 10 °C | days |
| 2 | `FROST0` | Annual number of frost days, i.e. days with minimum temperature below 0 °C | days |
| 3 | `ICE0` | Annual number of ice days, i.e. days with maximum temperature below 0 °C | days |
| 4 | `T_JAN` | Mean January temperature | °C |
| 5 | `T_APR` | Mean April temperature | °C |
| 6 | `T_JUL` | Mean July temperature | °C |
| 7 | `T_OCT` | Mean October temperature | °C |
| 8 | `RAIN1` | Annual number of precipitation days with precipitation of at least 1 mm | days |
| 9 | `P_APR_SEP` | Precipitation total from April to September, i.e. the vegetation period | mm |
| 10 | `P_OCT_MAR` | Precipitation total from October to March, i.e. the winter period | mm |

The indicator order is important because it corresponds both to the `ind` axis in the Rasdaman collection and to the mapping in `FWF5_quitt_limits_11_rasdaman_index.json`.

---

## 5. Overview of the calculation workflow

The full calculation can be summarised in the following steps:

1. **Select the collection and period**  
   The user provides the Rasdaman collection, start year and end year. The API checks that the requested years are available.

2. **Select the location or area**  
   The user provides either a point (`lat`, `lon`) or a spatial extent (`bbox`). For grid output, the user may also provide the output-grid spacing `step_lat` and `step_lon`.

3. **Read annual indicators from Rasdaman**  
   A data block is read from the collection for the selected years, indicators 0 to 10, and the relevant spatial window.

4. **Calculate multi-year averages**  
   Indicator values are summed across years and divided by the number of years. This creates an array of average indicators for the whole period.

5. **Interpolate to the requested point or custom output grid**  
   If the output is calculated outside the original cell centres, the 11 indicator values are interpolated using IDW from neighbouring Rasdaman cells.

6. **Compare with the Quitt limit table**  
   For each of the 23 climate units, the match between each indicator and the interval for that unit is calculated.

7. **Select the best climate unit**  
   For each unit, the average fuzzy score is calculated. The unit with the highest score is selected. In the case of a tie, the warmer unit is selected.

8. **Create the output**  
   The result is GeoJSON with point geometry. The properties contain at least `best_unit` and `best_score`; for point calculations, they also contain the averaged values of all 11 indicators. Trend endpoints follow the same GeoJSON structure, but their properties contain trend metrics derived from annual values.

---

## 6. Calculation of multi-year averages

Let:

- `i` be the index of the climate characteristic,
- `y` be the year,
- `s` be a Rasdaman grid cell,
- `I_i(y, s)` be the annual value of indicator `i` in year `y` and cell `s`,
- `Y` be the set of years from `start_year` to `end_year`, inclusive,
- `n` be the number of years in the selection.

The multi-year average of an indicator is calculated as:

```text
mean_i(s) = (1 / n) * sum_{y in Y} I_i(y, s)
```

For efficiency, the code uses a Rasdaman query with `condense + over yr`, which sums the selected years directly on the data-store side. Python then divides the sum by the number of years.

The calculation assumes that the Rasdaman collection contains complete coverage for the requested period. If a requested year is missing from the collection, the calculation stops with an error. Numeric missing values are represented as `NaN` and may affect subsequent interpolation or scoring.

---

## 7. Spatial grid handling

Grid coordinates are loaded using the universal module `grid_coords_universal.py`. The implementation supports two types of grids:

1. **Separable 1D grid**  
   Longitudes and latitudes can be generated as two separate 1D arrays, typically using `linspace`. An example is the Czech ERA5-Land grid with regular spacing.

2. **Lookup 2D grid**  
   Latitude and longitude are stored as 2D coordinate layers. This mode is suitable, for example, for rotated regional climate-model grids.

For a point calculation, the nearest grid cell is first found and a candidate window is selected around it. For a grid calculation, the spatial data extent required from Rasdaman is determined from the requested area.

---

## 8. IDW interpolation

If the requested point or a point of a custom output grid does not lie exactly at the centre of a Rasdaman cell, the 11 indicator values are interpolated using IDW, i.e. inverse distance weighting.

Let:

- `p` be the requested point,
- `g_j` be neighbouring Rasdaman points,
- `d_j` be the distance between `p` and `g_j`, calculated using haversine distance,
- `p_idw` be the IDW power parameter, with default value `2.0`,
- `k` be the maximum number of neighbours used, with default value `9`.

The weight of a neighbour is:

```text
w_j = 1 / d_j^p_idw
```

The interpolated value of an indicator is:

```text
I_hat_i(p) = sum_j(w_j * mean_i(g_j)) / sum_j(w_j)
```

If the requested point lies exactly at the centre of a grid cell, interpolation is not used and the value of that cell is returned.

Default settings:

| Parameter | Meaning | Default value |
|---|---|---:|
| `idw_power` | distance-weighting exponent | `2.0` |
| `idw_k` | maximum number of nearest neighbours for grid calculation | `9` |
| `idw_radius_px` | radius of the candidate window in pixels | usually `2` in the API |
| `idw_max_dist_m` | optional maximum neighbour distance in metres | `null` |

The higher the `idw_power`, the more influence the nearest grid cell has. The lower the `idw_power`, the more the result approaches a simple average of neighbouring cells.

---

## 9. Quitt climate-unit limit table

Limit values are stored in the file:

```text
FWF5_quitt_limits_11_rasdaman_index.json
```

The file contains:

- the source of limits derived from Quitt’s table of climate characteristics,
- the list of the 11 indicators used,
- indicator units,
- the order of climate units from cold to warm,
- the mapping of indicators to the `ind` axis in the Rasdaman collection,
- value intervals for each climate unit.

Climate units are evaluated in the following order:

```text
CH1, CH2, CH3, CH4, CH5, CH6, CH7,
MT1, MT2, MT3, MT4, MT5, MT6, MT7, MT8, MT9, MT10, MT11,
T1, T2, T3, T4, T5
```

This order is also important for resolving ties. Because the code overwrites the previous unit with a later unit if the score is identical, the warmer unit wins in the case of a tie.

---

## 10. Fuzzy scoring of climate units

For each climate unit `u` and each indicator `i`, there is an interval:

```text
[lower_{u,i}, upper_{u,i}]
```

The indicator value `v` receives a partial score `mu_{u,i}(v)`:

- if `v` lies inside the interval, the score is `1`,
- if `v` lies below the lower bound, the score decreases linearly according to the distance from the lower bound,
- if `v` lies above the upper bound, the score decreases linearly according to the distance from the upper bound,
- if the value is farther away than the tolerance band, the score is `0`.

Formally:

```text
mu = 1                                      if lower <= v <= upper
mu = max(0, 1 - (lower - v) / tolerance)    if v < lower
mu = max(0, 1 - (v - upper) / tolerance)    if v > upper
```

The tolerances used are:

| Indicator group | Indicators | Tolerance |
|---|---|---:|
| Day counts | `SUM25`, `VEG10`, `FROST0`, `ICE0`, `RAIN1` | 10 days |
| Temperatures | `T_JAN`, `T_APR`, `T_JUL`, `T_OCT` | 1 °C |
| Precipitation totals | `P_APR_SEP`, `P_OCT_MAR` | 50 mm |

The total score of a unit is the arithmetic mean of partial scores across all available indicators:

```text
score_u = mean_i(mu_{u,i})
```

The resulting unit is:

```text
best_unit = argmax_u(score_u)
```

The value `best_score` is the corresponding maximum score. Interpretation:

| `best_score` | Meaning |
|---:|---|
| close to `1.0` | the location matches the intervals of the selected unit very well |
| approximately `0.5–0.8` | the location partly matches the unit; it may lie near the boundary between multiple units |
| low score | the result should be interpreted cautiously; the best unit is only the closest one among the available options |

`best_score` is not a statistical probability. It is an algorithmic measure of similarity to the limit intervals.

---

## 11. Point calculation

Point calculation is used for an endpoint such as:

```text
POST /climaticzones/point
```

Typical input:

```json
{
  "rasdaman_collection": "era5l_cz_yindicators",
  "start_year": 1991,
  "end_year": 2020,
  "lat": 49.595,
  "lon": 17.251,
  "idw_radius_px": 2,
  "idw_power": 2.0
}
```

Workflow:

1. The API checks that the point lies within the spatial extent of the selected collection.
2. The nearest grid cell is found.
3. A small surrounding window is read from the Rasdaman collection.
4. The multi-year average is calculated for each of the 11 characteristics.
5. If the point does not lie exactly at the cell centre, the values are interpolated using IDW.
6. The 11 interpolated values are compared with the Quitt limit table.
7. GeoJSON with a single point is returned.

A typical output contains:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [17.251, 49.595]
      },
      "properties": {
        "best_unit": "MT7",
        "best_score": 0.842424,
        "years": {
          "start": 1991,
          "end": 2020,
          "count": 30
        },
        "indicators_avg_11": {
          "SUM25": 42.1,
          "VEG10": 166.4,
          "FROST0": 102.0,
          "ICE0": 31.6,
          "T_JAN": -1.2,
          "T_APR": 8.4,
          "T_JUL": 18.2,
          "T_OCT": 8.1,
          "RAIN1": 105.0,
          "P_APR_SEP": 420.5,
          "P_OCT_MAR": 235.8
        }
      }
    }
  ]
}
```

The values in the example are illustrative.

---

## 12. Area or output-grid calculation

Grid calculation is used for an endpoint such as:

```text
POST /climaticzones/grid
```

### 12.1 Native-grid mode

If `step_lat` and `step_lon` are not provided, the calculation works directly with the points of the native Rasdaman grid. An optional `bbox` can be provided to restrict the output to a specific area.

In this mode:

1. a block of values is read from the Rasdaman collection,
2. multi-year averages are calculated for each native cell,
3. fuzzy comparison with the Quitt table is performed for each cell,
4. the result is GeoJSON with points at the centres of the Rasdaman cells.

### 12.2 Custom output-grid mode

If `step_lat` and `step_lon` are provided, the API creates a custom regular output grid in geographic coordinates. Both parameters must be provided together.

Typical input:

```json
{
  "rasdaman_collection": "era5l_cz_yindicators",
  "start_year": 1991,
  "end_year": 2020,
  "bbox": {
    "start_lat": 48.6,
    "end_lat": 51.0,
    "start_lon": 12.2,
    "end_lon": 18.8
  },
  "step_lat": 0.05,
  "step_lon": 0.05,
  "idw_power": 2.0,
  "idw_k": 9,
  "idw_radius_px": 2
}
```

In this mode:

1. a list of target coordinates is created from `bbox`, `step_lat` and `step_lon`,
2. an expanded surrounding area required for IDW is read from the Rasdaman collection,
3. multi-year averages are calculated in native cells,
4. the 11 indicators are interpolated by IDW for each target point,
5. fuzzy classification is performed for each target point,
6. the result is GeoJSON with points on the custom output grid.

---

## 13. Trend calculations

Trend calculations are separate from the static multi-year climate-zone calculation. They are used when the user wants to describe how a climate characteristic or a climate zone changes over time within the selected period.

The key difference is that trend endpoints use the annual time series for all years from `start_year` to `end_year`, inclusive. The trend is not derived only from the first and last year, because individual years may be anomalously warm, cold, dry or wet.

For each requested point or output-grid point, the workflow is:

1. read annual values of the 11 indicators for all requested years,
2. interpolate annual values to the requested point using the same IDW logic as other endpoints,
3. derive the annual trend variable, depending on the endpoint and selected metric,
4. estimate the trend using Sen’s slope,
5. return the trend as GeoJSON feature properties.

The trend method is identified in the output as:

```text
sen_slope_yearly
```

At least two valid years are required for trend calculation.

### 13.1 Sen’s slope

Sen’s slope is a robust non-parametric trend estimate. For all valid year pairs `(a, b)`, where `b > a`, the slope is calculated as:

```text
slope_ab = (value_b - value_a) / (year_b - year_a)
```

The final slope is the median of all pairwise slopes:

```text
slope_per_year = median(slope_ab)
```

The API also reports:

```text
slope_per_decade = 10 * slope_per_year
estimated_total_change = slope_per_year * (end_year - start_year)
```

This approach reduces the influence of individual anomalous years compared with a simple first-year/last-year difference.

The output may also contain:

| Field | Meaning |
|---|---|
| `valid_years` | number of years with valid values used for the trend |
| `pair_count` | number of valid year pairs used for Sen’s slope |
| `sign_consistency` | share of non-zero year-pair slopes with the dominant sign |
| `direction` | qualitative trend direction, based on `slope_per_decade` and a threshold |
| `strength` | qualitative strength class derived from the absolute slope |

The field `sign_consistency` is not a formal significance test. It is a simple diagnostic indicating whether most pairwise year-to-year slopes point in the same direction.

### 13.2 Trends of individual climate characteristics

Climate-characteristic trend endpoints calculate the trend for one selected indicator only. They do not calculate climate-zone trends and they do not return all 11 indicators together.

Typical endpoint pattern:

```text
POST /climatecharacteristics/trend/<characteristic>/point
POST /climatecharacteristics/trend/<characteristic>/grid
```

where `<characteristic>` can be:

```text
sum25, veg10, frost0, ice0,
t_jan, t_apr, t_jul, t_oct,
rain1, p_apr_sep, p_oct_mar
```

For example:

```text
POST /climatecharacteristics/trend/t_jul/point
POST /climatecharacteristics/trend/p_apr_sep/grid
```

The annual value of the selected indicator is used directly as the trend variable. The resulting slope therefore has the physical unit of the indicator, for example:

- days per decade for `SUM25`, `VEG10`, `FROST0`, `ICE0` and `RAIN1`,
- °C per decade for `T_JAN`, `T_APR`, `T_JUL` and `T_OCT`,
- mm per decade for `P_APR_SEP` and `P_OCT_MAR`.

A typical trend object for a single indicator is:

```json
{
  "method": "sen_slope_yearly",
  "characteristic": "T_JUL",
  "unit": "°C",
  "slope_per_year": 0.043,
  "slope_per_decade": 0.43,
  "estimated_total_change": 1.29,
  "direction": "increase",
  "valid_years": 30,
  "pair_count": 435,
  "sign_consistency": 0.71
}
```

The direction is `increase`, `decrease`, `stable` or `unknown`, depending on the slope and the selected `trend_threshold_per_decade`.

### 13.3 Trends of climate zones

Climate-zone trend endpoints calculate a trend from the annual 11-indicator vectors. They are separate from individual indicator trends.

Endpoint pattern:

```text
POST /climaticzones/trend/point
POST /climaticzones/trend/grid
```

The request can select the trend metric using:

```json
{
  "trend_metric": "expected"
}
```

or:

```json
{
  "trend_metric": "exceedance"
}
```

The accepted metric names are:

```text
expected, expected_fuzzy_rank,
exceedance, exceedance_rank_equivalent
```

For every year, the 11 annual indicators are first interpolated to the requested point or output-grid point. The selected climate-zone trend metric is then calculated for each year. Sen’s slope is then estimated from the annual metric values.

#### 13.3.1 `expected_fuzzy_rank`

The `expected_fuzzy_rank` metric expresses the annual climate-zone position as a continuous value on the ordered Quitt scale.

The climate units are ordered from cold to warm:

```text
CH1, CH2, ..., CH7, MT1, ..., MT11, T1, ..., T5
```

They are assigned ranks:

```text
CH1 = 0
...
T5 = 22
```

For a given year, the calculation first computes fuzzy scores for all climate units. Instead of using only the winning unit, it calculates a weighted average rank:

```text
expected_rank = sum_u(rank_u * score_u) / sum_u(score_u)
```

This value is more stable than a hard annual class such as `MT7` or `MT8`, especially near zone boundaries.

The trend output then describes movement along the official Quitt scale:

- positive `slope_per_decade` means a shift toward warmer climate units,
- negative `slope_per_decade` means a shift toward colder climate units,
- values near zero indicate a stable position on the scale.

A typical output fragment is:

```json
{
  "method": "sen_slope_yearly",
  "metric": "expected_fuzzy_rank",
  "slope_per_decade": 0.72,
  "slope_unit": "climate_zone_rank_per_decade",
  "estimated_total_change": 2.16,
  "direction": "warmer",
  "strength": "moderate"
}
```

This example means that, over the selected period, the location moves toward warmer Quitt units by about 0.72 rank units per decade.

#### 13.3.2 `exceedance_rank_equivalent`

The `exceedance_rank_equivalent` metric is designed for situations where the official Quitt scale reaches its edge. The warm edge of the scale is `T5`; the cold edge is `CH1`. If a location is already classified near `T5`, it can continue to warm even though no official warmer Quitt unit exists. Similarly, a location could theoretically move beyond the cold edge near `CH1`.

This metric therefore measures how far the annual indicator values exceed the official Quitt scale at either edge and expresses that exceedance as a synthetic rank-equivalent value.

For each indicator, the algorithm determines whether increasing or decreasing values represent movement toward warmer zones. Examples:

- for `SUM25`, `VEG10`, `T_JAN`, `T_APR`, `T_JUL` and `T_OCT`, increasing values generally move toward warmer zones,
- for `FROST0` and `ICE0`, decreasing values generally move toward warmer zones,
- precipitation indicators are handled according to their position in the cold-to-warm Quitt limit table, not by a fixed hard-coded assumption.

For every indicator and year, the method derives:

- a cold-edge threshold from `CH1`,
- a warm-edge threshold from `T5`,
- a typical one-rank step size from neighbouring climate units,
- a normalized exceedance beyond the warm edge,
- a normalized exceedance beyond the cold edge.

The annual metric is calculated as a signed value:

```text
signed_exceedance = mean(warm_edge_exceedance) - mean(cold_edge_exceedance)
```

Interpretation:

| Value | Meaning |
|---:|---|
| positive | the annual values exceed the warm edge of the official scale more than the cold edge |
| negative | the annual values exceed the cold edge of the official scale more than the warm edge |
| close to 0 | values are inside the official scale or exceed both edges only weakly |

Sen’s slope is then calculated from the annual signed exceedance values.

A typical output fragment is:

```json
{
  "method": "sen_slope_yearly",
  "metric": "exceedance_rank_equivalent",
  "slope_per_decade": 0.67,
  "slope_unit": "climate_zone_rank_equivalent_per_decade",
  "estimated_total_change": 2.0,
  "direction": "beyond_warm_edge_increase",
  "strength": "strong",
  "scale": {
    "reference_cold_zone": "CH1",
    "reference_warm_zone": "T5",
    "is_beyond_official_quitt_scale_metric": true
  }
}
```

This does **not** mean that a new official Quitt unit such as `T7` exists. It means that the trend corresponds to approximately two synthetic rank-equivalent units beyond the warm edge of the official scale.

The same logic also works in the opposite direction. A negative trend may indicate increasing exceedance beyond the cold edge of the official scale.

### 13.4 Optional annual series in trend output

Trend endpoints may optionally include annual values used for the trend calculation. This is controlled by:

```json
{
  "include_yearly_series": true
}
```

For large grid outputs this can substantially increase the response size. For map layers, it is usually preferable to keep `include_yearly_series` set to `false` and map only the final trend fields.

## 14. Difference between climate-zone endpoints and climate-characteristic endpoints

The API distinguishes several groups of outputs:

1. **Climate characteristics – multi-year values**  
   These return indicator values themselves, for example the average number of summer days or precipitation totals. Area endpoints can return either all 11 indicators together or one selected indicator.

2. **Climate characteristics – trends**  
   These return the Sen-slope trend of one selected annual indicator, for a point or output grid.

3. **Climate zones – static classification**  
   These return the result of comparing the 11 multi-year averaged characteristics with the Quitt limit table, i.e. `best_unit` and `best_score`.

4. **Climate zones – trends**  
   These return a trend derived from annual 11-indicator vectors, using either `expected_fuzzy_rank` or `exceedance_rank_equivalent`.

Climate zones are therefore derived outputs built on top of climate characteristics. Climate-zone trends are derived from annual climate-characteristic values, not from a single multi-year average.

## 15. Checks and validation rules

The implementation performs, in particular, the following checks:

- existence and readability of the Rasdaman registry,
- existence of the selected Rasdaman collection in the registry,
- correct collection structure `[year, ind, y, x]`,
- indicator-axis range `0:10`,
- availability of the requested years,
- at least two years for trend endpoints,
- spatial extent of the point or `bbox`,
- numeric validity of parameters `lat`, `lon`, `step_lat`, `step_lon`, `idw_power`, `idw_k`, `idw_radius_px`, `trend_threshold_per_decade` and related optional parameters,
- existence and readability of the Quitt limit table.

If the requested temporal or spatial extent lies outside the data, the API returns a validation error instead of silently clipping the result.

## 16. Limitations and interpretation of results

The results should be interpreted with the following limitations in mind:

1. **Only 11 of the original Quitt characteristics are used**  
   The original Quitt scheme works with a broader set of characteristics. This implementation uses the 11 characteristics available in the data and computation pipeline.

2. **Algorithmic classification, not a manually generalised map**  
   The output is the result of a repeatable calculation over gridded data. It may differ from the published cartographic map, which may include generalisation, expert interpretation, or different weighting of groups of variables.

3. **The fuzzy score is not a probability**  
   `best_score` expresses similarity to intervals, not the probability of occurrence of a climate unit.

4. **Results near zone boundaries are more sensitive**  
   If a location lies on a transition between climate units, small changes in period, dataset, or interpolation can change the resulting unit code. Trend metric `expected_fuzzy_rank` reduces, but does not completely remove, this sensitivity.

5. **The result depends on the selected data collection and period**  
   The same location may be classified differently for historical data, reanalysis, or a climate scenario.

6. **Precipitation and temperature indicators have different spatial uncertainties**  
   Some variables, especially precipitation and characteristics related to snow cover or cloudiness, are usually more spatially uncertain than temperature characteristics. Because this implementation omits some characteristics, it should not be treated as a full reconstruction of the original Quitt classification.

7. **Trend direction is not automatically an impact assessment**  
   A warmer or drier trend may be interpreted as adverse in many climate-risk applications, but the API trend metrics themselves are descriptive. Terms such as `warmer`, `cooler`, `increase`, `decrease`, `beyond_warm_edge_increase` and `beyond_cold_edge_increase` describe the direction of change, not a universal value judgement.

8. **Exceedance is a synthetic metric beyond the official scale**  
   `exceedance_rank_equivalent` is useful for avoiding edge saturation at `T5` or `CH1`, but it does not define new official Quitt units.

## 17. Recommended way to cite this methodology in the API

In an OpenAPI specification, this methodology can be linked as follows:

```yaml
externalDocs:
  description: Methodology for calculating climate zones according to Quitt’s classification
  url: https://<namespace>.gitlab.io/<project>/methodology_climate_zones_quitt/
```

For a specific endpoint:

```yaml
paths:
  /climaticzones/grid:
    post:
      summary: Calculate climate zones for an area or grid
      externalDocs:
        description: Detailed climate-zone calculation methodology
        url: https://<namespace>.gitlab.io/<project>/methodology_climate_zones_quitt/
```

The same methodology should also be linked from trend endpoints such as `/climaticzones/trend/grid`, because those endpoints use the same indicator set, IDW interpolation and Quitt limit table.

If the document is not published through GitLab Pages and is only stored in the repository, it is possible to link directly to the Markdown file in GitLab. For a public API, however, a readable HTML version published through GitLab Pages is preferable.

## 18. Implementation-related files

| File | Role in the calculation |
|---|---|
| `FWF5_API.py` | Flask API and public endpoints |
| `FWF5_trends.py` | trend calculations for climate characteristics and climate zones |
| `rasdaman_registry.json` | registry of Rasdaman collections, time axes, grids and coordinates |
| `FWF5_quitt_limits_11_rasdaman_index.json` | Quitt unit limit table for the 11 indicators |
| `FWF5_climate_metadata.json` | descriptions of climate characteristics and zones |
| `FWF5_quitt_fuzzy_match_point.py` | point calculation of the climate unit |
| `FWF5_quitt_fuzzy_match_area_custom_grid.py` | climate-unit calculation for the native or custom grid |
| `FWF5_yindicators_custom_grid_means.py` | export of multi-year averages of the 11 climate characteristics |
| `FWF5_yindicators_yearly_point.py` | export of annual values of the 11 climate characteristics for a point |
| `grid_coords_universal.py` | grid-coordinate handling, spatial subsets and IDW interpolation |

## 19. Reproducibility

To make the result reproducible, published outputs should specify:

- the name of the Rasdaman collection used,
- the version of the Rasdaman registry,
- the version of the Quitt interval limit table,
- the start and end year,
- the spatial extent used,
- whether the calculation was performed on the native or custom grid,
- the values of `idw_power`, `idw_k`, `idw_radius_px` and, if used, `idw_max_dist_m`,
- for trend outputs, the trend method and metric, for example `sen_slope_yearly` and `expected_fuzzy_rank`,
- the value of `trend_threshold_per_decade`, if a non-default threshold is used,
- whether annual series were included in the output,
- the API version or repository commit.

Recommended minimum metadata record for static climate-zone calculation:

```json
{
  "method": "quitt_fuzzy_match_11_indicators",
  "rasdaman_collection": "era5l_cz_yindicators",
  "start_year": 1991,
  "end_year": 2020,
  "quitt_limits": "FWF5_quitt_limits_11_rasdaman_index.json",
  "idw": {
    "power": 2.0,
    "k_nearest": 9,
    "radius_px": 2,
    "max_dist_m": null
  },
  "output": "GeoJSON FeatureCollection"
}
```

Recommended minimum metadata record for climate-zone trend calculation:

```json
{
  "method": "sen_slope_yearly",
  "trend_metric": "expected_fuzzy_rank",
  "rasdaman_collection": "era5l_cz_yindicators",
  "start_year": 1991,
  "end_year": 2020,
  "quitt_limits": "FWF5_quitt_limits_11_rasdaman_index.json",
  "idw": {
    "power": 2.0,
    "k_nearest": 9,
    "radius_px": 2,
    "max_dist_m": null
  },
  "trend_threshold_per_decade": 0.25,
  "include_yearly_series": false,
  "output": "GeoJSON FeatureCollection"
}
```

For exceedance-based climate-zone trend, set:

```json
{
  "trend_metric": "exceedance_rank_equivalent"
}
```

## 20. One-sentence methodology summary

A static climate zone is determined by calculating or interpolating 11 multi-year climate characteristics for a selected location and period, comparing them with the intervals of 23 Quitt climate units, and selecting the unit with the highest fuzzy similarity score; trend endpoints use annual values for all years in the selected period and estimate Sen-slope trends either for individual indicators or for climate-zone metrics such as `expected_fuzzy_rank` and `exceedance_rank_equivalent`.
