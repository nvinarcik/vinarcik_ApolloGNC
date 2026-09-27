---
ontology: http://www.example.com/apollo-cm-gnc/project/description
title: Routines
---

# Routines

The central question this model exists to answer is which flight programs depend on the IMU, and what breaks if it fails. Every program listed here that touches the IMU — directly, or transitively through a program it depends on — is reasoner-classified as `method:IMUDependentRoutine`; the two alignment programs (P51, P52) are additionally classified as `method:IMUCalibrator`. That classification isn't asserted below — it's inferred; run `oml reason -o build/reasoned -t` and check the entailments to see it (see the README's "Checking the answer" section).

The table below is generated from `RoutineShape`; the shape is the source of truth. See `src/method/shapes/www.example.com/apollo-cm-gnc/method/routine-shape.ttl` for the canonical definition — keep the two in sync if you edit either one.

```table-editor
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#> .

method:RoutineShape
    a sh:NodeShape ;
    sh:targetClass method:Routine ;
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
