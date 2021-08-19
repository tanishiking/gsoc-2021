# GSoC 2021 Final Report: Add synthetics and symbol information for semanticdb in Scala 3
Rikito Taniguchi (rikiriki1238@gmail.com)

## Overview
[SemanticDB](https://scalameta.org/docs/semanticdb/guide.html) is a data model for semantic information such as symbols and types in Scala programs, it is widely used for developing Scala's devtools such as `scalafix` and `Metals`. However, SemanticDB extractor for Scala3 was work in progress and some features in devtools for Scala3 were unavailable because of the lack of semantic information.

In this project, I worked on extracting more information for SemanticDB from Scala3 compiler to improve the developer experience of Scala3. More specifically, I worked on extracting the following data from the compiler.

- Signature information
- Synthetics information
- Some other information on SymbolInforamtion

## Work
### (1) Generate the Scala3 code for SemanticDB
The first thing I did was automating the process of generating Scala3 code from protobuf scheme of SemanticDB.

Previously, the code for Semanticdb data models in Scala3 were copy-and-pasted and adjusted by hand to remove dependencies to `ScalaPB` and `protobuf` library (see the [discussion](https://github.com/scalameta/metals/discussions/2593#discussioncomment-529949)).

This had been plausible way of generating Scala3 code when all the information we extract for SemanticDB was limited. However, once we start extracting more information for SemanticDB, we need much more classes to generate, and hand-crafting all the code and maintain them was unrealistic.

So I proposed some ways to improve the process of code generation, and we decided to generate Scala code using ScalaPB and **automatically** adjust them for Scala3 compiler using `scalafix`. See the [discussion](https://github.com/scalameta/scalameta/issues/2367).

[Here](https://github.com/tanishiking/semanticdb-for-scala3)'s a project to generate and adjust Scala code, and generated code had been [merged into dotty](https://github.com/lampepfl/dotty/pull/12780).

---

### (2) Support Signature information
https://github.com/lampepfl/dotty/pull/12885 (MERGED)

This is the main contribution of this project.
Before the Attributor starts fixpoint iteration it has to identify which abstract attributes it is going to deduce, this is usually called seeding.

This feature is intended to help for adding new attributes and debugging existing ones. Before this feature it was harder to add new attributes because the changes that  a specific attribute would make would be lost in the crowd of other changes that are made by other attributes that are seed by default. 
I Introduced a command line option to specify the kind of abstract attributes that are allowed to be seeded. The command line just takes a comma separated list of attribute names.  
This makes debugging a lot easier.



---

### Phase3: Support Synthetics information of SemanticDB in Scala3

---

### Other contributions
Also, I worked on some 
https://github.com/lampepfl/dotty/issues/12963

https://github.com/lampepfl/dotty/pull/12885


## Link to all the contributions during GSoC

## Future Work
- [Support MatchType and TypeLambda](https://github.com/lampepfl/dotty/issues/12766)
  - Here's a WIP PR to support those types in SemanticDB https://github.com/scalameta/scalameta/pull/2414
- [Extend Signatures to have multiple type parameter clauses](https://github.com/scalameta/scalameta/issues/2450)
- [Support nested annotations in SemanticDB signatures](https://github.com/lampepfl/dotty/issues/13291)
- [Support missing fields in SymbolInformation](https://github.com/lampepfl/dotty/issues/12963)
  - `annotations` and `document` fields are missing at this moment.
