# Construct Queries
These queries both construct the KWG-lite subgraph and re-bind the KnowWhereGraph entity to the KWG-lite namespace.

## Prefixes
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX kwg-ont: <http://stko-kwg.geog.ucsb.edu/lod/ontology/>
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX kwgr: <http://stko-kwg.geog.ucsb.edu/lod/resource/>
PREFIX kwglr: <http://stko-kwg.geog.ucsb.edu/lod/lite-resource/>
PREFIX kwgl-ont: <http://stko-kwg.geog.ucsb.edu/lod/lite-ontology/>
PREFIX usgs: <http://gnis-ld.org/lod/usgs/ontology/>
PREFIX gnis: <http://gnis-ld.org/lod/gnis/ontology/>
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX gn: <http://www.geonames.org/ontology#>
```

## Query 1
* Places that have been impacted by a Hazard.
```sparql
select * where
{ 
  ?p kwgl-ont:impactedBy ?h.
	?h kwgl-ont:hasImpacted ?p.
}
 ```
 
## Query 2
* Retrieve the following properties associated with a Hazard Event :
*  a. affectedAreaInAcres 
```sparql
select * where { ?h kwgl-ont:affectedAreaInAcres ?area_burnt.}
 ```
 
 * b. numberOfDeaths
 
```sparql
select * where { ?h kwgl-ont:numberOfDeaths ?death.}
```

* c. damageToInfrastructureInDollars
```sparql
select * where{ ?h kwgl-ont:damageToInfrastructureInDollars ?property_damage.}
 ```
 
 * d. damageToCropsInDollars 
 ```sparql
select * where { ?h kwgl-ont:damageToCropsInDollars ?crop_damage.}
 ```
 ## Query 3
 
 * Get all hazard events, their name, start date, end date, and KWG entity
 ```sparql
select * where { ?h a kwgl-ont:HazardEvent.
    		?h kwgl-ont:hasName ?name.
			?h kwgl-ont:hasStartDate ?startTime.
			?h kwgl-ont:hasEndDate ?endTime.
			?h kwgl-ont:hasKWGEntity ?hazard_uri.
}
 ```
 
 ## Query 4
 * Get all places, their name, and KWG entity 
 
```sparql
select * where{ ?p a kwgl-ont:Place.
    		?p kwgl-ont:hasName ?name.
			?p kwgl-ont:hasKWGEntity ?place_uri.
}
 ```
 
 ## Query 5
 
 * Get the following health attributes of places: percentObese, percentBelowPovertyLine, percentDiabetic
 
```sparql
select * where { ?p kwgl-ont:percentObese ?obesity.
			?p kwgl-ont:percentBelowPovertyLine ?below_poverty_line.
			?p kwgl-ont:percentDiabetic ?diabetes.
}
```

## Query 6

* Get the following demographic attributes associated with a place hasPopulation, hasNumberOfHouseHolds

```sparql
select * where { ?p kwgl-ont:hasPopulation ?totalPop.
			?p kwgl-ont:hasNumberOfHouseHolds ?householdUnits.
}
```

## Query 7

* Get the following fire-specific attributes related to a place :
* a. numberOfFiresImpactingPlace2021 :
```sparql
select * where { #?p kwgl-ont:numberOfFiresImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfFiresImpactingPlace2019 ?count.
   			#?p kwgl-ont:numberOfFiresImpactingPlace2020 ?count.
            ?p kwgl-ont:numberOfFiresImpactingPlace2021 ?count.
            #?p kwgl-ont:numberOfFiresImpactingPlace2022 ?count.
}
```

* b. dollarDamageOfFiresImpactingPlace2021
```sparql


```

## Query 8
 * Get the following hurricane-specific attributes relate to a place: 
 *  a. numberOfHurricanesImpactingPlace2021
 Issue with triplification 

* b. dollarDamageOfHurricanesImpactingPlace2021

```sparql
select * where{ #?p kwgl-ont:numberOfHurricanesImpactingPlace2018 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2019 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2020 ?count.
    ?p kwgl-ont:numberOfHurricanesImpactingPlace2021 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2022 ?count.
}
```

* c. Dollar damage

```sparql
```

## Query 9

* Number of earthquakes : ( not running )

```sparql
select * where { ?p kwgl-ont:numberOfEarthquakesImpactingPlace2018 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2019 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2020 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2021 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2022 ?count.
}
 ```
 
 ## Query 10

* Get the following tornado-specific attributes related to a a place: 
* a. numberOfTornadoesImpactingPlace2021

```sparql
select * where { #?p kwgl-ont:numberOfTornadoesImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2019 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2020 ?count.
    		?p kwgl-ont:numberOfTornadoesImpactingPlace2021 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2022 ?count.
}
```

* b. Dollar damage
???

## Query 11

* Get the following storm-specific attributes related to a a place: 
* a. Number of storm surges

```sparql
select * where { ?p kwgl-ont:numberOfStormSurgesImpactingPlace2018 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2019 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2020 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2021 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2022 ?count.
}
```

* b. Dollar damage

```sparql
```

## Query 12

* Get the following flood-specific attributes related to a a place: 
* a. numberOfFloodsImpactingPlace2021

```sparql
select * where { #?p kwgl-ont:numberOfFloodsImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2019 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2020 ?count.
   		 	?p kwgl-ont:numberOfFloodsImpactingPlace2021 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2022 ?count.
}
```

* b. dollarDamageOfFloodsImpactingPlace2021

```sparql
```

## Query 13

* Get the following landslide-specific attributes related to a a place: 
* a. averageHeatingDegreeDaysPerMonth

```sparql
select * where { #?p kwgl-ont:averageCoolingDegreeDaysPerMonthJan2021 ?cdd_mean ## uncomment to use ?p if below BIND statements work
    ?climate_division kwgl-ont:averageCoolingDegreeDaysPerMonthJan2021 ?cdd_mean. #change predicate month according to the month in the FILTER below
}
 ```
 
 * b. averageCoolingDegreeDaysPerYear
 
```sparql
select * where { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:averageHeatingDegreeDaysPerMonthDec2021 ?hdd_mean. #change predicate month according to the month in the FILTER below
}
```

* c. averageMonthlyTemperatureInCelsius

```sparql
select * where { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:averageMonthlyTemperatureInCelsiusJan2021 ?avg_temp_cel. #change predicate month according to the month in the FILTER below
}
```

* d. maximumMonthlyTemperatureInCelsius

```sparql
select * where{ ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:maxMonthlyTemperatureInCelsiusDec2021 ?max_temp_cel. #change predicate month according to the month in the FILTER below
}
```

* e. minimumMonthlyTemperatureInCelsius

```sparql
select * where { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:minMonthlyTemperatureInCelsiusDec2021 ?min_temp_cel. #change predicate month according to the month in the FILTER below
}
```

## Query 15

* Get all places and spatial relationships with each other 

```sparql
select * where { ?p1 ?spatial_relation ?p2.
}
```

## Query 16

* Get all fires and the type of Fire

```sparql
select * where { ?h a kwgl-ont:Fire.
    ?h kwgl-ont:isOfFireType ?ft.
}
 ```
 
 ## Query 17
 * Get all hurricanes
 
```sparql
select * where { ?h a kwgl-ont:Hurricane.
}
```

## Query 18

* Get all tornadoes

```sparql
select * where{ ?h a kwgl-ont:Tornado.
}
```

## Query 19

* Get all earthquakes

```sparql
select * where { ?h a kwgl-ont:Earthquake.
}
```





 
 
 

