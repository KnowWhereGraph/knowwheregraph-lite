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
