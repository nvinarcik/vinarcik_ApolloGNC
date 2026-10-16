---
name: oml-mcp
description: 'Use when working with OML ontologies, models, validation, reasoning, containment, or SPARQL and the OML MCP tools are available. Prefer this skill before text search or direct file reads/writes for OML questions.'
---

## Load tool schemas first

If OML MCP tool schemas are not yet loaded, batch-load "all" of them in a single ToolSearch call before any other action:

```
ToolSearch("select:oml_workspace,oml_ontologies,oml_search,oml_about,oml_members,oml_paths,oml_sparql,oml_shapes,oml_update,oml_validate")
```

## Planning discipline

Determine if the user question is about retrieving information from ontologies, updating ontologies, or verifying ontologies. Follow the corresponding workflow below.

**Trust results; don't repeat.** Tool results reflect the current state of the workspace. Once a tool has answered something, do not re-run it to "double check" — no repeated searches, `oml_about`, or queries to confirm a result you already have. Re-run a tool only after a change that could have altered its answer (e.g. an `oml_update`). When you have the answer, state it.

**Batch independent calls.** When you need several searches, or `oml_about` on several members, issue them in a single message rather than one round-trip each. Each tool call has real latency, so fewer calls is faster.

## Retrieve context workflow

The standard progression is: **search → about → (members or paths if needed) → sparql**.

**1. Search first.**
Use `oml_search` to fuzzy-search for members matching words from the user prompt. Favor short key words over phrases (phrases usually return nothing). Each result includes the member's `omlType`, so you can tell a class from a property from an instance without a follow-up call. Search once; if it returns nothing, try one alternate key word; if that also fails, ask the user to clarify. Do not search repeatedly for the same intent.

**2. Learn about found members.**
Use `oml_about` on the members found in step 1 to understand how they are defined and how they relate to other things. Pass all relevant IRIs in a single call. This reveals predicates, types, and neighbouring members without writing SPARQL — and often answers the question entirely without needing a SPARQL query at all. Use a root from `oml_workspace` as context for broad coverage.

> **Warning (writes only):** `oml_about` output may show predicates (e.g. inverse relations) that are inferred, not explicitly asserted. When you later intend to *write*, do **not** use a predicate seen here as justification for an `addAssertion` — the shape (`oml_shapes`) is the only authoritative source of assertable predicates. For *reading*, the `oml_about` result is trustworthy as-is.

**3. Dig deeper if needed — discover predicates before writing SPARQL.**
Before you write any SPARQL query, make sure you know the exact type and predicate IRIs you need to use. Use the following tools to discover them.

Tip: Do NOT use `oml_search` any more after using `oml_about`. Instead:

- Use `oml_paths` to discover predicate chains connecting source and target IRIs. This is often the case when the question is about how two known things are related. Use a root from `oml_workspace` as context. Hint: If one of IRIs belongs to an instance, and the other belongs to a class or property IRI, use the instance IRI in the source position.
- Use `oml_members(ontology, kind)` to enumerate all members of an ontology of a given kind.
- Use `oml_ontologies(includeImports=true)` only when the import structure itself is the question.

**4. Query precisely with SPARQL.**
Use `oml_sparql` for filtered, aggregated, or multi-hop results once you know the exact type and/or predicate IRIs from steps 2–3. Use a **root ontology** from `oml_workspace` as context for full import closure coverage — only narrow if you have a specific reason.

Hint: Only if a query you *expected* to return data comes back empty, run one diagnostic probe before changing query logic: `SELECT ?s ?p ?o WHERE { ?s <pred> ?o } LIMIT 5`. Empty probe = wrong predicate IRI. Non-empty probe = query logic is the problem. An empty result that is itself the answer needs no probe.

## Update ontologies workflow

1. Retrieve context first:
   - Identify relevant classes, properties, and existing instances.
   - Identify candidate target ontologies for each intended change.
   - For every instance you will create, find existing instances of the same kind (e.g. other routines) and note their types. New instances follow the same typing pattern as their siblings.
   - For every value you will change, read the member's annotations (e.g. `dc:description`) and note any text that states or explains the old value.

**Scope: do only what was asked.**
- Assert only the facts the user requested, plus the facts the shape requires (`sh:minCount >= 1`). If other facts seem useful (e.g. an extra `precedes` link), list them as suggestions in your answer instead of asserting them.
- A request to add or change data is a description-only change. Do not add or change vocabulary concepts, properties, or shapes to make it fit. If no existing class fits, stop and ask the user.

2. Use `oml_shapes` on the class IRI of every instance type you intend to create or modify. Read the result carefully — it is your specification for `oml_update`. Extract:
   - `sh:targetClass` — confirms the class IRI to assert as the instance type. Assert the **most specific** (leaf) class that fits, matching how sibling instances are typed. Do not assert an abstract or intermediate class (one that has subclasses) when its siblings use a leaf class; superclasses are inferred by the reasoner.
   - `sh:property / sh:path` — the **exact predicate IRI** to use in each `addAssertion` or `addAnnotation` operation. Do not guess or invent predicate IRIs; use only what the shape lists.
   - `sh:class` — the class of any cross-referenced instance (the linked target must be of this class).

   Before adding any operations, double check that its predicate IRI matches a `sh:path` from the shape. If a predicate IRI has no matching `sh:path`, do not assert it.

3. Build **all** `oml_update` operations in a single call using the predicate IRIs from step 2:
   - `oml_update` is atomic: all operations succeed or all fail — plan the full set before calling.
   - Order: `createInstance` → `addAssertion`/`addAnnotation` for required facts → cross-links to other instances.
   - Whenever you add a `create*` operation, also add `addAssertion` to asser the instance's type.
   - When you change a value, include in the same call an update to any annotation on that member that states the old value (found in step 1), so the description never contradicts the data.
   - Apply directly for routine changes built from a shape (single-instance creates, simple assertions) — `oml_update` is atomic and reports lint problems on apply, so a mistake is caught and fixable without a separate preview. Reserve `preview: true` (then a second call to apply) for removals, bulk edits, or unfamiliar patterns where seeing the generated text first is worth the extra round-trip.
   - When removing data, remove dependent links before deleting members.
   - When using `createOntology`, specify the `targetFolder` as the relative workspace path to an `oml` folder that should nest the new ontology.

4. Avoid these pitfalls:
   - Do not issue multiple sequential `oml_update` calls to fix up a previous one — plan from the shape and do it in one call.
   - Do not create instances without asserting their type and all required (`sh:minCount >= 1`) facts in the same call.
   - Do not edit ontology files directly. Always use `oml_update`.
   - Never call `addImport` on a description ontology; only on a description bundle.

5. If `oml_update` reports lint problems, call a new `oml_update` to address the reported problems.

6. If lint free, call `oml_validate` to validate the changes.

7. Review the textual footprint of the change. `oml_update` may re-serialize the whole file (re-indenting, merging repeated assertions into comma lists, writing typed literals such as `"42.0"^^xsd:decimal`). Compare `git diff --stat` with `git diff -w --stat`, and read `git diff -w`. In your answer to the user:
   - list the substantive changes (what was added or changed in the model), and
   - state separately whether the file was also reformatted, and how many lines that touched, so the user can decide whether to keep or normalize it.
   Do not hide reformatting inside a summary of "the change".
