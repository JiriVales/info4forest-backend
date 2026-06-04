# Metodika výpočtu klimatických zón inspirovaných Quittovou klasifikací

**Účel dokumentu:** metodický popis výpočtu klimatických zón používaný v API. Dokument je určen k publikaci v GitLabu/GitLab Pages a k odkázání z OpenAPI/Swagger dokumentace pomocí `externalDocs`.

**Verze dokumentu:** 1.1  
**Datum:** 2026-06-04  
**Metodický základ:** E. Quittova klimatická klasifikace a metodika popsaná v publikaci *Klimatické oblasti Česka: klasifikace podle Quitta za období 1961–2000* vydané Univerzitou Palackého v Olomouci a ČHMÚ.  
**Odkaz na publikaci:** <https://www.cartography.upol.cz/MAPS/MAPS_Num3_brozura.pdf>

---

## 1. Stručné vysvětlení pro laiky

Výpočet klimatické zóny funguje podobně, jako kdybychom pro každé místo vyplnili klimatický dotazník a pak hledali, ke které známé klimatické oblasti se místo nejvíce podobá.

Pro každé místo a zvolené období, například 1991–2020, se nejprve vezme 11 klimatických charakteristik. Patří mezi ně například počet letních dnů, počet mrazových dnů, průměrná teplota v lednu, dubnu, červenci a říjnu nebo množství srážek ve vegetačním a zimním období. Tyto charakteristiky popisují, jestli je místo spíše teplé nebo chladné, suché nebo vlhké a jak výrazná je zima a léto.

Potom se pro každou charakteristiku spočítá průměr za celé vybrané období. Pokud uživatel zadá konkrétní bod, který neleží přesně ve středu datové buňky, hodnoty se dopočítají z okolních buněk metodou IDW, tedy váženým průměrem, kde bližší buňky mají větší váhu než vzdálenější.

Následně se výsledných 11 hodnot porovná s tabulkou Quittových klimatických jednotek. Quittova klasifikace rozlišuje klimatické jednotky typu `CH1` až `CH7` pro chladnou oblast, `MT1` až `MT11` pro mírně teplou oblast a `T1` až `T5` pro teplou oblast. Každá jednotka má předepsané intervaly hodnot, například kolik má mít letních dnů nebo jaká má být průměrná červencová teplota.

Protože reálná data málokdy přesně zapadnou do jedné jediné jednotky, výpočet nepoužívá pouze tvrdé pravidlo „spadá / nespadá“. Místo toho používá podobnostní skóre od 0 do 1. Hodnota 1 znamená, že daná charakteristika leží přesně v intervalu dané klimatické jednotky. Pokud je hodnota těsně mimo interval, dostane částečné skóre. Pokud je příliš daleko, dostane skóre 0. Celkové skóre klimatické jednotky je průměr skóre přes všech 11 charakteristik.

Výsledná klimatická zóna je ta jednotka, která dosáhne nejvyššího skóre. Pokud dvě jednotky vyjdou stejně dobře, výpočet upřednostní teplejší jednotku. To odpovídá principu popsanému v metodice UPOL u digitálního přiřazování podle maxima splněných kritérií.

Výstupem API je GeoJSON. U každého bodu je uveden kód nejlepší klimatické jednotky, například `MT7`, a hodnota `best_score`, která říká, jak dobře dané místo odpovídá vybrané jednotce. Skóre není pravděpodobnost, ale míra podobnosti k intervalům Quittovy tabulky.


Kromě statického výpočtu klimatické zóny pro zvolené víceleté období API podporuje také trendové výpočty. Trendové endpointy neporovnávají pouze první a poslední rok. Pracují s ročními hodnotami všech let ve zvoleném období a odhadují robustní Senovu směrnici. Výsledek je proto méně citlivý na jednotlivé anomální roky. Trendy lze počítat buď pro jednotlivé klimatické charakteristiky, například červencovou teplotu nebo srážky ve vegetačním období, nebo pro klimatické zóny. Trend klimatických zón podporuje dvě metriky: `expected_fuzzy_rank` a `exceedance_rank_equivalent`.

---

## 2. Vztah k Quittově a UPOL metodice

Quittova klimatická klasifikace je klasifikací komplexní klimatologie. Území se nezařazuje podle jedné veličiny, například jen podle průměrné teploty, ale podle kombinace více klimatologických charakteristik. Původní Quittovo schéma pracuje s 23 klimatickými jednotkami ve třech základních oblastech:

- **CH** – chladná oblast (`CH1` až `CH7`),
- **MT** – mírně teplá oblast (`MT1` až `MT11`),
- **T** – teplá oblast (`T1` až `T5`).

Původní Quittova klasifikace podle publikace UPOL/ČHMÚ rozlišuje 23 jednotek definovaných kombinacemi hodnot 14 klimatologických charakteristik. Hranice jednotek jsou dány změnami těchto charakteristik a jednotlivé intervaly se mohou částečně překrývat. Publikace UPOL upozorňuje, že praktická realizace klasifikace je zatížena neurčitostí, protože jedno místo může splňovat znaky více jednotek současně.

Tato implementace je **inspirována Quittovou klasifikací a metodikou UPOL**, ale není přímou reprodukcí původní mapy ani ruční kartografické interpretace. Výpočet je algoritmický, opakovatelný a založený na předpočtených klimatických indikátorech v rastru. Oproti původním 14 charakteristikám používá implementace 11 dostupných charakteristik. Metadata limitní tabulky uvádějí jako nepoužité charakteristiky kódy `PDZAT`, `PDJAS`, `PDSCE` a `PDOBL`.

Hlavní rozdíl oproti jednoduché metodě „maximum splněných kritérií“ je ten, že tato implementace používá fuzzy skórování. Místo aby každá charakteristika dala pouze hodnotu 0 nebo 1, hodnota těsně mimo interval dostane částečnou shodu. Tím je výpočet stabilnější v případech, kdy se hodnoty nacházejí blízko hranice klimatických jednotek.

---

## 3. Vstupní data

Výpočet pracuje s předpočtenou rasdaman kolekcí ročních klimatických indikátorů. Kolekce má čtyři dimenze:

```text
[year, ind, y, x]
```

kde:

- `year` je kalendářní rok,
- `ind` je index klimatické charakteristiky 0 až 10,
- `y` je index řádku prostorové mřížky,
- `x` je index sloupce prostorové mřížky.

Příklad produkční kolekce v přiloženém registru:

```text
era5l_cz_yindicators
```

Tato kolekce je popsaná jako roční předpočtené klimatické indikátory na české mřížce, uložené jako 4D pole `[year, ind, y, x]`. Pro výpočty scénářových nebo modelových dat lze použít i jiné kolekce stejného typu, pokud mají stejnou strukturu indikátorů.

Prostorová mřížka je definována v souboru `rasdaman_registry.json`. Registr obsahuje metadata kolekcí, časových os, prostorových mřížek, souřadnic a napojení na rasdaman.

---

## 4. Použitých 11 klimatických charakteristik

| Index `ind` | Kód | Český význam | Jednotka |
|---:|---|---|---|
| 0 | `SUM25` | Roční počet letních dnů, tj. dnů s maximální teplotou alespoň 25 °C | dny |
| 1 | `VEG10` | Roční počet dnů s průměrnou teplotou alespoň 10 °C | dny |
| 2 | `FROST0` | Roční počet mrazových dnů, tj. dnů s minimální teplotou pod 0 °C | dny |
| 3 | `ICE0` | Roční počet ledových dnů, tj. dnů s maximální teplotou pod 0 °C | dny |
| 4 | `T_JAN` | Průměrná teplota v lednu | °C |
| 5 | `T_APR` | Průměrná teplota v dubnu | °C |
| 6 | `T_JUL` | Průměrná teplota v červenci | °C |
| 7 | `T_OCT` | Průměrná teplota v říjnu | °C |
| 8 | `RAIN1` | Roční počet srážkových dnů se srážkami alespoň 1 mm | dny |
| 9 | `P_APR_SEP` | Úhrn srážek za duben až září, tedy vegetační období | mm |
| 10 | `P_OCT_MAR` | Úhrn srážek za říjen až březen, tedy zimní období | mm |

Pořadí indikátorů je důležité, protože odpovídá ose `ind` v rasdaman kolekci i mapování v souboru `FWF5_quitt_limits_11_rasdaman_index.json`.

---

## 5. Přehled výpočetního postupu

Celý výpočet lze shrnout do těchto kroků:

1. **Výběr kolekce a období**  
   Uživatel zadá rasdaman kolekci, počáteční rok a koncový rok. API ověří, že požadované roky jsou dostupné.

2. **Výběr místa nebo území**  
   Uživatel zadá buď bod (`lat`, `lon`), nebo prostorový rozsah (`bbox`). U mřížkového výstupu může zadat také krok výstupní mřížky `step_lat` a `step_lon`.

3. **Načtení ročních indikátorů z Rasdamanu**  
   Z kolekce se načte blok dat pro zadané roky, indikátory 0 až 10 a příslušné prostorové okno.

4. **Výpočet víceletých průměrů**  
   Hodnoty indikátorů se sečtou přes roky a vydělí počtem let. Vznikne pole průměrných indikátorů pro celé období.

5. **Interpolace na zadaný bod nebo vlastní výstupní mřížku**  
   Pokud je výstup počítán mimo původní středy buněk, hodnoty 11 indikátorů se dopočítají metodou IDW z okolních rasdaman buněk.

6. **Porovnání s Quittovou limitní tabulkou**  
   Pro každou z 23 klimatických jednotek se spočítá míra shody každého indikátoru s intervalem dané jednotky.

7. **Výběr nejlepší klimatické jednotky**  
   Pro každou jednotku se spočítá průměrné fuzzy skóre. Vybere se jednotka s nejvyšším skóre. Při shodě vyhrává teplejší jednotka.

8. **Vytvoření výstupu**  
   Výsledkem je GeoJSON s bodovou geometrií. Vlastnosti obsahují minimálně `best_unit` a `best_score`; u bodového výpočtu také průměrné hodnoty všech 11 indikátorů. Trendové endpointy používají stejnou strukturu GeoJSON, ale jejich vlastnosti obsahují trendové metriky odvozené z ročních hodnot.

---

## 6. Výpočet víceletých průměrů

Nechť:

- `i` je index klimatické charakteristiky,
- `y` je rok,
- `s` je buňka rasdaman mřížky,
- `I_i(y, s)` je roční hodnota indikátoru `i` v roce `y` a buňce `s`,
- `Y` je množina let od `start_year` do `end_year` včetně,
- `n` je počet let ve výběru.

Víceletý průměr indikátoru se počítá jako:

```text
mean_i(s) = (1 / n) * sum_{y in Y} I_i(y, s)
```

V kódu se pro efektivitu používá rasdaman dotaz typu `condense + over yr`, který sečte vybrané roky přímo na straně datového úložiště. Python potom vydělí součet počtem let.

Výpočet předpokládá, že rasdaman kolekce obsahuje kompletní pokrytí požadovaného období. Pokud některý požadovaný rok v kolekci chybí, výpočet se zastaví chybou. Numerické chybějící hodnoty jsou reprezentovány jako `NaN` a mohou ovlivnit následnou interpolaci nebo skórování.

---

## 7. Prostorová práce s mřížkou

Souřadnice mřížky se načítají univerzálním modulem `grid_coords_universal.py`. Implementace podporuje dva typy mřížek:

1. **Separable 1D grid**  
   Zeměpisné délky a šířky lze vytvořit jako dvě samostatná 1D pole, typicky pomocí `linspace`. Příkladem je česká ERA5-Land mřížka s pravidelným krokem.

2. **Lookup 2D grid**  
   Zeměpisná šířka a délka jsou uloženy jako 2D souřadnicové vrstvy. Tento režim je vhodný například pro rotované regionální klimatické mřížky.

Pro bodový výpočet se nejprve najde nejbližší mřížková buňka a kolem ní se vybere kandidátní okno. Pro mřížkový výpočet se z požadovaného území stanoví prostorový rozsah dat, který je potřeba načíst z Rasdamanu.

---

## 8. IDW interpolace

Pokud požadovaný bod nebo bod vlastní výstupní mřížky neleží přesně ve středu rasdaman buňky, dopočítají se hodnoty 11 indikátorů metodou IDW, tedy inverse distance weighting.

Nechť:

- `p` je požadovaný bod,
- `g_j` jsou okolní rasdaman body,
- `d_j` je vzdálenost mezi `p` a `g_j`, počítaná haversinovou vzdáleností,
- `p_idw` je parametr mocniny IDW, výchozí hodnota je `2.0`,
- `k` je maximální počet použitých sousedů, výchozí hodnota je `9`.

Váha souseda je:

```text
w_j = 1 / d_j^p_idw
```

Interpolovaná hodnota indikátoru je:

```text
I_hat_i(p) = sum_j(w_j * mean_i(g_j)) / sum_j(w_j)
```

Pokud požadovaný bod leží přesně ve středu některé mřížkové buňky, interpolace se nepoužije a vrátí se hodnota této buňky.

Výchozí nastavení:

| Parametr | Význam | Výchozí hodnota |
|---|---|---:|
| `idw_power` | exponent vážení vzdáleností | `2.0` |
| `idw_k` | maximální počet nejbližších sousedů pro mřížkový výpočet | `9` |
| `idw_radius_px` | poloměr kandidátního okna v pixelech | u API zpravidla `2` |
| `idw_max_dist_m` | volitelný maximální dosah souseda v metrech | `null` |

Čím vyšší je `idw_power`, tím větší vliv má nejbližší mřížková buňka. Čím nižší je `idw_power`, tím více se výsledek blíží běžnému průměru okolních buněk.

---

## 9. Limitní tabulka Quittových jednotek

Limitní hodnoty jsou uloženy v souboru:

```text
FWF5_quitt_limits_11_rasdaman_index.json
```

Soubor obsahuje:

- zdroj limitů odvozený z Quittovy tabulky klimatických charakteristik,
- seznam použitých 11 indikátorů,
- jednotky indikátorů,
- pořadí klimatických jednotek od chladných po teplé,
- mapování indikátorů na osu `ind` v rasdaman kolekci,
- intervaly hodnot pro každou klimatickou jednotku.

Klimatické jednotky jsou vyhodnocovány v tomto pořadí:

```text
CH1, CH2, CH3, CH4, CH5, CH6, CH7,
MT1, MT2, MT3, MT4, MT5, MT6, MT7, MT8, MT9, MT10, MT11,
T1, T2, T3, T4, T5
```

Toto pořadí je důležité i pro řešení shodných výsledků. Protože kód při stejném skóre přepíše předchozí jednotku pozdější jednotkou v pořadí, vyhrává při shodě teplejší jednotka.

---

## 10. Fuzzy skórování klimatických jednotek

Pro každou klimatickou jednotku `u` a každý indikátor `i` existuje interval:

```text
[lower_{u,i}, upper_{u,i}]
```

Hodnota indikátoru `v` dostane dílčí skóre `mu_{u,i}(v)`:

- pokud `v` leží uvnitř intervalu, skóre je `1`,
- pokud `v` leží pod dolní hranicí, skóre lineárně klesá podle vzdálenosti od dolní hranice,
- pokud `v` leží nad horní hranicí, skóre lineárně klesá podle vzdálenosti od horní hranice,
- pokud je hodnota dál než toleranční pásmo, skóre je `0`.

Formálně:

```text
mu = 1                                      pokud lower <= v <= upper
mu = max(0, 1 - (lower - v) / tolerance)    pokud v < lower
mu = max(0, 1 - (v - upper) / tolerance)    pokud v > upper
```

Použité tolerance jsou:

| Skupina indikátorů | Indikátory | Tolerance |
|---|---|---:|
| Počty dnů | `SUM25`, `VEG10`, `FROST0`, `ICE0`, `RAIN1` | 10 dnů |
| Teploty | `T_JAN`, `T_APR`, `T_JUL`, `T_OCT` | 1 °C |
| Srážkové úhrny | `P_APR_SEP`, `P_OCT_MAR` | 50 mm |

Celkové skóre jednotky je aritmetický průměr dílčích skóre přes všechny dostupné indikátory:

```text
score_u = mean_i(mu_{u,i})
```

Výsledná jednotka je:

```text
best_unit = argmax_u(score_u)
```

Hodnota `best_score` je příslušné maximální skóre. Interpretace:

| `best_score` | Význam |
|---:|---|
| blízko `1.0` | místo velmi dobře odpovídá intervalům vybrané jednotky |
| přibližně `0.5–0.8` | místo odpovídá jednotce částečně; může ležet poblíž hranic více jednotek |
| nízké skóre | výsledek je nutné interpretovat opatrně; nejlepší jednotka je pouze nejbližší z dostupných možností |

`best_score` není statistická pravděpodobnost. Je to algoritmická míra podobnosti k limitním intervalům.

---

## 11. Výpočet pro bod

Bodový výpočet se používá pro endpoint typu:

```text
POST /climaticzones/point
```

Typický vstup:

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

Postup:

1. API ověří, že bod leží v prostorovém rozsahu zvolené kolekce.
2. Najde se nejbližší mřížková buňka.
3. Načte se malé okolní okno z rasdaman kolekce.
4. Pro každou z 11 charakteristik se spočítá víceletý průměr.
5. Pokud bod neleží přesně ve středu buňky, hodnoty se interpolují metodou IDW.
6. Interpolovaných 11 hodnot se porovná s Quittovou limitní tabulkou.
7. Vrátí se GeoJSON s jedním bodem.

Typický výstup obsahuje:

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

Hodnoty v příkladu jsou ilustrativní.

---

## 12. Výpočet pro území nebo výstupní mřížku

Mřížkový výpočet se používá pro endpoint typu:

```text
POST /climaticzones/grid
```

### 12.1 Režim nativní mřížky

Pokud nejsou zadány `step_lat` a `step_lon`, výpočet pracuje přímo s body nativní rasdaman mřížky. Volitelně lze zadat `bbox`, který výstup omezí na konkrétní území.

V tomto režimu se:

1. načte blok hodnot z rasdaman kolekce,
2. spočítají se víceleté průměry pro každou nativní buňku,
3. pro každou buňku se provede fuzzy porovnání s Quittovou tabulkou,
4. výsledkem je GeoJSON s body ve středech rasdaman buněk.

### 12.2 Režim vlastní výstupní mřížky

Pokud jsou zadány `step_lat` a `step_lon`, API vytvoří vlastní pravidelnou výstupní mřížku v zeměpisných souřadnicích. Oba parametry musí být zadány současně.

Typický vstup:

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

V tomto režimu se:

1. vytvoří seznam cílových souřadnic podle `bbox`, `step_lat` a `step_lon`,
2. z rasdaman kolekce se načte rozšířené okolí potřebné pro IDW,
3. spočítají se víceleté průměry v nativních buňkách,
4. pro každý cílový bod se 11 indikátorů interpoluje metodou IDW,
5. pro každý cílový bod se provede fuzzy klasifikace,
6. výsledkem je GeoJSON s body vlastní výstupní mřížky.

---

## 13. Trendové výpočty

Trendové výpočty jsou oddělené od statického výpočtu klimatické zóny za víceleté období. Používají se tehdy, když má být popsáno, jak se klimatická charakteristika nebo klimatická zóna ve zvoleném období vyvíjí v čase.

Základní rozdíl je v tom, že trendové endpointy používají roční časovou řadu všech let od `start_year` do `end_year` včetně. Trend se tedy neodvozuje pouze z prvního a posledního roku, protože jednotlivé roky mohou být anomálně teplé, chladné, suché nebo vlhké.

Pro každý požadovaný bod nebo bod výstupní mřížky je postup následující:

1. načtou se roční hodnoty 11 indikátorů pro všechny požadované roky,
2. roční hodnoty se interpolují do požadovaného bodu stejnou IDW logikou jako u ostatních endpointů,
3. podle typu endpointu a zvolené metriky se odvodí roční trendová veličina,
4. trend se odhadne pomocí Senovy směrnice,
5. výsledek se vrátí jako vlastnosti GeoJSON prvků.

Trendová metoda je ve výstupu označena jako:

```text
sen_slope_yearly
```

Pro výpočet trendu jsou potřeba alespoň dva platné roky.

### 13.1 Senova směrnice

Senova směrnice je robustní neparametrický odhad trendu. Pro všechny platné dvojice let `(a, b)`, kde `b > a`, se spočítá dílčí sklon:

```text
slope_ab = (value_b - value_a) / (year_b - year_a)
```

Výsledný sklon je medián všech párových sklonů:

```text
slope_per_year = median(slope_ab)
```

API dále uvádí:

```text
slope_per_decade = 10 * slope_per_year
estimated_total_change = slope_per_year * (end_year - start_year)
```

Tento postup omezuje vliv jednotlivých anomálních roků ve srovnání s prostým rozdílem první rok / poslední rok.

Ve výstupu se mohou objevit také tyto položky:

| Položka | Význam |
|---|---|
| `valid_years` | počet let s platnými hodnotami použitými pro trend |
| `pair_count` | počet platných dvojic let použitých pro Senovu směrnici |
| `sign_consistency` | podíl nenulových párových sklonů s převládajícím znaménkem |
| `direction` | slovní směr trendu podle `slope_per_decade` a prahové hodnoty |
| `strength` | slovní síla trendu odvozená z absolutní hodnoty sklonu |

Položka `sign_consistency` není formální test statistické významnosti. Je to jednoduchá diagnostická informace, zda většina párových sklonů ukazuje stejným směrem.

### 13.2 Trendy jednotlivých klimatických charakteristik

Endpointy pro trend klimatických charakteristik počítají trend vždy pouze pro jeden vybraný indikátor. Nepočítají trend klimatických zón a nevracejí všech 11 indikátorů najednou.

Typický vzor endpointů:

```text
POST /climatecharacteristics/trend/<characteristic>/point
POST /climatecharacteristics/trend/<characteristic>/grid
```

kde `<characteristic>` může být:

```text
sum25, veg10, frost0, ice0,
t_jan, t_apr, t_jul, t_oct,
rain1, p_apr_sep, p_oct_mar
```

Například:

```text
POST /climatecharacteristics/trend/t_jul/point
POST /climatecharacteristics/trend/p_apr_sep/grid
```

Roční hodnota vybraného indikátoru se použije přímo jako trendová veličina. Výsledný sklon má tedy fyzikální jednotku daného indikátoru, například:

- dny za dekádu pro `SUM25`, `VEG10`, `FROST0`, `ICE0` a `RAIN1`,
- °C za dekádu pro `T_JAN`, `T_APR`, `T_JUL` a `T_OCT`,
- mm za dekádu pro `P_APR_SEP` a `P_OCT_MAR`.

Typický objekt trendu pro jeden indikátor:

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

Směr je `increase`, `decrease`, `stable` nebo `unknown` podle sklonu a zadané hodnoty `trend_threshold_per_decade`.

### 13.3 Trendy klimatických zón

Endpointy pro trend klimatických zón počítají trend z ročních vektorů 11 indikátorů. Jsou oddělené od trendů jednotlivých indikátorů.

Vzor endpointů:

```text
POST /climaticzones/trend/point
POST /climaticzones/trend/grid
```

V požadavku lze zvolit trendovou metriku pomocí:

```json
{
  "trend_metric": "expected"
}
```

nebo:

```json
{
  "trend_metric": "exceedance"
}
```

Povolené názvy metrik jsou:

```text
expected, expected_fuzzy_rank,
exceedance, exceedance_rank_equivalent
```

Pro každý rok se nejprve 11 ročních indikátorů interpoluje do požadovaného bodu nebo bodu výstupní mřížky. Potom se pro každý rok spočítá vybraná trendová metrika klimatické zóny. Z roční řady této metriky se následně odhadne Senova směrnice.

#### 13.3.1 `expected_fuzzy_rank`

Metrika `expected_fuzzy_rank` vyjadřuje roční pozici klimatické zóny jako spojitou hodnotu na seřazené Quittově škále.

Klimatické jednotky jsou seřazeny od chladných po teplé:

```text
CH1, CH2, ..., CH7, MT1, ..., MT11, T1, ..., T5
```

Dostanou pořadové ranky:

```text
CH1 = 0
...
T5 = 22
```

Pro daný rok se nejprve spočítají fuzzy skóre všech klimatických jednotek. Místo použití pouze vítězné jednotky se vypočte vážený průměr ranku:

```text
expected_rank = sum_u(rank_u * score_u) / sum_u(score_u)
```

Tato hodnota je stabilnější než tvrdá roční třída typu `MT7` nebo `MT8`, zejména poblíž hranic klimatických zón.

Trend potom popisuje pohyb po oficiální Quittově škále:

- kladný `slope_per_decade` znamená posun k teplejším klimatickým jednotkám,
- záporný `slope_per_decade` znamená posun k chladnějším klimatickým jednotkám,
- hodnoty blízké nule znamenají stabilní pozici na škále.

Typická část výstupu:

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

Tento příklad znamená, že se dané místo ve zvoleném období posouvá směrem k teplejším Quittovým jednotkám tempem přibližně 0,72 ranku za dekádu.

#### 13.3.2 `exceedance_rank_equivalent`

Metrika `exceedance_rank_equivalent` řeší situace, kdy oficiální Quittova škála narazí na svůj okraj. Teplým okrajem škály je `T5`, chladným okrajem je `CH1`. Pokud je místo už klasifikováno poblíž `T5`, může se dále oteplovat, i když žádná oficiální teplejší Quittova jednotka neexistuje. Stejně tak se může teoreticky posouvat za chladný okraj poblíž `CH1`.

Tato metrika proto měří, nakolik roční hodnoty indikátorů přesahují oficiální Quittovu škálu na některém z jejích okrajů, a tento přesah vyjadřuje jako syntetický rankový ekvivalent.

Pro každý indikátor algoritmus určí, zda posun k teplejším zónám znamená růst nebo pokles hodnoty. Příklady:

- u `SUM25`, `VEG10`, `T_JAN`, `T_APR`, `T_JUL` a `T_OCT` obecně posouvá růst hodnoty indikátoru směrem k teplejším zónám,
- u `FROST0` a `ICE0` obecně posouvá pokles hodnoty indikátoru směrem k teplejším zónám,
- srážkové indikátory se vyhodnocují podle své polohy v Quittově limitní tabulce od chladných po teplé jednotky, nikoli podle pevně zakódovaného předpokladu.

Pro každý indikátor a rok se odvodí:

- hranice chladného okraje z jednotky `CH1`,
- hranice teplého okraje z jednotky `T5`,
- typický krok jedné klimatické jednotky z rozdílů sousedních jednotek,
- normalizovaný přesah za teplý okraj,
- normalizovaný přesah za chladný okraj.

Roční metrika se počítá jako znaménková hodnota:

```text
signed_exceedance = mean(warm_edge_exceedance) - mean(cold_edge_exceedance)
```

Interpretace:

| Hodnota | Význam |
|---:|---|
| kladná | roční hodnoty více přesahují teplý okraj oficiální škály |
| záporná | roční hodnoty více přesahují chladný okraj oficiální škály |
| blízká nule | hodnoty jsou uvnitř oficiální škály nebo přesahují okraje pouze slabě |

Ze série ročních znaménkových přesahů se následně spočítá Senova směrnice.

Typická část výstupu:

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

To **neznamená**, že existuje nová oficiální Quittova jednotka například `T7`. Znamená to, že trend odpovídá přibližně dvěma syntetickým rankovým ekvivalentům za teplým okrajem oficiální škály.

Stejná logika funguje i opačným směrem. Záporný trend může znamenat rostoucí přesah za chladný okraj oficiální škály.

### 13.4 Volitelná roční série ve výstupu trendu

Trendové endpointy mohou volitelně vracet roční hodnoty použité pro výpočet trendu. Řídí se to parametrem:

```json
{
  "include_yearly_series": true
}
```

U velkých mřížkových výstupů to může výrazně zvětšit odpověď. Pro mapové vrstvy je obvykle vhodnější ponechat `include_yearly_series` na hodnotě `false` a mapovat pouze výsledné trendové položky.

## 14. Rozdíl mezi endpointy klimatických zón a klimatických charakteristik

API rozlišuje několik skupin výstupů:

1. **Klimatické charakteristiky – víceleté hodnoty**  
   Vrací samotné hodnoty indikátorů, například průměrný počet letních dnů nebo úhrn srážek. Plošné endpointy mohou vracet buď všech 11 indikátorů najednou, nebo jeden vybraný indikátor.

2. **Klimatické charakteristiky – trendy**  
   Vrací Senův trend jednoho vybraného ročního indikátoru pro bod nebo výstupní mřížku.

3. **Klimatické zóny – statická klasifikace**  
   Vrací výsledek porovnání 11 víceletě zprůměrovaných charakteristik s Quittovou limitní tabulkou, tedy `best_unit` a `best_score`.

4. **Klimatické zóny – trendy**  
   Vrací trend odvozený z ročních vektorů 11 indikátorů, a to buď metrikou `expected_fuzzy_rank`, nebo `exceedance_rank_equivalent`.

Klimatické zóny jsou tedy odvozený výstup nad klimatickými charakteristikami. Trend klimatických zón je odvozený z ročních hodnot klimatických charakteristik, nikoli z jednoho víceletého průměru.

## 15. Kontroly a validační pravidla

Implementace provádí zejména tyto kontroly:

- existence a čitelnost rasdaman registru,
- existence zadané rasdaman kolekce v registru,
- správná struktura kolekce `[year, ind, y, x]`,
- rozsah indikátorové osy `0:10`,
- dostupnost požadovaných let,
- alespoň dva roky pro trendové endpointy,
- prostorový rozsah bodu nebo `bbox`,
- numerická validita parametrů `lat`, `lon`, `step_lat`, `step_lon`, `idw_power`, `idw_k`, `idw_radius_px`, `trend_threshold_per_decade` a dalších volitelných parametrů,
- existence a čitelnost Quittovy limitní tabulky.

Pokud požadovaný časový nebo prostorový rozsah leží mimo data, API vrací validační chybu místo tichého oříznutí výsledku.

## 16. Omezení a interpretace výsledků

Výsledky je nutné interpretovat s ohledem na následující omezení:

1. **Pouze 11 z původních Quittových charakteristik**  
   Původní Quittovo schéma pracuje s širší sadou charakteristik. Tato implementace používá 11 charakteristik dostupných v datové a výpočetní pipeline.

2. **Algoritmická klasifikace, nikoli ručně generalizovaná mapa**  
   Výstup je výsledkem opakovatelného výpočtu nad rastrovými daty. Nemusí se shodovat s publikovanou kartografickou mapou, která může zahrnovat generalizaci, expertní zásahy nebo odlišné vážení skupin prvků.

3. **Fuzzy skóre není pravděpodobnost**  
   `best_score` vyjadřuje podobnost k intervalům, nikoli pravděpodobnost výskytu klimatické jednotky.

4. **Výsledky u hranic zón jsou citlivější**  
   Pokud místo leží na přechodu mezi klimatickými jednotkami, mohou malé změny období, datové sady nebo interpolace změnit výsledný kód jednotky. Trendová metrika `expected_fuzzy_rank` tuto citlivost snižuje, ale zcela ji neodstraňuje.

5. **Výsledek závisí na zvolené datové kolekci a období**  
   Stejné místo může být klasifikováno jinak pro historická data, reanalýzu nebo klimatický scénář.

6. **Srážkové a teplotní indikátory mají rozdílnou prostorovou nejistotu**  
   Některé prvky, zejména srážky nebo charakteristiky spojené se sněhem a oblačností, bývají prostorově nejistější než teplotní charakteristiky. Tato implementace s vynechanými charakteristikami proto nemá být považována za úplnou rekonstrukci původní Quittovy klasifikace.

7. **Směr trendu není automaticky hodnocení dopadu**  
   Teplejší nebo sušší trend lze v řadě klimatických rizik interpretovat jako nepříznivý, ale samotné API trendové metriky jsou popisné. Výrazy jako `warmer`, `cooler`, `increase`, `decrease`, `beyond_warm_edge_increase` a `beyond_cold_edge_increase` popisují směr změny, nikoli univerzální hodnotový soud.

8. **Exceedance je syntetická metrika za oficiální škálou**  
   `exceedance_rank_equivalent` je užitečná pro odstranění saturace na okrajích `T5` nebo `CH1`, ale nedefinuje nové oficiální Quittovy jednotky.

## 17. Doporučený způsob citace metodiky v API

V OpenAPI specifikaci lze na tuto metodiku odkázat takto:

```yaml
externalDocs:
  description: Metodika výpočtu klimatických zón podle Quittovy klasifikace
  url: https://<namespace>.gitlab.io/<project>/metodika_klimaticke_zony_quitt/
```

U konkrétního endpointu:

```yaml
paths:
  /climaticzones/grid:
    post:
      summary: Výpočet klimatických zón pro území nebo mřížku
      externalDocs:
        description: Detailní metodika výpočtu klimatických zón
        url: https://<namespace>.gitlab.io/<project>/metodika_klimaticke_zony_quitt/
```

Stejnou metodiku je vhodné odkázat také u trendových endpointů, například `/climaticzones/trend/grid`, protože používají stejnou sadu indikátorů, IDW interpolaci a Quittovu limitní tabulku.

Pokud dokument nebude publikován přes GitLab Pages, ale pouze uložen v repozitáři, lze odkázat na Markdown soubor v GitLabu. Pro veřejné API je však vhodnější publikovat čitelnou HTML verzi přes GitLab Pages.

## 18. Soubory související s implementací

| Soubor | Role ve výpočtu |
|---|---|
| `FWF5_API.py` | Flask API a veřejné endpointy |
| `FWF5_trends.py` | trendové výpočty klimatických charakteristik a klimatických zón |
| `rasdaman_registry.json` | registr rasdaman kolekcí, časových os, mřížek a souřadnic |
| `FWF5_quitt_limits_11_rasdaman_index.json` | limitní tabulka Quittových jednotek pro 11 indikátorů |
| `FWF5_climate_metadata.json` | popisy klimatických charakteristik a zón |
| `FWF5_quitt_fuzzy_match_point.py` | bodový výpočet klimatické jednotky |
| `FWF5_quitt_fuzzy_match_area_custom_grid.py` | výpočet klimatických jednotek pro nativní nebo vlastní mřížku |
| `FWF5_yindicators_custom_grid_means.py` | export víceletých průměrů 11 klimatických charakteristik |
| `FWF5_yindicators_yearly_point.py` | export ročních hodnot 11 klimatických charakteristik pro bod |
| `grid_coords_universal.py` | práce se souřadnicemi mřížek, výřezy a IDW interpolací |

## 19. Reprodukovatelnost

Aby byl výsledek reprodukovatelný, je nutné při publikaci výsledků uvádět:

- název použité rasdaman kolekce,
- verzi rasdaman registru,
- verzi limitní tabulky Quittových intervalů,
- počáteční a koncový rok,
- použitý prostorový rozsah,
- zda byl výpočet proveden na nativní nebo vlastní mřížce,
- hodnoty parametrů `idw_power`, `idw_k`, `idw_radius_px` a případně `idw_max_dist_m`,
- u trendových výstupů trendovou metodu a metriku, například `sen_slope_yearly` a `expected_fuzzy_rank`,
- hodnotu `trend_threshold_per_decade`, pokud byla použita jiná než výchozí,
- zda byly ve výstupu zahrnuty roční série,
- verzi API nebo commit repozitáře.

Doporučený minimální záznam metadat pro statický výpočet klimatické zóny:

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

Doporučený minimální záznam metadat pro trend klimatických zón:

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

Pro trend klimatických zón založený na přesahu mimo okraje škály se nastaví:

```json
{
  "trend_metric": "exceedance_rank_equivalent"
}
```

## 20. Shrnutí metodiky v jedné větě

Statická klimatická zóna se určí tak, že se pro zadané místo a období spočítá nebo interpoluje 11 víceletých klimatických charakteristik, ty se porovnají s intervaly 23 Quittových klimatických jednotek a jako výsledek se vybere jednotka s nejvyšší fuzzy mírou shody; trendové endpointy používají roční hodnoty všech let zvoleného období a pomocí Senovy směrnice počítají trendy buď pro jednotlivé indikátory, nebo pro metriky klimatických zón `expected_fuzzy_rank` a `exceedance_rank_equivalent`.
