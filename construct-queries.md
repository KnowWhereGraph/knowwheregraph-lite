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
CONSTRUCT 
{ 
  ?p kwgl-ont:impactedBy ?h.
	?h kwgl-ont:hasImpacted ?p.
}
WHERE
{
   ?place a kwg-ont:Region.
   ?place kwg-ont:spatialRelation ?hazard.
   ?hazard a kwg-ont:Hazard.
   BIND(STRAFTER(STR(?place), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
   BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
   BIND( IRI(?litePlace) AS ?p ).
   
   BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
   BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
   BIND( IRI(?liteHazard) AS ?h ).
}
 ```
 
## Query 2
* Retrieve the following properties associated with a Hazard Event :
*  a. affectedAreaInAcres 
```sparql
CONSTRUCT { ?h kwgl-ont:affectedAreaInAcres ?area_burnt.}
WHERE
 {
    {?hazard a kwg-ont:NIFCFire} UNION {?hazard a kwg-ont:MTBSFire}.
    ?hazard	sosa:isFeatureOfInterestOf ?obs.
    ?obs sosa:hasSimpleResult ?area_burnt.
    {?obs sosa:observedProperty kwgr:nifcFireObservableProperty.calculatedAcres} UNION {?obs sosa:observedProperty kwgr:mtbsFireObservableProperty.NumberOfAcresBurned}.
    
    BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
 ```
 
 * b. numberOfDeaths
 
```sparql
CONSTRUCT { ?h kwgl-ont:numberOfDeaths ?death.}
WHERE
 {
    ?hazard a kwg-ont:NOAAHazardEvent;
            sosa:isFeatureOfInterestOf ?obs_collection.
    ?obs_collection a kwg-ont:ImpactObservationCollection;
                    sosa:hasMember ?obs_dir_death;
                    sosa:hasMember ?obs_indir_death.
    ?obs_dir_death sosa:hasSimpleResult ?direct_death;
    	 sosa:observedProperty kwgr:impactObservableProperty.deathDirect.
	?obs_indir_death sosa:hasSimpleResult ?indirect_death;
    	 sosa:observedProperty kwgr:impactObservableProperty.deathIndirect.
    BIND((?direct_death + ?indirect_death) AS ?death)   
    FILTER (?death > 0)
    
    BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
#GROUP BY ?hazard
```

* c. damageToInfrastructureInDollars
```sparql
CONSTRUCT { ?h kwgl-ont:damageToInfrastructureInDollars ?property_damage.}
WHERE
 {
    ?hazard a kwg-ont:NOAAHazardEvent;
    		sosa:isFeatureOfInterestOf ?obs_collection.
    ?obs_collection a kwg-ont:ImpactObservationCollection;
                    sosa:hasMember ?obs.
    ?obs sosa:hasSimpleResult ?property_damage;
    	 sosa:observedProperty kwgr:impactObservableProperty.damageProperty.
    FILTER (?property_damage > 0)
    
    BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
 ```
 
 * d. damageToCropsInDollars 
 ```sparql
 CONSTRUCT { ?h kwgl-ont:damageToCropsInDollars ?crop_damage.}
WHERE
 {
    ?hazard a kwg-ont:NOAAHazardEvent;
    		sosa:isFeatureOfInterestOf ?obs_collection.
    ?obs_collection a kwg-ont:ImpactObservationCollection;
                    sosa:hasMember ?obs.
    ?obs sosa:hasSimpleResult ?crop_damage;
    	 sosa:observedProperty kwgr:impactObservableProperty.damageCrop.
    FILTER (?crop_damage > 0)
    
    BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
     
 }
 ```
 ## Query 3
 
 * Get all hazard events, their name, start date, end date, and KWG entity
 ```sparql
 CONSTRUCT { ?h a kwgl-ont:HazardEvent.
    		?h kwgl-ont:hasName ?name.
			?h kwgl-ont:hasStartDate ?startTime.
			?h kwgl-ont:hasEndDate ?endTime.
			?h kwgl-ont:hasKWGEntity ?hazard_uri.
}
WHERE
 {
    ?hazard a kwg-ont:Hazard;
    		rdfs:label ?name;
      		kwg-ont:hasTemporalScope ?time.
     ?time time:hasBeginning/time:inXSDDateTime | time:inXSDDate ?startTime;
          time:hasEnd/time:inXSDDateTime | time:inXSDDate ?endTime .
    BIND(REPLACE(str(?hazard),str("kwgr:"), str("http://stko-kwg.geog.ucsb.edu/lod/resource/")) as ?uri)
    #uncomment below line if you want URI format but then ?hazard and ?hazard_uri would be the same 
    BIND(URI(?uri) as ?hazard_uri)
    
    BIND(STRAFTER(STR(?hazard), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
 ```
 
 ## Query 4
 * Get all places, their name, and KWG entity 
 
```sparql
CONSTRUCT { ?p a kwgl-ont:Place.
    		?p kwgl-ont:hasName ?name.
			?p kwgl-ont:hasKWGEntity ?place_uri.
}
WHERE
 {
    {?place a kwg-ont:Region} UNION {?place a gnis:County} UNION {?place a kwg-ont:State}.
    ?place rdfs:label ?name.

    {BIND(REPLACE(str(?place),str("kwgr:"), str("http://stko-kwg.geog.ucsb.edu/lod/resource/")) as ?uri)} UNION
    {BIND(str(?place) as ?uri)}
    #uncomment below line if you want URI format but then ?hazard and ?hazard_uri would be the same 
    BIND(URI(?uri) as ?place_uri)

    BIND(STRAFTER(STR(?place), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
    
     
 }
 ```
 
 ## Query 5
 
 * Get the following health attributes of places: percentObese, percentBelowPovertyLine, percentDiabetic
 
```sparql
CONSTRUCT { ?p kwgl-ont:percentObese ?obesity.
			?p kwgl-ont:percentBelowPovertyLine ?below_poverty_line.
			?p kwgl-ont:percentDiabetic ?diabetes.
}
WHERE
 {
    ?oc a kwg-ont:PublicHealthObservationCollection;
        sosa:hasFeatureOfInterest ?adminRegion;
    	sosa:hasMember ?obs_bpl;
     	sosa:hasMember ?obs_ob;
        sosa:hasMember ?obs_diab.
    # retrieve observations and results
    ?obs_bpl sosa:hasSimpleResult ?below_poverty_line;
         sosa:observedProperty kwgr:publicHealthObservableProperty.belowPovertyLevelPercent.
    ?obs_ob sosa:hasSimpleResult ?obesity;
         sosa:observedProperty kwgr:publicHealthObservableProperty.obesityAgeAdjusted20PlusPercent.
	?obs_diab sosa:hasSimpleResult ?diabetes;
         sosa:observedProperty kwgr:publicHealthObservableProperty.diabetesAgeAdjusted20PlusPercent.
    
     BIND(STRAFTER(STR(?adminRegion), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }

```

## Query 6

* Get the following demographic attributes associated with a place hasPopulation, hasNumberOfHouseHolds

```sparql
CONSTRUCT { ?p kwgl-ont:hasPopulation ?totalPop.
			?p kwgl-ont:hasNumberOfHouseHolds ?householdUnits.
}
WHERE
 {
    ?oc a kwg-ont:CensusACS5YearEstimatesObservationCollection;
    	sosa:hasMember ?obs_tp;
        sosa:hasMember ?obs_hu;
     	sosa:hasFeatureOfInterest ?census_msa.
    # retrieve observations and results
    ?obs_tp sosa:hasSimpleResult ?totalPop;
         sosa:observedProperty kwgr:censusObservableProperty.totalPopulation.
    ?obs_hu sosa:hasSimpleResult ?householdUnits;
         sosa:observedProperty kwgr:censusObservableProperty.householdUnits.
         #sosa:isMemberOf kwg-ont:CensusACS5YearEstimatesObservationCollection.
    ?census_msa kwg-ont:sfWithin ?adminRegion.
    
       BIND(STRAFTER(STR(?census_msa), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }
```

## Query 7

* Get the following fire-specific attributes related to a place :
* a. numberOfFiresImpactingPlace2021 :
```sparql
CONSTRUCT { ?p kwgl-ont:numberOfFiresImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfFiresImpactingPlace2019 ?count.
   			#?p kwgl-ont:numberOfFiresImpactingPlace2020 ?count.
            #?p kwgl-ont:numberOfFiresImpactingPlace2021 ?count.
            #?p kwgl-ont:numberOfFiresImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?nifc_fire) AS ?count) 
		WHERE
 		{
     		?nifc_fire a kwg-ont:NIFCFire; # we only count NIFC Fire since we do not have coresolution yet
    					kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:inXSDgYear ?year .
    		FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            #FILTER (?year = "2022"^^xsd:gYear)
             
 		}
		GROUP BY ?admin_region
        
        }
    
    BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
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
CONSTRUCT { ?p kwgl-ont:numberOfHurricanesImpactingPlace2018 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2019 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2020 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2021 ?count.
   # ?p kwgl-ont:numberOfHurricanesImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?hurricane) AS ?count) 
		WHERE
 		{
            {?hurricane a kwg-ont:NOAAHurricane} UNION {?hurricane a kwg-ont:NOAAMarineHurricaneTyphoon}
    		 ?hurricane kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:hasBeginning/time:inXSDgYear | time:inXSDgYear ?year .
    		FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            #FILTER (?year = "2022"^^xsd:gYear)
 		}
		GROUP BY ?admin_region
    }
    BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }

```

* c. Dollar damage

???

## Query 9

* Number of earthquakes : ( not running )

```sparql
CONSTRUCT { ?p kwgl-ont:numberOfEarthquakesImpactingPlace2018 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2019 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2020 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2021 ?count.
    #?p kwgl-ont:numberOfEarthquakesImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?earthquake) AS ?count) 
		WHERE
 		{
     		?earthquake a kwg-ont:Earthquake; 
    					kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:inXSDgYear ?year .
    		FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            #FILTER (?year = "2022"^^xsd:gYear)
 		}
		GROUP BY ?admin_region
    }
     BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }
 ```
 
 ## Query 10

* Get the following tornado-specific attributes related to a a place: 
* a. numberOfTornadoesImpactingPlace2021

```sparql
CONSTRUCT { #?p kwgl-ont:numberOfTornadoesImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2019 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2020 ?count.
    		#?p kwgl-ont:numberOfTornadoesImpactingPlace2021 ?count.
    		?p kwgl-ont:numberOfTornadoesImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?tornado) AS ?count) 
		WHERE
 		{
            ?tornado a kwg-ont:NOAATornado;
                        kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:hasBeginning/time:inXSDgYear | time:inXSDgYear ?year .
    		#FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            FILTER (?year = "2022"^^xsd:gYear)
 		}
		GROUP BY ?admin_region
    }
     BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }
```

* b. Dollar damage
???

## Query 11

* Get the following storm-specific attributes related to a a place: 
* a. Number of storm surges

```sparql
CONSTRUCT { ?p kwgl-ont:numberOfStormSurgesImpactingPlace2018 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2019 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2020 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2021 ?count.
    #?p kwgl-ont:numberOfStormSurgesImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?storm_surge) AS ?count) 
		WHERE
 		{
            ?storm_surge a kwg-ont:NOAAStormSurgeTide;
                        kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:hasBeginning/time:inXSDgYear | time:inXSDgYear ?year .
    		FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            #FILTER (?year = "2022"^^xsd:gYear)
 		}
		GROUP BY ?admin_region
    }
     BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }
```

* b. Dollar damage

???

## Query 12

* Get the following flood-specific attributes related to a a place: 
* a. numberOfFloodsImpactingPlace2021

```sparql
CONSTRUCT { ?p kwgl-ont:numberOfFloodsImpactingPlace2018 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2019 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2020 ?count.
   		 	#?p kwgl-ont:numberOfFloodsImpactingPlace2021 ?count.
    		#?p kwgl-ont:numberOfFloodsImpactingPlace2022 ?count.
}
WHERE
 {
    {
        SELECT ?admin_region (COUNT (DISTINCT ?flood) AS ?count) 
		WHERE
 		{
            {?flood a kwg-ont:NOAAFlood} UNION {?flood a kwg-ont:NOAAFlashFlood} UNION {?flood a kwg-ont:NOAACoastalFlood} UNION {?flood a kwg-ont:NOAALakeshoreFlood}
    		 ?flood kwg-ont:spatialRelation ?admin_region;
    					kwg-ont:hasTemporalScope ?time .
    		?admin_region a kwg-ont:AdministrativeRegion.
    		?time time:hasBeginning/time:inXSDgYear | time:inXSDgYear ?year .
    		FILTER (?year = "2018"^^xsd:gYear)
            #FILTER (?year = "2019"^^xsd:gYear)
            #FILTER (?year = "2020"^^xsd:gYear)
            #FILTER (?year = "2021"^^xsd:gYear)
            #FILTER (?year = "2022"^^xsd:gYear)
 		}
		GROUP BY ?admin_region
    }
    BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p ).
 }
```

* b. dollarDamageOfFloodsImpactingPlace2021
???

## Query 13

* Get the following landslide-specific attributes related to a a place: 
* a. averageHeatingDegreeDaysPerMonth

```sparql
CONSTRUCT { #?p kwgl-ont:averageCoolingDegreeDaysPerMonthJan2021 ?cdd_mean ## uncomment to use ?p if below BIND statements work
    ?climate_division kwgl-ont:averageCoolingDegreeDaysPerMonthJan2021 ?cdd_mean. #change predicate month according to the month in the FILTER below
}
WHERE
 {
    ?climate_division a kwg-ont:USClimateDivision;
                  		sosa:isFeatureOfInterestOf ?obs_collection.
    	   ?obs_collection sosa:hasMember ?obs_collection_cdd.
    		
    		?obs_collection_cdd sosa:hasMember ?obs_hdd;
    	 		sosa:observedProperty kwgr:climateObservableProperty.CDD.
     		?obs_cdd sosa:hasResult ?cdd;
                      sosa:resultTime ?cdd_months.
    		?cdd kwg-ont:mean/qudt:numericValue ?cdd_mean.
    		BIND(SUBSTR(STR(?cdd_months), 1, 4) as ?year)
			BIND(SUBSTR(STR(?cdd_months), 6, 2) as ?month)
    	    FILTER (?cdd_mean >0)
    FILTER (?year = '2021')#change year according to the predicate year and generate results
    		FILTER (?month = '01') #Jan
    		#FILTER (?month = '02') #Feb
    		#FILTER (?month = '03') #Mar
    		#FILTER (?month = '04') #Apr
    		#FILTER (?month = '05') #May
    		#FILTER (?month = '06') #Jun
    		#FILTER (?month = '07') #Jul
    		#FILTER (?month = '08') #Aug
    		#FILTER (?month = '09') #Sep
    		#FILTER (?month = '10') #Oct
    		#FILTER (?month = '11') #Nov
    		#FILTER (?month = '12') #Dec
    
    ## commented out because query times out. Instead the kwgr prefix is replaced with kwglr in the output ttls
    #BIND(STRAFTER(STR(?admin_region), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    #BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    #BIND( IRI(?litePlace) AS ?p ).
 }
 ```
 
 * b. averageCoolingDegreeDaysPerYear
 
```sparql
CONSTRUCT { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:averageHeatingDegreeDaysPerMonthDec2021 ?hdd_mean. #change predicate month according to the month in the FILTER below
}
WHERE
 {
    ?climate_division a kwg-ont:USClimateDivision;
                  		sosa:isFeatureOfInterestOf ?obs_collection.
    	   ?obs_collection sosa:hasMember ?obs_collection_hdd.
    		
    		?obs_collection_hdd sosa:hasMember ?obs_hdd;
    	 		sosa:observedProperty kwgr:climateObservableProperty.CDD.
     		?obs_hdd sosa:hasResult ?hdd;
                      sosa:resultTime ?hdd_months.
    		?hdd kwg-ont:mean/qudt:numericValue ?hdd_mean.
    		BIND(SUBSTR(STR(?hdd_months), 1, 4) as ?year)
			BIND(SUBSTR(STR(?hdd_months), 6, 2) as ?month)
    	    FILTER (?hdd_mean >0)
    FILTER (?year = '2021')#change year according to the predicate year and generate results
    		#FILTER (?month = '01') #Jan
    		#FILTER (?month = '02') #Feb
    		#FILTER (?month = '03') #Mar
    		#FILTER (?month = '04') #Apr
    		#FILTER (?month = '05') #May
    		#FILTER (?month = '06') #Jun
    		#FILTER (?month = '07') #Jul
    		#FILTER (?month = '08') #Aug
    		#FILTER (?month = '09') #Sep
    		#FILTER (?month = '10') #Oct
    		#FILTER (?month = '11') #Nov
    		FILTER (?month = '12') #Dec
    
 }
```

* c. averageMonthlyTemperatureInCelsius

```sparql
CONSTRUCT { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:averageMonthlyTemperatureInCelsiusJan2021 ?avg_temp_cel. #change predicate month according to the month in the FILTER below
}
WHERE
 {
    ?climate_division a kwg-ont:USClimateDivision;
                  		sosa:isFeatureOfInterestOf ?obs_collection.
    	   ?obs_collection sosa:hasMember ?obs_collection_avg_temp.
    		
    		?obs_collection_avg_temp sosa:hasMember ?obs_avg_temp;
    	 		sosa:observedProperty kwgr:climateObservableProperty.avgTemp.
     		?obs_avg_temp sosa:hasResult ?temp;
                      sosa:resultTime ?temp_months.
    		?temp kwg-ont:mean/qudt:numericValue ?avg_temp.
    		BIND(SUBSTR(STR(?temp_months), 1, 4) as ?year)
			BIND(SUBSTR(STR(?temp_months), 6, 2) as ?month)
    		BIND((xsd:float(?avg_temp) - 32)/1.8 AS ?avg_temp_cel)
    	    FILTER (?avg_temp_cel >0)
    FILTER (?year = '2021')#change year according to the predicate year and generate results
    		FILTER (?month = '01') #Jan
    		#FILTER (?month = '02') #Feb
    		#FILTER (?month = '03') #Mar
    		#FILTER (?month = '04') #Apr
    		#FILTER (?month = '05') #May
    		#FILTER (?month = '06') #Jun
    		#FILTER (?month = '07') #Jul
    		#FILTER (?month = '08') #Aug
    		#FILTER (?month = '09') #Sep
    		#FILTER (?month = '10') #Oct
    		#FILTER (?month = '11') #Nov
    		#FILTER (?month = '12') #Dec
    
 }
```

* d. maximumMonthlyTemperatureInCelsius

```sparql
CONSTRUCT { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:maxMonthlyTemperatureInCelsiusDec2021 ?max_temp_cel. #change predicate month according to the month in the FILTER below
}
WHERE
 {
    ?climate_division a kwg-ont:USClimateDivision;
                  		sosa:isFeatureOfInterestOf ?obs_collection.
    	   ?obs_collection sosa:hasMember ?obs_collection_max_temp.
    		
    		?obs_collection_max_temp sosa:hasMember ?obs_max_temp;
    	 		sosa:observedProperty kwgr:climateObservableProperty.maxTemp.
     		?obs_max_temp sosa:hasResult ?temp;
                      sosa:resultTime ?temp_months.
    		?temp kwg-ont:mean/qudt:numericValue ?max_temp.
    		BIND(SUBSTR(STR(?temp_months), 1, 4) as ?year)
			BIND(SUBSTR(STR(?temp_months), 6, 2) as ?month)
    		BIND((xsd:float(?max_temp) - 32)/1.8 AS ?max_temp_cel)
    	    FILTER (?max_temp_cel >0)
    FILTER (?year = '2021')#change year according to the predicate year and generate results
    		FILTER (?month = '01') #Jan
    		#FILTER (?month = '02') #Feb
    		#FILTER (?month = '03') #Mar
    		#FILTER (?month = '04') #Apr
    		#FILTER (?month = '05') #May
    		#FILTER (?month = '06') #Jun
    		#FILTER (?month = '07') #Jul
    		#FILTER (?month = '08') #Aug
    		#FILTER (?month = '09') #Sep
    		#FILTER (?month = '10') #Oct
    		#FILTER (?month = '11') #Nov
    		#FILTER (?month = '12') #Dec
    
 }
```

* e. minimumMonthlyTemperatureInCelsius

```sparql
CONSTRUCT { ?climate_division a kwg-lite:Place.
    		?climate_division kwg-lite:minMonthlyTemperatureInCelsiusDec2021 ?min_temp_cel. #change predicate month according to the month in the FILTER below
}
WHERE
 {
    ?climate_division a kwg-ont:USClimateDivision;
                  		sosa:isFeatureOfInterestOf ?obs_collection.
    	   ?obs_collection sosa:hasMember ?obs_collection_min_temp.
    		
    		?obs_collection_min_temp sosa:hasMember ?obs_min_temp;
    	 		sosa:observedProperty kwgr:climateObservableProperty.minTemp.
     		?obs_min_temp sosa:hasResult ?temp;
                      sosa:resultTime ?temp_months.
    		?temp kwg-ont:mean/qudt:numericValue ?min_temp.
    		BIND(SUBSTR(STR(?temp_months), 1, 4) as ?year)
			BIND(SUBSTR(STR(?temp_months), 6, 2) as ?month)
    		BIND((xsd:float(?min_temp) - 32)/1.8 AS ?min_temp_cel)
    	    FILTER (?min_temp_cel >0)
    FILTER (?year = '2021')#change year according to the predicate year and generate results
    		#FILTER (?month = '01') #Jan
    		#FILTER (?month = '02') #Feb
    		#FILTER (?month = '03') #Mar
    		#FILTER (?month = '04') #Apr
    		#FILTER (?month = '05') #May
    		#FILTER (?month = '06') #Jun
    		#FILTER (?month = '07') #Jul
    		#FILTER (?month = '08') #Aug
    		#FILTER (?month = '09') #Sep
    		#FILTER (?month = '10') #Oct
    		#FILTER (?month = '11') #Nov
    		FILTER (?month = '12') #Dec
    
 }
   
```

## Query 15

* Get all places and spatial relationships with each other 

```sparql
CONSTRUCT { ?p1 ?spatial_relation ?p2.
}
WHERE
 {
    # retrive all regions along with their label and location-specific class
    ?region1 a kwg-ont:Region;
            rdfs:label ?region_label;
            a ?class_type;
    		?spatial_relation ?region2. # retrieve the specific spatial relation with another region
    ?class_type rdfs:subClassOf kwg-ont:Region.
    ?region2 a kwg-ont:Region.
    
    BIND(STRAFTER(STR(?region1), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName ) as ?litePlace)
    BIND( IRI(?litePlace) AS ?p1 ).
    
    BIND(STRAFTER(STR(?region2), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?placeName2)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?placeName2 ) as ?litePlace2)
    BIND( IRI(?litePlace2) AS ?p2 ).
    
    #this should be uncommented later when some GH issues are fixed. Currently there are geo relations in the graph, which is incorrect
    #BIND(STRAFTER(STR(?spatial_relation), "http://stko-kwg.geog.ucsb.edu/lod/ontology/") as ?srName)
    #BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?srName ) as ?liteSR)
    #BIND( IRI(?liteSR) AS ?sr ).
 }
```

## Query 16

* Get all fires and the type of Fire

```sparql
CONSTRUCT { ?h a kwgl-ont:Fire.
    ?h kwgl-ont:isOfFireType ?ft.
}
WHERE
 {
     ?fire a kwg-ont:Fire;
            a ?fire_type.
    ?fire_type rdfs:subClassOf kwg-ont:Fire;
    	rdfs:label ?fire_type_label.
    
 	BIND(STRAFTER(STR(?fire), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
    
    
    BIND(STRAFTER(STR(?fire_type), "http://stko-kwg.geog.ucsb.edu/lod/ontology/") as ?fType)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?fType ) as ?liteFType)
    BIND( IRI(?liteFType) AS ?ft ).
 }
 ```
 
 ## Query 17
 * Get all hurricanes
 
```sparql
CONSTRUCT { ?h a kwgl-ont:Hurricane.
}
WHERE
 {
    {?hurricane a kwg-ont:NOAAHurricane} UNION {?hurricane a kwg-ont:NOAAMarineHurricaneTyphoon}
    BIND(STRAFTER(STR(?hurricane), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
```

## Query 18

* Get all tornadoes

```sparql
CONSTRUCT { ?h a kwgl-ont:Tornado.
}
WHERE
 {
    ?tornado a kwg-ont:NOAATornado.
     BIND(STRAFTER(STR(?tornado), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
```

## Query 19

* Get all earthquakes

```sparql
CONSTRUCT { ?h a kwgl-ont:Earthquake.
}
WHERE
 {
    ?earthquake a kwg-ont:Earthquake.
     BIND(STRAFTER(STR(?earthquake), "http://stko-kwg.geog.ucsb.edu/lod/resource/") as ?hazardName)
    BIND(CONCAT( "http://stko-kwg.geog.ucsb.edu/lod/lite-resource/", ?hazardName ) as ?liteHazard)
    BIND( IRI(?liteHazard) AS ?h ).
 }
```

## Query 20

* Get all floods in 2018 impacting a place

```sparql
select * where { ?p kwgl-ont:numberOfFloodsImpactingPlace2018 ?count.}
```

## Query 21

* Get the average of cooling degree days per month of 2021

```sparql
select * where{ 
    ?climate_division kwgl-ont:averageCoolingDegreeDaysPerMonthJan2021 ?cdd_mean. 
}
```

## Query 22

* Select all spatial relations

```sparql
select * where { ?p1 ?spatial_relation ?p2.
}
```

## Query 23

* Select all fire types

```sparql
select * where { ?h a kwgl-ont:Fire.
    ?h kwgl-ont:isOfFireType ?ft.
}
```

# Query 24

* Select all hurricanes

```sparql
select * where { ?h a kwgl-ont:Hurricane.
}
```

## Query 25

* Select all tornadoes

```sparql
select * where{ ?h a kwgl-ont:Tornado.
}
```

## Query 26

* Select all earthquakes

```sparql
select * where { ?h a kwgl-ont:Earthquake.
}
```

## Query 27

* Select the average heating degree days per month in 2021

```sparql
select * where { ?climate_division a kwgl-ont:Place.
    		?climate_division kwgl-ont:averageHeatingDegreeDaysPerMonthDec2021 ?hdd_mean. 
}
```

## Query 28

* Select the average monthly temperature in celcius in 2021

```sparql
select * where { ?climate_division a kwgl-ont:Place.
    ?climate_division kwgl-ont:averageMonthlyTemperatureInCelsiusJan2021 ?avg_temp_cel. }
```

## Query 29

* Select the maximum monthly temperature in celcius in 2021

```sparql
select * where { ?climate_division a kwgl-ont:Place.
    		?climate_division kwgl-ont:maxMonthlyTemperatureInCelsiusDec2021 ?max_temp_cel. #change predicate month according to the month in the FILTER below
}
```

## Query 30

* Select the minimun monthly temperature in celcius in 2021

```sparql
select * where { ?climate_division a kwgl-ont:Place.
    		?climate_division kwgl-ont:minMonthlyTemperatureInCelsiusDec2021 ?min_temp_cel. 
}
```




 
 
 

