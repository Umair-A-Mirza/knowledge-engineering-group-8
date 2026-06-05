## Research Question #1 - 

### Are there statistics trending similarly to livability?

For the research question above, it is imperative for us to identify meaningful features which are trending with (or even against) livability scores. Some notable features to consider are:
- **Population** - This is the primary driver the client is concerned about, how population pressure impacts livability, or associates with it.
- **Safety Rating** - This is an important consideration to include the perspectives of residents in the analysis.
- **Cohesion Rating** - This is an important consideration to include the perspectives of residents in the analysis.
- **WOZ Property Values** - The valuation of properties in neighborhoods reflects the desirability of living in a particular neighborhood.
- **Total Dwellings** - The housing supply is a direct response to population pressure.
- **Percentage Unsafe** - The inverse relation to safety rating, could be useful in the presence of missingness.
- **Facilities Deviation** - A direct Leefbaarometer sub-dimension.
- **Housing Stock Deviation** - A direct Leefbaarometer sub-dimension.

#### Query 1 - Overall Eindhoven Trend

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?year
       (AVG(?livabilityScore) as ?avgLivability)
       (AVG(?livabilityDeviation) as ?avgLivabilityDeviation)
       (AVG(?population) as ?avgPopulation)
       (AVG(?safetyRating) as ?avgSafety)
       (AVG(?cohesionScore) as ?avgCohesion)
       (AVG(?avgWozValue) as ?avgWoz)
       (AVG(?totalDwellings) as ?avgTotalDwellings)
       (AVG(?pctUnsafe) as ?avgPctUnsafe)
       (AVG(?physicalDev) as ?avgPhysicalDeviation)
       (AVG(?nuisanceDev) as ?avgNuisanceDeviation)
       (AVG(?socialDev) as ?avgSocialDeviation)
       (AVG(?facilitiesDev) as ?avgFacilitiesDeviation)
       (AVG(?housingDev) as ?avgHousingDeviation)
       (SAMPLE(?dimFlag) as ?dimensionScoresAvailable)
       (SAMPLE(?surveyFlag) as ?surveyAvailable)
       (SAMPLE(?wozFlag) as ?wozAvailable)
WHERE {
    ?obs a ein:Observation ;
         ein:inYear ?year ;
         ein:population ?population .

    OPTIONAL { ?obs ein:livabilityScore ?livabilityScore . }
    OPTIONAL { ?obs ein:livabilityDeviation ?livabilityDeviation . }
    OPTIONAL { ?obs ein:totalDwellings ?totalDwellings . }
    OPTIONAL { ?obs ein:dimensionScoresAvailable ?dimFlag . }
    OPTIONAL { ?obs ein:surveyAvailable ?surveyFlag . }
    OPTIONAL { ?obs ein:wozAvailable ?wozFlag . }

    OPTIONAL {
        ?obs ein:dimensionScoresAvailable true ;
             ein:physicalEnvironmentDeviation ?physicalDev ;
             ein:nuisanceInsecurityDeviation ?nuisanceDev ;
             ein:socialCohesionDeviation ?socialDev ;
             ein:facilitiesDeviation ?facilitiesDev ;
             ein:housingStockDeviation ?housingDev .
    }

    OPTIONAL {
        ?obs ein:surveyAvailable true ;
             ein:safetyRating ?safetyRating ;
             ein:pctFeelsUnsafe ?pctUnsafe .
    }

    OPTIONAL {
        ?obs ein:surveyAvailable true ;
             ein:cohesionScaleScore ?cohesionScore .
    }

    OPTIONAL {
        ?obs ein:wozAvailable true ;
             ein:avgWozValue ?avgWozValue .
    }
}
GROUP BY ?year
ORDER BY ?year
```

The goal of this query is to provide an overview of core components and how they change over time at an Eindhoven-level.

**Analysis** - 
- We can plot all features normalized to the same scale (0–1) on a single line chart with year on the x-axis to visualize which features move together with livability.
- Identify which years show the largest drops in livability and check whether population, dwellings, or safety moved in the same period.

#### Query 2 - Weighted Average Scores Per Year

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?year
       (SUM(?livabilityScore * ?population) / SUM(?population) as ?weightedAvgLivability)
       (SUM(?livabilityDeviation * ?population) / SUM(?population) as ?weightedAvgDeviation)
       (SUM(?facilitiesDev * ?population) / SUM(?population) as ?weightedFacilitiesDeviation)
       (SUM(?housingDev * ?population) / SUM(?population) as ?weightedHousingDeviation)
       (SUM(?safetyRating * ?population) / SUM(?population) as ?weightedSafetyRating)
       (SUM(?pctUnsafe * ?population) / SUM(?population) as ?weightedPctUnsafe)
       (SUM(?population) as ?totalPopulation)
WHERE {
    ?obs a ein:Observation ;
         ein:inNeighborhood ?nb ;
         ein:inYear ?year ;
         ein:population ?population ;
         ein:livabilityScore ?livabilityScore .

    OPTIONAL { ?obs ein:livabilityDeviation ?livabilityDeviation . }
    OPTIONAL {
        ?obs ein:dimensionScoresAvailable true ;
             ein:facilitiesDeviation ?facilitiesDev ;
             ein:housingStockDeviation ?housingDev .
    }
    OPTIONAL {
        ?obs ein:surveyAvailable true ;
             ein:safetyRating ?safetyRating ;
             ein:pctFeelsUnsafe ?pctUnsafe .
    }
}
GROUP BY ?year
ORDER BY ?year
```

The goal of this query is to provide a comparison of the average annual livability score from Query 1 to a weighted average (weighted by population of neighborhood) annual livability score. 

**Analysis** - 
- We can make a simple plot of this score over time and also with the total population too, in order to compare with the average livability score from Query 1.
- Determine whether neighborhoods with larger populations are generally more livable.

#### Query 3 - Year-over-Year Differences

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?neighborhood 
       ?year1 ?year2
       (?score2 - ?score1 as ?livabilityDelta)
       (?pop2 - ?pop1 as ?populationDelta)
       (?dwellings2 - ?dwellings1 as ?dwellingsDelta)
       (?woz2 - ?woz1 as ?wozDelta)
       (?safety2 - ?safety1 as ?safetyDelta)
       (?facDev2 - ?facDev1 as ?facilitiesDeviationDelta)
       (?housDev2 - ?housDev1 as ?housingDeviationDelta)
WHERE {
    ?obs1 a ein:Observation ;
          ein:inNeighborhood ?nb ;
          ein:inYear ?year1 ;
          ein:livabilityScore ?score1 ;
          ein:population ?pop1 .

    ?obs2 a ein:Observation ;
          ein:inNeighborhood ?nb ;
          ein:inYear ?year2 ;
          ein:livabilityScore ?score2 ;
          ein:population ?pop2 .

    ?nb ein:hasName ?neighborhood .

    OPTIONAL { ?obs1 ein:totalDwellings ?dwellings1 . 
               ?obs2 ein:totalDwellings ?dwellings2 . }
    OPTIONAL { ?obs1 ein:avgWozValue ?woz1 . 
               ?obs2 ein:avgWozValue ?woz2 . }
    OPTIONAL { ?obs1 ein:safetyRating ?safety1 . 
               ?obs2 ein:safetyRating ?safety2 . }
    OPTIONAL { ?obs1 ein:facilitiesDeviation ?facDev1 . 
               ?obs2 ein:facilitiesDeviation ?facDev2 . }
    OPTIONAL { ?obs1 ein:housingStockDeviation ?housDev1 . 
               ?obs2 ein:housingStockDeviation ?housDev2 . }

    FILTER (?year1 < ?year2)
    FILTER (?year1 IN (2014, 2016, 2018, 2020, 2022))
    FILTER (?year2 IN (2016, 2018, 2020, 2022, 2024))
    FILTER (?year2 = ?year1 + 2)
}
ORDER BY ?neighborhood ?year1
```

The goal of this query is to provide aligned deltas per neighborhood in consecutive years to show how differences in features correspond to changes in livability.

**Analysis** - 
- Compute correlation between livabilityDelta and each other delta column across all neighborhood-year pairs in pandas/NumPy.
- Count what percentage of neighborhood-year pairs show population increasing while livability decreases.
- Identify neighborhoods that consistently show negative livabilityDelta alongside positive populationDelta across multiple year pairs which form the most relevant cases for supporting our discussion.
- Check whether dwellingsDelta keeps pace with populationDelta because if population grows faster than dwellings, it suggests housing supply is not meeting demand, which could explain livability decline.
- Compare facilitiesDeviationDelta against populationDelta because if facilities deteriorate relative to the national average as population grows, it supports the infrastructure scaling hypothesis.

#### Query 4 - National Average Comparison

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?year
       (AVG(?livabilityScore) as ?avgLivability)
       (AVG(?livabilityDeviation) as ?avgLivabilityDeviation)
       (AVG(?physicalDev) as ?avgPhysicalDeviation)
       (AVG(?nuisanceDev) as ?avgNuisanceDeviation)
       (AVG(?socialDev) as ?avgSocialDeviation)
       (AVG(?facilitiesDev) as ?avgFacilitiesDeviation)
       (AVG(?housingDev) as ?avgHousingDeviation)
       (SAMPLE(?dimFlag) as ?dimensionScoresAvailable)
WHERE {
    ?obs a ein:Observation ;
         ein:inYear ?year ;
         ein:livabilityScore ?livabilityScore .

    OPTIONAL { ?obs ein:livabilityDeviation ?livabilityDeviation . }
    OPTIONAL { ?obs ein:dimensionScoresAvailable ?dimFlag . }

    OPTIONAL {
        ?obs ein:dimensionScoresAvailable true ;
             ein:physicalEnvironmentDeviation ?physicalDev ;
             ein:nuisanceInsecurityDeviation ?nuisanceDev ;
             ein:socialCohesionDeviation ?socialDev ;
             ein:facilitiesDeviation ?facilitiesDev ;
             ein:housingStockDeviation ?housingDev .
    }
}
GROUP BY ?year
ORDER BY ?year
```

The goal of this query is to returns measurement years where livability exists, and shows which sub-dimensions are above or below the national average, where a negative deviation means Eindhoven is performing worse than the national average on that dimension.

**Analysis** - 
- We can profile the livability in Eindhoven by identifying if sub-dimensions are consistently negative across measurement years.
- Check whether any sub-dimension shows a worsening trend over time.

#### Query 5 - Fastest Improving and Declining Neighborhoods

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?neighborhood
       (MIN(?livabilityScore) as ?minScore)
       (MAX(?livabilityScore) as ?maxScore)
       (MAX(?livabilityScore) - MIN(?livabilityScore) as ?scoreRange)
       (AVG(?livabilityScore) as ?avgScore)
       (SAMPLE(?urbanityClass) as ?urbanityClass)
WHERE {
    ?obs a ein:Observation ;
         ein:inNeighborhood ?nb ;
         ein:inYear ?year ;
         ein:livabilityScore ?livabilityScore .
    ?nb ein:hasName ?neighborhood .
    OPTIONAL { ?obs ein:urbanityClass ?urbanityClass . }
}
GROUP BY ?neighborhood
ORDER BY DESC(?scoreRange)
LIMIT 20

PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?neighborhood ?year ?livabilityScore
WHERE {
    ?obs a ein:Observation ;
         ein:inNeighborhood ?nb ;
         ein:inYear ?year ;
         ein:livabilityScore ?livabilityScore .
    ?nb ein:hasName ?neighborhood .
}
ORDER BY ?neighborhood ?year
```

The goal of this query is to identify the 20 neighborhoods with the largest livability score range across all measurement years, capturing both the fastest improvers and most severe decliners.

**Analysis** - 
- Check whether the highest-volatility neighborhoods cluster in a particular urbanity class or area of Eindhoven
- Cross-reference declining neighborhoods with the Query 2 population deltas to see if the neighborhoods with the largest livability drops show the largest population increases.

#### Query 6 - Urbanity Class Comparison

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?urbanityClass ?year
       (AVG(?livabilityScore) as ?avgLivability)
       (AVG(?population) as ?avgPopulation)
       (COUNT(DISTINCT ?nb) as ?neighborhoodCount)
WHERE {
    ?obs a ein:Observation ;
         ein:inNeighborhood ?nb ;
         ein:inYear ?year ;
         ein:urbanityClass ?urbanityClass ;
         ein:population ?population .
    OPTIONAL { ?obs ein:livabilityScore ?livabilityScore . }
}
GROUP BY ?urbanityClass ?year
ORDER BY ?urbanityClass ?year
```

The goal of this query is to use CBS urbanity class to show how neighborhoods of different urban density types perform differently over time.

**Analysis** - 
- Compare the livability trend within each urbanity class.
- If livability deterioration is concentrated in specific urbanity classes, interventions can be targeted accordingly.

## Research Question #2 - 

### What facilities need to grow faster or what other considerable measures should the municipality of Eindhoven take to accommodate the growth in population?

#### Query 1 - Facility Access in High-Growth Neighborhoods

```
PREFIX ein: <http://eindhoven.nl/livability/>
SELECT ?name (?p2 - ?p1 AS ?growth)
       ?supermarket ?groceries ?gp ?pharmacy ?primarySchool ?daycare ?trainStation
WHERE {
  ?nb ein:hasName ?name ; ein:hasObservation ?o1 , ?o2 .
  ?o1 ein:inYear 2015 ; ein:population ?p1 .
  ?o2 ein:inYear 2025 ; ein:population ?p2 ;
      ein:distSupermarket    ?supermarket ;
      ein:distDailyGroceries ?groceries ;
      ein:distGpPractice     ?gp ;
      ein:distPharmacy       ?pharmacy ;
      ein:distPrimarySchool  ?primarySchool ;
      ein:distDaycare        ?daycare ;
      ein:distTrainStation   ?trainStation .
  FILTER (?p2 > ?p1)
}
ORDER BY DESC(?growth)
```

The goal of this query is to identify neighborhoods that experienced population growth between 2015 and 2025 and show their 2025 distances to essential facilities, ordered by population growth to outline the neighborhoods under the most pressure.

**Analysis** - 
- Identify neighborhoods with the highest population growth and check whether their facility distances are above average.
- Cross-reference with Query 3 (average essential distance) to check neighborhoods appearing in both the top growth and top distance lists.
    - These are neighborhoods where infrastructure is most clearly failing to keep pace.

#### Query 2 - Population Growth versus Housing Growth.

```
PREFIX ein: <http://eindhoven.nl/livability/>
SELECT ?name (?p2 - ?p1 AS ?popGrowth) (?dw2 - ?dw1 AS ?dwellingGrowth) ?pctOccupied2025
WHERE {
  ?nb ein:hasName ?name ; ein:hasObservation ?o1 , ?o2 .
  ?o1 ein:inYear 2015 ; ein:population ?p1 ; ein:totalDwellings ?dw1 .
  ?o2 ein:inYear 2025 ; ein:population ?p2 ; ein:totalDwellings ?dw2 ;
      ein:pctOccupied ?pctOccupied2025 .
  FILTER (?p2 > ?p1)
}
ORDER BY DESC(?popGrowth)
```

The goal of this query is to compare population growth against dwelling growth between 2015 and 2025 alongside the 2025 occupancy rate, directly testing whether housing supply is keeping pace with population demand.

**Analysis** - 
- Compute the ratio of dwellingGrowth to popGrowth in pandas where a ratio below 1 means population is growing faster than housing supply.
- Flag neighborhoods where pctOccupied2025 is close to 1.0 which are neighborhoods with full capacity and further population growth cannot be managed.

#### Query 3 - Population Growth Ranked By Distances To Facilities

```
PREFIX ein: <http://eindhoven.nl/livability/>
SELECT ?name (?p2 - ?p1 AS ?growth)
       ((?super + ?groc + ?gp + ?pharm + ?prim + ?day + ?train) / 7 AS ?avgEssentialKm)
       ?train
WHERE {
  ?nb ein:hasName ?name ; ein:hasObservation ?o1 , ?o2 .
  ?o1 ein:inYear 2015 ; ein:population ?p1 .
  ?o2 ein:inYear 2025 ; ein:population ?p2 ;
      ein:distSupermarket    ?super ;
      ein:distDailyGroceries ?groc ;
      ein:distGpPractice     ?gp ;
      ein:distPharmacy       ?pharm ;
      ein:distPrimarySchool  ?prim ;
      ein:distDaycare        ?day ;
      ein:distTrainStation   ?train .
  FILTER (?p2 - ?p1 > 500)
}
ORDER BY DESC(?avgEssentialKm)
```

The goal of this query is to analyze neighborhoods with population growth exceeding 500 residents between 2015 and 2025 and rank them by a composite average distance across seven essential facility types, identifying where infrastructure is most clearly failing to keep pace with growth.

**Analysis** - 
- Neighborhoods at the top of this list (high growth AND high average distance) are the primary candidates for infrastructure investment recommendations to the municipality.
- For each neighborhood, identify which specific facility is driving the high average.

#### Query 4 - Average Distances Over Time

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?year
       (AVG(?distGp) as ?avgDistGp)
       (AVG(?distHospital) as ?avgDistHospital)
       (AVG(?distPrimary) as ?avgDistPrimary)
       (AVG(?distSecondary) as ?avgDistSecondary)
       (AVG(?distTrain) as ?avgDistTrain)
       (AVG(?distSupermarket) as ?avgDistSupermarket)
       (AVG(?distPharmacy) as ?avgDistPharmacy)
       (AVG(?distDaycare) as ?avgDistDaycare)
       (SUM(?population) as ?totalPopulation)
WHERE {
    ?obs a ein:Observation ;
         ein:inYear ?year ;
         ein:population ?population .
    OPTIONAL { ?obs ein:distGpPractice ?distGp . }
    OPTIONAL { ?obs ein:distHospital ?distHospital . }
    OPTIONAL { ?obs ein:distPrimarySchool ?distPrimary . }
    OPTIONAL { ?obs ein:distSecondarySchool ?distSecondary . }
    OPTIONAL { ?obs ein:distTrainStation ?distTrain . }
    OPTIONAL { ?obs ein:distSupermarket ?distSupermarket . }
    OPTIONAL { ?obs ein:distPharmacy ?distPharmacy . }
    OPTIONAL { ?obs ein:distDaycare ?distDaycare . }
}
GROUP BY ?year
ORDER BY ?year
```

The goal of this query is to track whether average distances to essential facilities across all Eindhoven neighborhoods have increased, decreased, or remained stable over the full 2015–2025 period alongside total population growth.

**Analysis** - 
- Plot each facility distance as a line over time alongside total population, if distances increase as population grows, it directly answers RQ2.
- A flat or decreasing distance trend despite population growth would indicate facilities are being added proportionally.

#### Query 5 - Access To Facilities (Worst)

```
PREFIX ein: <http://eindhoven.nl/livability/>

SELECT ?neighborhood ?distGp ?distHospital 
       ?distPrimary ?distSecondary ?distTrain
       ?population
WHERE {
    ?obs a ein:Observation ;
         ein:inNeighborhood ?nb ;
         ein:inYear ?year ;
         ein:population ?population .
    ?nb ein:hasName ?neighborhood .
    OPTIONAL { ?obs ein:distGpPractice ?distGp . }
    OPTIONAL { ?obs ein:distHospital ?distHospital . }
    OPTIONAL { ?obs ein:distPrimarySchool ?distPrimary . }
    OPTIONAL { ?obs ein:distSecondarySchool ?distSecondary . }
    OPTIONAL { ?obs ein:distTrainStation ?distTrain . }
    FILTER (?year = 2024)
}
ORDER BY DESC(?distGp)
LIMIT 10
```

The goal of this query is to identify the ten neighborhoods with the worst access to key facilities in 2024, providing a concrete list of the most "at-risk" neighborhoods in Eindhoven.

**Analysis** - 
- Compare population size of worst-access neighborhoods as a large population with poor access is more critical than a small one.