# Building a Morphir Backend

Backends in Morphir are essential because they allow Morphir models to immediately become useful in a target environment.
The Morphir tooling already ships with a backends for Scala2, TypeScript, JsonSchema and Spark as of the time of writing this document.

This document talks about the necessary considerations to make and strategies that can be applied when building a backend for Morphir.

> As of the time of writing this document, the current released version of morphir-elm is 2.74.0

___

##### Table of Contents  
- [Background](#background)
- [Key Considerations](#key-considerations)
	- [Documenting](#documenting)
	- [Scope of Support](#scope-of-support)
		- [Morphir Feature Mapping](#morphir-feature-mapping)
		- [Morphir SDK](#morphir-sdk)
	- [Generating Tests](#generating-tests)
- [Building A Backend](#building-a-backend)

<a name="background"></a>
## Background

Morphir is an executable specification language that is used to model business logic with all of its business concepts.
For this reason, Morphir does not support a number of technology related concepts like multi-threading,  sessions, etc. because these are only required in terms of a specific technology but are not required for the business domain. As a consequence, Morphir exists in isolation and cannot integrate with other technologies out of the box.

Backends in Morphir are essential because they allow Morphir models to immediately become useful in a target environment. a Morphir backend takes a Morphir model, stored in json format, and converts that into code that is executable or valid for the target environment.

<a name="key-considerations"></a>
## Key Considerations

There are a number of things to consider when building a backend for Morphir. These things may or may not be relevant to your backend, but it is important to think about them before deciding whether or not they are relevant or even supported in your target environment.

##### Considerations:
- [Documenting](#documenting)
- [Scope of Support](#scope-of-support)
	- [Morphir Feature Mapping](#morphir-feature-mapping)
	- [Morphir SDK](#morphir-sdk)
- [Generating Tests](#generating-tests)

<a name="documenting"></a>
### Documenting

Documenting should be your first consideration as it helps you understand the possibilities and limitations before you write a single line of code. Consider the documentation an explanation of the considerations listed above.

As part of the other consideration, documentation will be mentioned frequently.

<a name="scope-of-support"></a>
### Scope of Support

The first thing to understand is that Morphir is not a general purpose language and as a consequence of this, mapping Morphir features to a general purpose language is much more feasible as opposed to mapping it to language that isn't. 

For example, `morphir-elm@2.74.0` ships with backends for Scala and JsonSchema.
When generating Scala, you can be sure that 100% of any Morphir model can be generated in Scala.
We can't have that same level of confidence when generating JsonSchema for any Morphir model because certain types are not supported in JsonSchema, ex: function types. 

It is important to start off your documentation with a mapping of all the Morphir features, showing what is supported and how, and also showing what isn't or cannot be supported.

<a name="morphir-feature-mapping"></a>
### Morphir Language Feature Mapping

All Morphir language features can be found _[here](insert link here)_ and a key consideration would be creating a table that lists these and how they are supported in your target language.

*As a guideline, your table can be formatted like:*

|          | Morphir Feature | Target Feature | Comment                          |
|----------|-----------------|----------------|----------------------------------|
|Types     |                 |                |                                  |
|          |String           |string          |                                  |
|          |Decimal          |float           |no direct support, mapped to float|
|          |Date             |string          |not supported                     |


This table provides not only users of your backend with an understand of it's capabilities, but also provides you a clear pathway on what decision to make for certain features.

<a name="morphir-sdk"></a>
### Morphir SDK

The Morphir SDK provides support native functionality found in many languages. It also provides additional types that users can use when creating a model.

The SDK should also be put into consideration when building a backend, an important question to answer is "can I support all these functionalities in my backend without manually creating an SDK?", if no, you will need to create manually write out an SDK that can be included and/or referenced in the generated code/generated file.
For example, the Scala backend that ships with Morphir provides a manually written SDK whereas the JsonSchema backend maps all the relevant SDK types to types that are already present in JsonSchema.

<a name="generating-tests"></a>
### Generating Tests

For every Morphir model, Morphir tests can be added via the `morphir-elm develop` UI, so another key consideration is generating test cases based on the `morphir-tests.json`. 

As part of code generation, you can generate test cases for any testing framework. Depending on the target language and what is possible in it, you can also just generate generic test cases that don't depend on any testing framework, but contain inputs and outputs that can be utilized later in other testing framework  

The current Scala backend already has this [functionality](https://github.com/nwokafor-choongsaeng/morphir/blob/main/docs/users-guide/morphir-scala-gen.md) included and it goes to ensure the generated code behaves the same as the model did.

<a name="building-a-backend"></a>
## Building A Backend
> *Having a documentation would be great in making decisions  in this section*

The most convenient language for building a backend would be ELM as Morphir's language frontend for ELM was written in ELM. This means that there are types and utility functions that would make your life easier if you're already working in ELM. Support for TypeScript is currently developing but is not as good as ELM as of the time of writing this.

At a high level, all backends have a signature of 
`myBackend : Morphir.Distribution -> FileMap`

`Distribution` here is the In memory representation of a Morphir Model and
`FileMap` is a collection of files by their file paths.
`myBackend` is the entry point of your backend.

The concepts of the target language can be represented as an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree), and only the parts of the language required to satisfy the feature mappings previously documented needs to be captured as part of the AST.

What follows after defining an AST is to implement functions that move from a Morphir feature to it's counterpart in the AST.
At a high level, the function can be represented as: `conceptMapper : Morphir.Feature -> Target.AST `
where `Morphir.Feature` represents the features in Morphir and `Target.AST` represents the types that were defined as part of your AST.

For references, you can look at the [Scala.AST](https://github.com/finos/morphir-elm/blob/main/src/Morphir/Scala/AST.elm) and the [Scala.Backend](https://en.wikipedia.org/wiki/Abstract_syntax_tree) and see real implementations of the concepts discussed above.
