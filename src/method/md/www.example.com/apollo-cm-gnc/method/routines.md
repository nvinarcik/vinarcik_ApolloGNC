---
template:
  id: http://www.example.com/apollo-cm-gnc/method/routines
  name: "Routines"
  rank: 1
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Routines

The flight programs (routines) stored in the guidance computer. Programs that depend on the IMU, directly or through another program, are classified by the reasoner as `method:IMUDependentRoutine`; programs that calibrate the IMU are also classified as `method:IMUCalibrator`. Neither classification is asserted here.

The table is driven by `method:RoutineShape`, which also ships with this method as `shapes/www.example.com/apollo-cm-gnc/method/routine-shape.ttl`. The shape flags a routine that calibrates an IMU it does not also depend on.

```table-editor
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#> .

method:RoutineShape
    a sh:NodeShape ;
    # Each leaf subclass is listed explicitly: instances in the description
    # are typed only with leaf classes (never method:Routine directly), and
    # without inference a bare `a method:Routine` query matches nothing.
    sh:targetClass method:Routine ,
                   method:StateVectorNavigation ,
                   method:Alignment ,
                   method:PoweredFlight ,
                   method:Reentry ;

    # Materialize `partOf` from the asserted `contains` edge so the shape
    # works whether or not the reasoned entailments are loaded.
    sh:rule [
        a sh:SPARQLRule ;
        sh:order 0 ;
        sh:construct """
            PREFIX method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#>
            CONSTRUCT { $this method:partOf ?parent }
            WHERE { ?parent method:contains $this }
        """ ;
    ] ;
    sh:property [
        sh:path method:name ;
        sh:name "Name" ;
        sh:datatype xsd:string ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path method:programNumber ;
        sh:name "Program #" ;
        sh:datatype xsd:integer ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path method:missionPhase ;
        sh:name "Mission Phase" ;
        sh:datatype xsd:string ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path method:criticality ;
        sh:name "Criticality" ;
        sh:in ( "Critical" "Non-Critical" ) ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path method:calibrates ;
        sh:name "Calibrates IMU" ;
        sh:class method:IMU ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path method:partOf ;
        sh:name "Container" ;
        sh:class method:Component ;
        sh:maxCount 1 ;
        dash:readOnly true ;
        sh:order 6 ;
    ] ;
    sh:property [
        sh:path method:dependsOn ;
        sh:name "Depends on" ;
        sh:nodeKind sh:IRI ;
        sh:order 7 ;
    ] ;
    sh:property [
        sh:path method:precedes ;
        sh:name "Precedes" ;
        sh:class method:Routine ;
        sh:order 8 ;
    ] ;
    sh:sparql [
        sh:message "This Routine calibrates an IMU it does not also dependsOn. Add the matching dependsOn edge to the same IMU." ;
        sh:select """
            PREFIX method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#>
            SELECT $this
            WHERE {
                $this method:calibrates ?imu .
                FILTER NOT EXISTS { $this method:dependsOn ?imu }
            }
        """ ;
    ] ;
    .
```
