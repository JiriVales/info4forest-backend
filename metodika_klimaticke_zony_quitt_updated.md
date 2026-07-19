# Info4forest WF5: klimatické oblasti inspirované Quittovou klasifikací

## Popis vizualizace a metodika výpočtu

**Verze dokumentu:** 3.0  
**Datum revize:** 2026-07-19  
**Určení dokumentu:** věcný popis workflow WF5 v aplikaci Info4forest, vysvětlení zobrazovaných výsledků a transparentní metodika jejich výpočtu.  
**Metodický základ:** Quittova klasifikace klimatu a její rozbor v publikaci Univerzity Palackého v Olomouci a Českého hydrometeorologického ústavu [1].  

> Tento dokument není návodem k ovládání jednotlivých tlačítek ani příručkou k API endpointům. Vysvětluje především, **co aplikace Info4forest ve workflow WF5 zobrazuje, co výsledky znamenají, z jakých dat vycházejí a jak jsou vypočteny**.

---

## Obsah

1. Co je workflow WF5
2. Co aplikace zobrazuje
3. Jak číst hlavní výsledky
4. Klimatické zdroje a sledované lokality
5. Vztah k Quittově klasifikaci
6. Jedenáct použitých klimatických charakteristik
7. Přehled klimatických oblastí a jednotek
8. Metodika výpočtu klimatické jednotky
9. Metodika trendů
10. Interpretace, nejistoty a omezení
11. API jako datové a výpočetní zázemí
12. Doporučené citování
13. Zdroje
14. Příloha A – limitní intervaly používané implementací

---

## 1. Co je workflow WF5

Workflow **WF5 – Forest-related Climate Area Map** v aplikaci Info4forest převádí klimatická data do srozumitelné mapy klimatických oblastí a souvisejících charakteristik. Uživatel zvolí klimatický zdroj, lokalitu a období. Aplikace následně ukáže, kterému typu klimatu se jednotlivá místa v daném období nejvíce podobají.

Výsledek není založen pouze na jedné veličině, například na průměrné teplotě. Pro každé místo se společně vyhodnocuje jedenáct teplotních a srážkových charakteristik. Patří mezi ně počet letních, mrazových a ledových dnů, teploty ve čtyřech reprezentativních měsících, počet srážkových dnů a sezónní srážkové úhrny.

Zjednodušeně lze výpočet chápat jako **porovnání klimatického profilu místa s 23 známými klimatickými jednotkami**. Aplikace vybere jednotku, jejímž intervalům se profil místa podobá nejvíce. Jednotky tvoří škálu od nejchladnějších typů `CH1` až po nejteplejší typ `T5`.

> **Stručné vysvětlení pro uživatele aplikace:** Klimatické oblasti inspirované Quittovou klasifikací ukazují, jakému typu klimatu se zvolené místo ve vybraném období nejvíce podobá. Výpočet porovnává teploty, srážky a další klimatické ukazatele s 23 jednotkami od chladných po teplé oblasti a zobrazí nejbližší shodu.

Výpočet používá 11 charakteristik dostupných v datovém systému Info4forest. Jde o algoritmickou a opakovatelnou realizaci inspirovanou Quittovým schématem; nejde o prosté překreslení historické Quittovy mapy.

---

## 2. Co aplikace zobrazuje

### 2.1 Výběr klimatického zdroje, lokality a období

Aplikace umožňuje zvolit:

- **klimatický zdroj**, například historickou reanalýzu ERA5-Land nebo budoucí klimatický model;
- **lokalitu**, pro kterou je zdroj dostupný;
- **počáteční a koncový rok** analyzovaného období.

Laický název zdroje je určen pro běžného uživatele, například „Historické klima / model ERA5-Land / 10 × 10 km“. Technický název dostupný v informační nápovědě upřesňuje konkrétní model, regionální model a scénář, například `EC-Earth / REMO2020 / SSP3-7.0` [T4].

### 2.2 Mapa klimatických oblastí

Základní mapa zobrazuje pro každé mapové místo nejlépe odpovídající klimatickou jednotku. Barevná škála postupuje od chladných jednotek přes mírně teplé k teplým. Mapa poskytuje prostorový přehled a umožňuje rozpoznat rozdíly mezi nížinami, středními polohami a horskými oblastmi.

Po kliknutí do mapy se zobrazí podrobnosti pro konkrétní bod:

- klimatická jednotka a její slovní popis;
- interní český kód, například `T2`, a anglický/atlasový kód, například `W2`;
- skóre shody;
- průměrné hodnoty jedenácti klimatických charakteristik;
- trend klimatické jednotky a jeho statistická významnost;
- nejhorší souvislá nepříznivá odchylka od trendové linie;
- trendy jednotlivých klimatických charakteristik.

### 2.3 Mapa trendu klimatických oblastí

Mapa **Trend klimatických oblastí / Climate zone trends** ukazuje odhadovaný posun po škále klimatických jednotek během zvoleného období. Vizualizovaná veličina `estimated_total_change` vyjadřuje odhad celkové změny:

- kladná hodnota znamená posun směrem k teplejším klimatickým jednotkám;
- záporná hodnota znamená posun směrem k chladnějším jednotkám;
- hodnota blízká nule znamená malý nebo žádný systematický posun.

Jde o spojitou odhadovanou změnu pořadí jednotek, nikoli o prosté odečtení kódu prvního a posledního roku. Výpočet používá roční hodnoty za celé období.

### 2.4 Jedenáct map klimatických charakteristik

Vedle klimatických oblastí lze samostatně zobrazit všech jedenáct charakteristik. Každá mapa zobrazuje průměrnou hodnotu charakteristiky za zvolené období. U vybraného bodu se zároveň zobrazuje její trend; neprůkazný trend je v aplikaci označen pomlčkou.

### 2.5 Trendové informace pro vybraný bod

Pro zvolený bod aplikace uvádí:

- **Celkovou změnu** – odhad změny za celé období;
- **Trend za dekádu** – odhadovanou změnu za deset let;
- **Významný trend: ano/ne** – výsledek Mannova–Kendallova testu;
- **Nejhorší odchylku od trendu** – nejzávažnější souvislé období nepříznivých odchylek;
- **Trvání** – délku této epizody v letech;
- **Průměrné zhoršení** – průměrnou velikost nepříznivé odchylky během epizody.

„Významný“ v tomto kontextu znamená **statisticky průkazný**, nikoli automaticky ekologicky nebo hospodářsky významný.

### 2.6 Ukázky rozhraní

![Mapa klimatických oblastí a analytický panel workflow WF5](info4forest_WF5_ukazka_mapy.png)

*Obrázek 1: Základní zobrazení mapy klimatických oblastí, volby zdroje a období a detailu klimatické jednotky s charakteristikami.*

![Workflow WF5 v kontextu aplikace Info4forest](info4forest_WF5_ukazka_aplikace.png)

*Obrázek 2: Workflow WF5 v prostředí aplikace Info4forest včetně projektové navigace a panelu výsledků.*

---

## 3. Jak číst hlavní výsledky

### 3.1 Klimatická oblast a klimatická jednotka

Quittovo schéma rozlišuje tři široké oblasti:

- **CH – chladná oblast**: `CH1` až `CH7`;
- **MT – mírně teplá oblast**: `MT1` až `MT11`;
- **T – teplá oblast**: `T1` až `T5`.

Aplikace vrací konkrétní **klimatickou jednotku**, nikoliv pouze jednu ze tří širokých oblastí. Jednotka je nejbližší klimatický typ podle společného vyhodnocení všech charakteristik.

Pro anglickou verzi se používají atlasové kódy `C1–C7`, `MW1–MW11` a `W1–W5`. Interní výpočetní kódy zůstávají `CH`, `MT` a `T` [T3].

### 3.2 Shoda

Hodnota **shoda** (`best_score`) nabývá hodnot od 0 do 1. Vyjadřuje, jak dobře se vypočtených jedenáct hodnot vejde do intervalů nejlepší klimatické jednotky.

- hodnota blízká `1` znamená velmi dobrou podobnost;
- střední hodnota znamená částečnou podobnost nebo polohu blízko hranic více jednotek;
- nízká hodnota znamená, že ani nejlepší z 23 jednotek neodpovídá místu příliš dobře.

Shoda **není pravděpodobnost**, interval spolehlivosti ani podíl území. Nelze například říci, že skóre `0,69` znamená 69% pravděpodobnost příslušnosti k jednotce.

### 3.3 Průměr

U klimatické charakteristiky označuje „Průměr“ aritmetický průměr ročních hodnot za celé vybrané období. Například průměrný počet letních dnů za období 1990–2009 je průměrem dvaceti ročních hodnot.

### 3.4 Trend a statistická významnost

Trend je odhadnut Senovou směrnicí z ročních hodnot. Statistická významnost je vyhodnocena Mannovým–Kendallovým testem. Pokud test neprokáže trend na nastavené hladině významnosti, aplikace u trendu jednotlivé charakteristiky zobrazí pomlčku. To neznamená, že hodnota byla po celé období neměnná; znamená to, že dostupná řada neposkytuje dostatečně silný důkaz o monotónním trendu.

### 3.5 Nejhorší nepříznivá epizoda

Nejhorší epizoda je nejzávažnější souvislý úsek let, ve kterém se hodnoty nacházely na nepříznivé straně robustní trendové linie. „Nepříznivý“ směr závisí na charakteristice. Například růst letních dnů je ve výchozím nastavení považován za nepříznivý nárůst, zatímco pokles srážkového úhrnu ve vegetačním období za nepříznivý pokles [T2].

---

## 4. Klimatické zdroje a sledované lokality

### 4.1 Lokality

Katalog aplikace obsahuje tyto lokality [T4]:

| Kód | Český název | Anglický název | Bounding box `[minLon, minLat, maxLon, maxLat]` |
|---|---|---|---|
| `cz` | Česko | Czechia | `[12.07, 48.48, 18.97, 51.08]` |
| `sk` | Slovensko | Slovakia | `[16.8174, 47.6296, 22.7629, 49.705]` |
| `maestrazgo_mount` | pohoří Maestrazgo | Maestrazgo Mountains | `[-1.09395, 39.8289, 0.297912, 40.8966]` |
| `southern_fi` | Jižní Finsko | Southern Finland | `[20.3719, 58.9769, 32.7637, 63.1035]` |

Bounding box vymezuje datové pokrytí zdroje. Neznamená automaticky administrativní hranici státu nebo pilotního území.

### 4.2 Klimatické zdroje

| Název zobrazovaný uživateli | Odborný technický název | Dostupné lokality |
|---|---|---|
| Historické klima / model ERA5-Land / 10 × 10 km | `ERA5-Land` | `cz` |
| Budoucí klima, pesimistický scénář / model MPI / 12 × 12 km | `MPI / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |
| Budoucí klima, pesimistický scénář / model EC-Earth / 12 × 12 km | `EC-Earth / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |
| Budoucí klima, pesimistický scénář / model MIROC / 12 × 12 km | `MIROC / REMO2020 / SSP3-7.0` | `cz`, `sk`, `maestrazgo_mount`, `southern_fi` |

Historická reanalýza a klimatický model nejsou totéž:

- **reanalýza** kombinuje meteorologická pozorování s numerickým modelem a poskytuje konzistentní rekonstrukci minulého klimatu;
- **klimatická projekce** simuluje možný budoucí vývoj podle zvoleného emisního/scénářového předpokladu a modelového řetězce.

Výsledky různých modelů se nemají chápat jako přesná předpověď konkrétního roku. Slouží k hodnocení možného dlouhodobého vývoje a modelové nejistoty.

---

## 5. Vztah k Quittově klasifikaci

### 5.1 Původní klasifikační schéma

Quittova klasifikace je příkladem **komplexní klimatologie**: území se nezařazuje podle jedné proměnné, ale podle kombinace tříd více klimatologických charakteristik. Publikace UPOL/ČHMÚ uvádí 23 klimatických jednotek ve třech oblastech, definovaných kombinacemi 14 charakteristik [1, s. 3–6; PDF s. 4–7].

Původní hranice byly vytvořeny pro klimatické poměry Československa a vycházely z historických mapových podkladů. Autoři UPOL/ČHMÚ upozorňují, že intervaly jednotlivých jednotek se překrývají a praktická realizace klasifikace obsahuje neurčitost i určitý podíl expertního rozhodování [1, s. 4; PDF s. 5].

### 5.2 Nejistota při přiřazení

Podrobná analýza UPOL/ČHMÚ ukázala, že žádný hodnocený čtverec nesplnil všech 14 původních kritérií a nejčastěji bylo splněno jen 6 až 9 kritérií. Shodné maximum navíc může nastat u více jednotek [1, s. 13–14; PDF s. 14–15]. Proto je vhodné výsledek chápat jako **nejbližší klimatický typ**, nikoliv jako absolutní a bezchybnou hranici.

UPOL/ČHMÚ popisuje také digitální metodu maxima splněných kritérií, v níž se při shodě upřednostňuje teplejší jednotka. Současně upozorňuje, že různé metody mohou při stejných vstupních datech vést k odlišné realizaci mapy [1, s. 14; PDF s. 15].

### 5.3 Co přebírá Info4forest a co je vlastní výpočetní rozšíření

Info4forest přebírá:

- systém 23 jednotek a tří širokých oblastí;
- kódy a slovní charakteristiky jednotek;
- intervalové meze použitých klimatických charakteristik;
- orientaci škály od chladnějších k teplejším jednotkám.

Info4forest doplňuje vlastní algoritmické postupy:

- používá jedenáct dostupných charakteristik;
- hodnotí podobnost fuzzy skórem místo čistě binárního splňuje/nesplňuje;
- interpoluje bodové hodnoty metodou IDW;
- umožňuje libovolné dostupné víceleté období;
- počítá robustní trendy, jejich statistickou významnost a nepříznivé epizody;
- pracuje s více klimatickými zdroji a lokalitami.

Proto je správné označení **„klimatické oblasti inspirované Quittovou klasifikací“**. Výstup není totožný s historickou Quittovou mapou ani s mapou UPOL/ČHMÚ pro období 1961–2000.

---

## 6. Jedenáct použitých klimatických charakteristik

| Index v datech | Kód | Název | Jednotka | Význam |
|---:|---|---|---|---|
| 0 | `SUM25` | Roční počet letních dnů | dny | Počet dnů, kdy maximální denní teplota dosáhne alespoň 25 °C. |
| 1 | `VEG10` | Roční počet dnů s průměrnou teplotou ≥ 10 °C | dny | Orientačně vyjadřuje délku teplejší části roku vhodné pro vegetaci. |
| 2 | `FROST0` | Roční počet mrazových dnů | dny | Počet dnů, kdy minimální denní teplota klesne pod 0 °C. |
| 3 | `ICE0` | Roční počet ledových dnů | dny | Počet dnů, kdy maximální denní teplota zůstane pod 0 °C. |
| 4 | `T_JAN` | Průměrná teplota v lednu | °C | Charakteristika teplotních poměrů uprostřed zimy. |
| 5 | `T_APR` | Průměrná teplota v dubnu | °C | Charakteristika jarních teplotních poměrů. |
| 6 | `T_JUL` | Průměrná teplota v červenci | °C | Charakteristika teplotních poměrů uprostřed léta. |
| 7 | `T_OCT` | Průměrná teplota v říjnu | °C | Charakteristika podzimních teplotních poměrů. |
| 8 | `RAIN1` | Roční počet srážkových dnů ≥ 1 mm | dny | Počet dnů se srážkovým úhrnem alespoň 1 mm. |
| 9 | `P_APR_SEP` | Srážkový úhrn duben–září | mm | Úhrn srážek za vegetační část roku od dubna do září. |
| 10 | `P_OCT_MAR` | Srážkový úhrn říjen–březen | mm | Úhrn srážek za chladnou část roku od října do března. |

Původní Quittovo schéma pracuje se 14 charakteristikami [1, s. 4–7; PDF s. 5–8]. Info4forest používá jedenáct charakteristik dostupných ve sjednocené datové struktuře WF5. Nejsou zahrnuty charakteristiky spojené se sněhovou pokrývkou, oblačností a jasnými/zamračenými dny. Tato redukce je jedním z důvodů, proč výstup označujeme jako klasifikací inspirovaný, nikoli jako její úplnou reprodukci.

---

## 7. Přehled klimatických oblastí a jednotek

### 7.1 Orientační škála

```text
NEJCHLADNĚJŠÍ                                                        NEJTEPLEJŠÍ
CH1 → CH2 → CH3 → CH4 → CH5 → CH6 → CH7 → MT1 → … → MT11 → T1 → T2 → T3 → T4 → T5
│----------- chladná oblast -----------││------ mírně teplá oblast ------││-- teplá --│
```

Pořadí je orientační klimatická škála používaná výpočtem. Rozdíl jednoho pořadového stupně nelze automaticky interpretovat jako konstantní fyzikální změnu teploty nebo srážek; sousední jednotky jsou definovány kombinací více proměnných.

### 7.2 Přehled všech 23 jednotek

Slovní charakteristiky v následující tabulce vycházejí z tabulky 1 publikace UPOL/ČHMÚ [1, s. 4–5; PDF s. 5–6] a jsou normalizovány podle metadat aplikace [T3].

| Interní kód | Anglický/atlasový kód | Český název | Slovní charakteristika |
|---|---|---|---|
| `CH1` | `C1` | Chladná 1 | Léto velmi krátké, chladné, velmi vlhké, přechodné období velmi dlouhé s velmi chladným jarem a chladným podzimem, zima velmi dlouhá, velmi chladná, velmi vlhká s velmi dlouhým trváním sněhové pokrývky |
| `CH2` | `C2` | Chladná 2 | Léto velmi krátké, chladné, velmi vlhké (ale ve srovnání s rajonem C1 s menším úhrnem srážek), přechodné období velmi dlouhé s velmi chladným jarem a chladným podzimem, zima velmi dlouhá, velmi chladná, velmi vlhká (ale ve srovnání s rajonem C1 s menším úhrnem srážek), s velmi dlouhým trváním sněhové pokrývky |
| `CH3` | `C3` | Chladná 3 | Léto velmi krátké, chladné a vlhké, přechodné období velmi dlouhé s velmi chladným až chladným jarem a chladným podzimem, zima velmi dlouhá, velmi chladná, vlhká s velmi dlouhým trváním sněhové pokrývky |
| `CH4` | `C4` | Chladná 4 | Léto velmi krátké, chladné a vlhké, přechodné období velmi dlouhé s chladným jarem a mírně chladným podzimem, zima velmi dlouhá, velmi chladná, vlhká, s velmi dlouhým trváním sněhové pokrývky |
| `CH5` | `C5` | Chladná 5 | Léto velmi krátké až krátké, mírně chladné a vlhké, přechodné období dlouhé s chladným jarem a mírně chladným podzimem, zima velmi dlouhá a chladná, mírně vlhká s dlouhým trváním sněhové pokrývky |
| `CH6` | `C6` | Chladná 6 | Léto velmi krátké až krátké, mírně chladné, vlhké až velmi vlhké, přechodné období dlouhé s chladným jarem a mírně chladným podzimem, zima velmi dlouhá, mírně chladná, vlhká, s dlouhým trváním sněhové pokrývky |
| `CH7` | `C7` | Chladná 7 | Léto velmi krátké až krátké, mírně chladné a vlhké, přechodné období dlouhé s mírně chladným jarem a mírným podzimem, zima dlouhá, mírná, mírně vlhká s dlouhým trváním sněhové pokrývky |
| `MT1` | `MW1` | Mírně teplá 1 | Léto krátké, mírně chladné a vlhké, přechodné období velmi dlouhé s mírně chladným jarem a mírným podzimem, zima normálně dlouhá, chladná, suchá až mírně suchá s dlouhým trváním sněhové pokrývky |
| `MT2` | `MW2` | Mírně teplá 2 | Léto krátké, mírné až mírně chladné, mírně vlhké, přechodné období krátké s mírným jarem a mírným podzimem, zima normálně dlouhá s mírnými teplotami, suchá s normálně dlouhým trváním sněhové pokrývky |
| `MT3` | `MW3` | Mírně teplá 3 | Léto krátké, mírné až mírně chladné, suché až mírně suché, přechodné období normální až dlouhé s mírným jarem a mírným podzimem, zima normálně dlouhá, mírná až mírně chladná, suchá až mírně suchá s normálním až krátkým trváním sněhové pokrývky |
| `MT4` | `MW4` | Mírně teplá 4 | Léto krátké, mírné, suché až mírně suché, přechodné období krátké s mírným jarem a mírným podzimem, zima normálně dlouhá, mírně teplá a suchá s krátkým trváním sněhové pokrývky |
| `MT5` | `MW5` | Mírně teplá 5 | Léto normálně dlouhé až krátké, mírné až mírně chladné, suché až mírně suché, přechodné období normální až dlouhé s mírným jarem a mírným podzimem, zima normálně dlouhá, mírně chladná, suchá až mírně suchá, s normálním trváním sněhové pokrývky |
| `MT6` | `MW6` | Mírně teplá 6 | Léto normálně dlouhé až dlouhé, mírné, mírně vlhké, přechodné období normální až dlouhé s mírným až mírně teplým jarem a mírným podzimem, zima normálně dlouhá, chladná, suchá až mírně suchá s normálním trváním sněhové pokrývky |
| `MT7` | `MW7` | Mírně teplá 7 | Léto normálně dlouhé, mírné, mírně suché, přechodné období krátké s mírným jarem a mírně teplým podzimem, zima normálně dlouhá, mírně teplá, suchá až mírně suchá s krátkým trváním sněhové pokrývky |
| `MT8` | `MW8` | Mírně teplá 8 | Léto dlouhé, teplé, mírně vlhké, přechodné období normálně dlouhé s mírně teplým jarem a mírně teplým podzimem, zima normálně dlouhá, mírná až mírně chladné, suchá, s krátkým trváním sněhové pokrývky |
| `MT9` | `MW9` | Mírně teplá 9 | Léto dlouhé, teplé, suché až mírně suché, přechodné období krátké s mírným až mírně teplým jarem a mírně teplým podzimem, zima krátká, mírná, suchá, s krátkým trváním sněhové pokrývky |
| `MT10` | `MW10` | Mírně teplá 10 | Léto dlouhé, teplé, mírně suché, přechodné období krátké s mírně teplým jarem a mírně teplým podzimem, zima krátká, mírně teplá a velmi suchá, s krátkým trváním sněhové pokrývky |
| `MT11` | `MW11` | Mírně teplá 11 | Léto dlouhé, teplé a suché, přechodné období krátké s mírně teplým jarem a mírně teplým podzimem, zima krátká, teplá a velmi suchá, s krátkým trváním sněhové pokrývky |
| `T1` | `W1` | Teplá 1 | Léto dlouhé, teplé a suché, přechodné období krátké s mírně teplým až teplým jarem a mírně teplým až teplým podzimem, zima krátká, mírná až mírně chladná, suchá až velmi suchá, s krátkým trváním sněhové pokrývky |
| `T2` | `W2` | Teplá 2 | Léto dlouhé, teplé a suché, přechodné období velmi krátké s teplým až mírně teplým jarem a mírně teplým až teplým podzimem, zima krátká, mírně teplá, suchá až velmi suchá, s velmi krátkým trváním sněhové pokrývky |
| `T3` | `W3` | Teplá 3 | Léto velmi dlouhé, velmi teplé a suché, přechodné období krátké s teplým jarem a teplým podzimem, zima krátká, mírná, suchá až velmi suchá, s krátkým trváním sněhové pokrývky |
| `T4` | `W4` | Teplá 4 | Léto velmi dlouhé, velmi teplé a velmi suché, přechodné období velmi krátké s teplým jarem a teplým podzimem, zima krátká, mírně teplá, suchá až velmi suchá, s velmi krátkým trváním sněhové pokrývky |
| `T5` | `W5` | Teplá 5 | Léto velmi dlouhé, velmi teplé a velmi suché, přechodné období velmi krátké s teplým jarem a teplým podzimem, zima velmi krátká, teplá, suchá až velmi suchá, s velmi krátkým trváním sněhové pokrývky |

---

## 8. Metodika výpočtu klimatické jednotky

### 8.1 Vstupní datová struktura

Roční klimatické charakteristiky jsou uloženy v rastrové kolekci v pořadí:

```text
[year, indicator, y, x]
```

Každá prostorová buňka tedy obsahuje pro každý rok jedenáct hodnot. Aplikace nejprve ověří, že zvolené období a souřadnice leží v rozsahu vybraného datového zdroje [T1].

### 8.2 Víceletý průměr

Pro statickou mapu se pro každý indikátor `i` a místo `s` vypočítá průměr přes všechny roky zvoleného období:

```text
mean_i(s) = (1 / n) × Σ I_i(year, s)
```

Počáteční i koncový rok jsou zahrnuty. Výpočet tedy neporovnává pouze první a poslední rok.

### 8.3 Prostorová interpolace

Mapový výstup na nativní datové mřížce používá přímo hodnoty zdrojových buněk. Pro přesně zvolený bod, který zpravidla neleží ve středu buňky, se používá **IDW – inverse distance weighting**. Bližší datové body mají vyšší váhu než vzdálenější:

```text
w_j = 1 / d_j^p
interpolovaná_hodnota = Σ(w_j × hodnota_j) / Σ(w_j)
```

Výchozí exponent je `p = 2`. Použije se nejvýše devět nejbližších platných bodů z lokálního okolí. Vzdálenost se počítá po zemském povrchu. Pokud požadovaná souřadnice přesně odpovídá zdrojovému bodu, vrátí se jeho hodnota bez průměrování [T1, T5].

IDW nezvyšuje skutečné prostorové rozlišení zdrojových dat. Vytváří plynulý odhad mezi dostupnými body.

### 8.4 Dílčí podobnost k intervalu

Každá klimatická jednotka obsahuje pro každý indikátor dolní a horní mez. Pro hodnotu uvnitř intervalu je podobnost rovna `1`. Mimo interval klesá lineárně až k nule:

```text
μ = 1                                         pro lower ≤ value ≤ upper
μ = max(0, 1 − (lower − value) / tolerance)   pro value < lower
μ = max(0, 1 − (value − upper) / tolerance)   pro value > upper
```

Použité toleranční šířky jsou [T5]:

| Skupina | Charakteristiky | Tolerance |
|---|---|---:|
| Počty dnů | `SUM25`, `VEG10`, `FROST0`, `ICE0`, `RAIN1` | 10 dnů |
| Teploty | `T_JAN`, `T_APR`, `T_JUL`, `T_OCT` | 1 °C |
| Srážkové úhrny | `P_APR_SEP`, `P_OCT_MAR` | 50 mm |

Tolerance nejsou převzaty jako samostatný parametr z původní Quittovy publikace; jsou součástí výpočetní implementace Info4forest.

### 8.5 Celkové skóre a výběr jednotky

Celkové skóre jednotky je aritmetický průměr dílčích podobností všech platných indikátorů:

```text
score(unit) = mean(μ_1, μ_2, …, μ_11)
best_unit   = jednotka s nejvyšším score
```

Při přesně shodném výsledku více jednotek se upřednostní jednotka pozdější v pořadí od `CH1` k `T5`, tedy teplejší jednotka [T5]. Tento princip je konzistentní s jednou z digitálních realizací popsaných UPOL/ČHMÚ, která při shodném maximu rovněž preferovala teplejší jednotku [1, s. 14; PDF s. 15].

---

## 9. Metodika trendů

### 9.1 Proč se nepoužívá jen první a poslední rok

Rozdíl mezi dvěma roky může být silně ovlivněn náhodně teplým, chladným, suchým nebo vlhkým rokem. Trendy proto používají všechny platné roční hodnoty ve zvoleném období.

### 9.2 Senova směrnice

Pro každou dvojici roků se vypočítá změna hodnoty dělená časovým rozdílem. Senova směrnice je medián všech těchto párových směrnic:

```text
slope_per_year = median((value_j − value_i) / (year_j − year_i))
slope_per_decade = 10 × slope_per_year
estimated_total_change = slope_per_year × (max_year − min_year)
```

Medián je méně citlivý na jednotlivé extrémní roky než běžná lineární regrese [T2].

### 9.3 Mannův–Kendallův test

Mannův–Kendallův test zjišťuje, zda hodnoty vykazují statisticky průkaznou monotónní tendenci. Implementace zohledňuje shodné hodnoty a používá normální aproximaci. Aplikace pracuje s výchozí hladinou `α = 0,05` [T2].

- `significant = true`: trend je na dané hladině statisticky průkazný;
- `significant = false`: průkaznost nebyla potvrzena.

Statistická průkaznost nevypovídá sama o velikosti nebo praktickém dopadu změny. Proto je nutné společně hodnotit i `estimated_total_change`, `slope_per_decade`, délku období a charakter datového zdroje.

### 9.4 Trend klimatické jednotky

Každá klimatická jednotka má pořadí od nejchladnější k nejteplejší. Pro každý rok se z fuzzy skóre odvodí spojitá očekávaná pozice na této škále (`expected_fuzzy_rank`). Následně se na roční řadu této pozice aplikuje Senova směrnice [T2].

Kladný trend znamená posun k teplejším jednotkám; záporný trend posun k chladnějším. Hodnota může být desetinná, protože jde o trend spojité očekávané pozice, nikoliv o počet přeskočených barevných kategorií.

### 9.5 Trend jednotlivé charakteristiky

U map a detailu jednotlivých charakteristik se trend počítá ve fyzikální jednotce dané proměnné:

- dny za dekádu u počtů dnů;
- °C za dekádu u teplot;
- mm za dekádu u srážkových úhrnů.

Celková změna je odhad trendu za celé období, nikoli rozdíl první a poslední naměřené/modelované hodnoty.

### 9.6 Nejhorší nepříznivá epizoda

Nejprve se vytvoří robustní trendová linie se Senovou směrnicí a mediánovým průsečíkem. Pro každý rok se určí odchylka od této linie v předem stanoveném nepříznivém směru. Algoritmus vyhledá souvislé úseky kladné nepříznivé odchylky a vybere úsek s nejvyšší celkovou závažností [T2].

- `duration` – počet po sobě jdoucích let;
- `severity` – součet nepříznivých odchylek;
- `mean_severity` – průměrná odchylka v epizodě;
- `peak_anomaly` – největší jednotlivá nepříznivá odchylka.

Tato diagnostika popisuje epizody **vzhledem k dlouhodobému trendu**, nikoli automaticky absolutně nejhorší klimatické období z hlediska lesního hospodářství.

---

## 10. Interpretace, nejistoty a omezení

### 10.1 Klimatická jednotka je nejbližší podobnost

Hranice klimatických jednotek se v původním schématu překrývají a jejich číselné meze jsou podle UPOL/ČHMÚ ve vztahu k realitě orientační [1, s. 18; PDF s. 19]. Výsledek se proto nemá interpretovat jako přesná přírodní hranice.

### 10.2 Klasifikace byla vytvořena pro jiné historické poměry

Původní meze vycházejí z klimatických hodnot historického Československa. Při aplikaci na jiné regiony, budoucí scénáře nebo hodnoty mimo historický rozsah mohou být některé profily vzdálené všem jednotkám. To se projeví nižší shodou nebo hodnotami na okrajích škály.

### 10.3 Používá se 11 charakteristik

Info4forest nepoužívá všechny původní charakteristiky. Výsledek proto nelze vydávat za úplnou reprodukci klasifikace z roku 1971 nebo mapy UPOL/ČHMÚ.

### 10.4 Rozlišení mapy je omezeno zdrojovými daty

Plynulé vykreslení a IDW mohou působit detailněji, než je skutečné rozlišení modelu. Například zdroj s rozlišením přibližně 10 × 10 km nemůže spolehlivě popsat mikroklima svahu, údolí nebo porostu v měřítku jednotlivých hektarů.

### 10.5 Modelové zdroje obsahují nejistotu

Budoucí klimatická data závisejí na globálním modelu, regionálním modelu, scénáři a přirozené variabilitě klimatu. Pro rozhodování je vhodné porovnat více zdrojů, období a případně další odborné podklady.

### 10.6 Trend klimatické jednotky je odvozená veličina

Jednotka kombinuje více charakteristik. Stejný posun v její trendové škále může být na různých místech způsoben jinou kombinací změn teploty a srážek. Pro věcnou interpretaci je proto vhodné současně prohlédnout trendy jednotlivých charakteristik.

### 10.7 Vhodné a nevhodné použití

Výstupy jsou vhodné zejména pro:

- orientační popis klimatického typu území;
- porovnání období, lokalit nebo klimatických zdrojů;
- screening dlouhodobých změn relevantních pro lesní plánování;
- identifikaci území vhodných pro podrobnější analýzu.

Výstupy samy o sobě nejsou náhradou za:

- lokální měření a mikroklimatické posouzení;
- dendrologické, půdní, hydrologické nebo stanovištní hodnocení;
- provozní předpověď počasí;
- jednoznačný podklad pro zásadní rozhodnutí bez další odborné interpretace.

---

## 11. API jako datové a výpočetní zázemí

Aplikace Info4forest získává katalog zdrojů, mapová data, bodové charakteristiky a trendové diagnostiky prostřednictvím FWF5 Climate API. Pro běžného uživatele není nutné API přímo používat, ale jeho zveřejnění podporuje transparentnost, reprodukovatelnost a integraci výsledků do dalších systémů.

API nabízí i podrobnější funkce, než které musí být vždy současně viditelné v základním rozhraní, například:

- úplný katalog klimatických zdrojů a lokalit;
- metadata klimatických charakteristik a jednotek;
- roční řady všech jedenácti charakteristik pro konkrétní bod;
- samostatné mapové a bodové trendové výstupy;
- technické informace o časovém a prostorovém rozsahu datových kolekcí;
- bohatší diagnostické údaje o trendu, kvalitě řady a posunech různě dlouhých období.

Aktuální technická dokumentace API je dostupná zde:

**<https://app.swaggerhub.com/apis/JIRKAVALES_1/Info4forest/1.0.0>**

Příklady endpointů, které jsou podkladem aplikace:

- `/climaticsourcecatalog` – klimatické zdroje a jejich laické i technické názvy;
- `/climaticsourcecatalog/locations` – lokality a datové rozsahy;
- `/climaticzones/grid` a `/climaticzones/point` – klimatické jednotky;
- `/climaticzones/trend/grid` a `/climaticzones/trend/point` – trend klimatických jednotek;
- `/climatecharacteristics/{characteristic}/grid` – mapy jednotlivých charakteristik;
- `/climatecharacteristics/trend/{characteristic}/point` – trend charakteristiky v bodě.

Tyto názvy jsou uvedeny pouze pro dohledatelnost technického zázemí; hlavní smysl tohoto dokumentu je věcná interpretace vizualizace a metodiky.

---

## 12. Doporučené citování

Při citování mapy nebo výsledku je vhodné uvést:

1. aplikaci a workflow: **Info4forest, WF5 – Forest-related Climate Area Map**;
2. klimatický zdroj/model a scénář;
3. lokalitu a zvolené období;
4. datum vytvoření nebo stažení výsledku;
5. odkaz na tuto metodiku;
6. odborný metodický zdroj [1] a podle potřeby původní Quittovu práci [2].

Příklad:

> Info4forest (2026): WF5 – Forest-related Climate Area Map, klimatický zdroj ERA5-Land, lokalita Česko, období 1991–2020. Klimatické jednotky inspirované Quittovou klasifikací; metodika podle Květoně a Voženílka (2011) a implementace Info4forest.

---

## 13. Zdroje

### Odborné a metodické zdroje

**[1]** KVĚTOŇ, Vít; VOŽENÍLEK, Vít. *Klimatické oblasti Česka: klasifikace podle Quitta za období 1961–2000 / Climatic Regions of Czechia: Quitt's Classification during Years 1961–2000*. Olomouc: Univerzita Palackého v Olomouci v koedici s Českým hydrometeorologickým ústavem, 2011. M.A.P.S., Num. 3. ISBN 978-80-244-2813-0; ISBN 978-80-86690-89-6.

**[2]** QUITT, Evžen. *Klimatické oblasti Československa*. Studia Geographica, sv. 16. Brno: Geografický ústav ČSAV, 1971, 73 s. Bibliografický údaj podle [1, s. 19; PDF s. 20].

### Technické zdroje implementace

**[T1]** `FWF5_API(10).py` – API vrstva, validace vstupů, katalog, výpočetní orchestrace a formát odpovědí.  
**[T2]** `FWF5_trends(11).py` – Senova směrnice, Mannův–Kendallův test, trend klimatických jednotek a nepříznivé epizody.  
**[T3]** `FWF5_climate_metadata(3).json` – české a anglické názvy, kódy a slovní popisy klimatických jednotek a charakteristik.  
**[T4]** `climate_source_catalog(6).json` – katalog klimatických zdrojů, lokalit, laických a technických názvů a názvů Rasdaman kolekcí.  
**[T5]** `FWF5_quitt_limits_11_rasdaman_index(3).json`, `FWF5_quitt_fuzzy_match_point(3).py` a `FWF5_quitt_fuzzy_match_area_custom_grid(3).py` – limitní intervaly, fuzzy skórování, IDW a pořadí klimatických jednotek.  
**[T6]** OpenAPI dokumentace FWF5 Climate API: <https://app.swaggerhub.com/apis/JIRKAVALES_1/Info4forest/1.0.0>.

---

## 14. Příloha A – limitní intervaly používané implementací

Následující tabulka uvádí přesné intervaly jedenácti charakteristik používané aktuální implementací [T5]. Jednotky: počty dnů v dnech, teploty v °C a srážkové úhrny v mm.

| Jednotka | SUM25 | VEG10 | FROST0 | ICE0 | T_JAN | T_APR | T_JUL | T_OCT | RAIN1 | P_APR_SEP | P_OCT_MAR |
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

### Poznámka k použití tabulky

Intervaly nemají být používány izolovaně jako jednoduchý rozhodovací strom. Implementace hodnotí všech jedenáct charakteristik společně a mimo interval používá plynule klesající fuzzy podobnost. UPOL/ČHMÚ současně upozorňuje, že původní číselné meze jsou orientační a různé realizace klasifikace mohou vést k odlišným mapám [1, s. 18; PDF s. 19].
