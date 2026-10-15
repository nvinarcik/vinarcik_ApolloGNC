---
ontology: http://www.example.com/apollo-cm-gnc/project/description
title: Components
---

# Components

The CM PGNCS decomposes into a small hierarchy: the computer contains its memory, and memory contains the flight programs stored in it. Everything else — the IMU, optical subsystem, DSKY, flight controls — sits alongside the computer rather than inside it, reflecting how these units actually interface as separate boxes joined by cabling, not physical nesting.

The tree below is generated from `ComponentShape`; the shape (not this page) is the source of truth for what a Component is expected to look like. See `src/method/shapes/www.example.com/apollo-cm-gnc/method/component-shape.ttl` for the canonical definition — keep the two in sync if you edit either one.

```tree-editor
---
columns: { this: { label: "Component" } }
containmentDirection: 'parent'
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#> .

method:ComponentShape
    a sh:NodeShape ;
    # Each leaf subclass is listed explicitly: instances in the description
    # are typed only with leaf classes (never method:Component directly), and
    # without inference a bare `a method:Component` query matches nothing.
    sh:targetClass method:Component ,
                   method:Computer ,
                   method:IMU ,
                   method:OpticalSubsystem ,
                   method:FlightControls ,
                   method:DSKY ,
                   method:Memory ,
                   method:ElectricalBus ;

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
        sh:path method:partOf ;
        sh:name "Container" ;
        sh:class method:Component ;
        sh:maxCount 1 ;
        dash:composite true ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path method:manufacturer ;
        sh:name "Manufacturer" ;
        sh:datatype xsd:string ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path method:massKg ;
        sh:name "Mass (kg)" ;
        sh:datatype xsd:decimal ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path method:powerConsumptionWatts ;
        sh:name "Power (W)" ;
        sh:datatype xsd:decimal ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path method:poweredBy ;
        sh:name "Electrical Bus" ;
        sh:class method:ElectricalBus ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    sh:sparql [
        sh:message "This Component is poweredBy an ElectricalBus but has no powerConsumptionWatts. Add a wattage, or remove the poweredBy link if it isn't known yet." ;
        sh:select """
            PREFIX method: <http://www.example.com/apollo-cm-gnc/method/vocabulary#>
            SELECT $this
            WHERE {
                $this method:poweredBy ?bus .
                FILTER NOT EXISTS { $this method:powerConsumptionWatts ?watts }
            }
        """ ;
    ] ;
    .
```
