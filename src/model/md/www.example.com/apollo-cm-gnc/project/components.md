---
ontology: http://www.example.com/apollo-cm-gnc/project/description
title: Components
---

# Components

The CM PGNCS decomposes into a small hierarchy: the computer contains its memory, and memory contains the flight programs stored in it. Everything else (the IMU, optical subsystem, DSKY, flight controls) sits alongside the computer rather than inside it, reflecting how these units actually interface as separate boxes joined by cabling, not physical nesting.

```compose
template: http://www.example.com/apollo-cm-gnc/method/components
```
