# Vinarcik Apollo GNC

An OML ontology of the Apollo Command Module's Primary Guidance, Navigation and Control System (PGNCS): the flight computer and its software, the optical subsystem (scanning telescope and sextant), the inertial measurement unit (gyroscopes and accelerometers), and the interfaces that translate their data into flight corrections and maneuvers. Scoped to the Command Module only — the Lunar Module is explicitly outside the boundary.

The model's central question: **which routines depend on the IMU, and what degrades if it fails?** The vocabulary defines two derived concepts, `IMUDependentRoutine` and `IMUCalibrator`, that let the reasoner classify the answer directly rather than requiring it to be read off the graph by hand — see "Checking the answer" below.

This is an [OML](https://www.modelware.io/) project, split into a method (vocabulary) and a model (description), matching the layout used in the `sierra-method` coursework repo.

## Repo layout

- `src/method/oml/www.example.com/apollo-cm-gnc/method/vocabulary.oml` — the vocabulary: concepts, relations, scalar properties, a defined-concept pair (`IMUDependentRoutine`, `IMUCalibrator`), and the `TransitiveDependency` rule for the CM PGNCS domain.
- `src/method/oml/www.example.com/apollo-cm-gnc/method/bundle.oml` — the vocabulary bundle. Bundling the vocabulary is what makes the reasoner treat sibling concepts without a common subtype as disjoint by closed-world closure; without it, disjointness axioms never fire.
- `src/model/oml/www.example.com/apollo-cm-gnc/project/description.oml` — the description: instances of the actual CM PGNCS components, interfaces, and routines (8 components, 14 flight programs).
- `src/model/oml/www.example.com/apollo-cm-gnc/project/bundle.oml` — the description bundle, closing the description over the method bundle.
- `src/method/shapes/www.example.com/apollo-cm-gnc/method/` — SHACL shapes for components and routines.
- `src/method/md/www.example.com/apollo-cm-gnc/method/` — editor templates (`components` tree editor, `routines` table editor), invoked from the thin pages in `src/model/md` with a `compose` block.
- `src/method/oml/www.example.com/apollo-cm-gnc/codegen/impact.oml` — the Component Impact Report generator (spec and template).
- `src/model/oml/www.example.com/apollo-cm-gnc/project/codegen.oml` — the generator config for this model.

## Component Impact Report

`oml codegen -c impact` writes `build/docs/Apollo-CM-GNC-impact.md`. For every component it lists the flight programs that depend on it, directly or through another program or component, with each program's kind, mission phase and criticality, plus the path the dependency takes. It also lists the components that depend on it and, for an electrical bus, the load on it. The summary table at the top answers the model's central question for every component at once.

## Using the method as a package

Everything under `src/method` is packaged as `@vinarcik/apollo-gnc-method`: the vocabulary, the shapes, the editor templates and the generator.

```bash
oml pack                                  # writes build/dist/*.tgz
oml publish -r http://localhost:4873      # local Verdaccio registry
```

A consumer declares the package under `dependencies` in its `.oml/settings.yml`, runs `oml install -r http://localhost:4873`, uses `<http://www.example.com/apollo-cm-gnc/method/bundle#>` from its description bundle, and adds a `codegen:Config` that points at `impact:ImpactReportSpec` with `path`, `file` and `title` bindings.

## Getting started

1. Open this folder in VS Code with the **OML Code** extension installed, and run all `oml` commands from VS Code's integrated terminal (a shell started elsewhere won't find the OML server).
2. Run `oml lint` to check the model for syntax and reference errors.
3. Run `oml reason` to check consistency (add `-e` to explain any inconsistency).
4. Edit the vocabulary and description files above to build out the model.

## Checking the answer

Asserted facts alone won't show which routines the model considers IMU-dependent — that's an inferred classification, not something written directly into `description.oml`. To see it, materialize the reasoner's entailments:

```
oml reason -o build/reasoned -t
```

This writes both the asserted graph (`description.ttl`) and a separate delta of newly inferred facts (`description__entailments.ttl`) per ontology under `build/reasoned/`. In the entailments file, every one of the 14 flight-program instances carries an inferred `rdf:type` of `method:IMUDependentRoutine`, and `P51`/`P52` additionally carry `method:IMUCalibrator`. `ApolloIMU` itself carries an inferred `method:dependedOnBy` edge to all 14 programs, giving a one-hop answer to "what depends on the IMU" straight from the reasoned graph. The `TransitiveDependency` rule is what extends IMU-dependency from the five navigation programs, which assert it directly, out to the maneuver and reentry programs, which only assert a dependency on a navigation program.

## Commands

- `oml lint` — lint OML files and report any problems
- `oml reason` — run the reasoner (`-e` explains inconsistencies, `-u false` disables the unique names assumption)
- `oml reason -o <dir> -t` — run the reasoner and materialize entailments (asserted + inferred deltas) to `<dir>`
- `oml start` / `oml list` — start or list the background OML server for this workspace
- `oml export -o build/owl` — export the asserted OWL graph only (no inferred facts)
