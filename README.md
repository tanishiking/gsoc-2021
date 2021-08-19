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

[Signature](https://scalameta.org/docs/semanticdb/specification.html#signature) in SemanticDB is a data structure that represents definition signatures (type information) for class, method, type, and local value. This information is consumed by, for example, metals to [show inferred types as decoration](https://scalameta.org/metals/blog/2021/02/24/tungsten/#type-decorations-for-definitions), and `scalafix` rules that utilizes type information. Previously, Scala3 doens't extract this information and Metals unable to show the inferred type for Scala3 programs.

In this project I extended Scala3 to extract signature information. While most of the types in Scala3 are covered, some new Scala3 types such as [TypeLambdas](https://docs.scala-lang.org/scala3/reference/new-types/type-lambdas-spec.html) and [MatchType](https://docs.scala-lang.org/scala3/reference/new-types/match-types.html) are not yet supported, because it requires extending the SemanticDB schema. This is a future work.

---

### (3) Support Synthetic
[Synthetic](https://scalameta.org/docs/semanticdb/specification.html#synthetic) is a information that represents trees synthesized by the compiler. For example, Metals uses this information to [show implicit parameters](https://scalameta.org/metals/docs/#implicit-decorations), show inferred type applications, and find references.

First, I [summarized](https://github.com/lampepfl/dotty/issues/13135) what kind of synthetic information we should extract, because Scala3 has different syntax and semantics than Scala2. During the coding period, I submit PRs to support several of synthetics but they are not yet merged.

- https://github.com/lampepfl/dotty/pull/13288
- https://github.com/tanishiking/dotty/pull/5


### (4) Support some missing fields of `SymbolInformation`
https://github.com/lampepfl/dotty/issues/12963

While working on this project, I realized some useful information is missing from `SymbolInformation` in Scala3.

- Access information: [PR](https://github.com/lampepfl/dotty/pull/12964)
- Overridden Symbols: [PR](https://github.com/lampepfl/dotty/pull/13295)

### (5) Other contributions
- 
  - https://github.com/lampepfl/dotty/issues/13270
- Related to `overriddenSymbols`, I refactored Metals using overiddenSybol shttps://github.com/scalameta/metals/pull/3045
- Support Scala3 modifiers
https://github.com/scalameta/scalameta/pull/2439

## Links to all the contributions during GSoC
- [lampepfl/dotty](https://github.com/lampepfl/dotty/issues?q=author%3Atanishiking+created%3A2021-06-01..2021-08-17)
- [scalameta/scalameta](https://github.com/scalameta/scalameta/issues?q=author%3Atanishiking+created%3A2021-06-01..2021-08-17)
- [scalameta/metals](https://github.com/scalameta/metals/issues?q=author%3Atanishiking+created%3A2021-06-01..2021-08-17)
- New repositories
  - [tanishiking/semanticdb-for-scala3](https://github.com/tanishiking/semanticdb-for-scala3)
  - [tanishiking/scalafix-unused](https://github.com/tanishiking/scalafix-unused)

## Future Work
- [Support MatchType and TypeLambda](https://github.com/lampepfl/dotty/issues/12766)
  - Here's a WIP PR to support those types in SemanticDB https://github.com/scalameta/scalameta/pull/2414
- [Extend Signatures to have multiple type parameter clauses](https://github.com/scalameta/scalameta/issues/2450)
- [Support nested annotations in SemanticDB signatures](https://github.com/lampepfl/dotty/issues/13291)
- [Support missing fields in SymbolInformation](https://github.com/lampepfl/dotty/issues/12963)
  - `annotations` and `document` fields are missing at this moment.
