---
ontology: http://www.example.com/apollo-cm-gnc/project/description
title: Routines
---

# Routines

The central question this model exists to answer is which flight programs depend on the IMU, and what breaks if it fails. Every program that touches the IMU, directly or transitively through a program it depends on, is reasoner-classified as `method:IMUDependentRoutine`; the two alignment programs (P51, P52) are additionally classified as `method:IMUCalibrator`. The generated Component Impact Report (`oml codegen -c impact`) lists them per component.

```compose
template: http://www.example.com/apollo-cm-gnc/method/routines
```
