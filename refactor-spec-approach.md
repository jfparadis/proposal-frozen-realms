# Towards a better factoring of Ecma262

All references to Ecma262 or The EcmaScript Specification, unless stated otherwise, are to [EcmaScript 2020](https://tc39.es/ecma262). This document outlines changes to its organization that should make no observable difference. As such, these would not be semantic changes, and so not need to go through the proposal approval process. However, they are substantial, and would result in a serious needs-consensus PR (pull request).

The purpose of the changes explained here are to prepare the ground for this SES proposal so that it can state its semantic changes more understandably.

## Refactoring the original ModuleRecord abstractions

The [original ModuleRecord abstractions](https://tc39.es/ecma262/#sec-abstract-module-records) mix three concerns
  * The static information that corresponds to the separately linkable units of compilation. In the SourceTextModuleRecord example, this would be the information that could be derived from the source text of one module by itself, with no inter-module analysis. We separate these into ***StaticModuleRecord*** abstractions.
  * A ***ModuleInstance*** has a ModuleStaticRecord and the additional state needed to have a fully linked and initialized stateful module instance. This corresponds most directly to the original ModuleRecord but is renamed to avoid confusion.
  * The ***ModuleInitialization*** bookkeeping needed during the instantiation and initialization of module instances, to take care of cycles, errors, phasing of initialization, etc.

We focus on refactoring the state of these abstractions. From the refactoring of the state, the relocation of the methods should often be obvious and may not be explicitly stated.

### StaticModuleRecord abstractions

The original abstract ModuleRecord has no static slots or methods relevant to the static records. For parallelism, we still define the abstract ***StaticModuleRecord*** as an empty supertype of the other StaticModuleRecord types.

The ***CyclicStaticModuleRecord*** is a StaticModuleRecord with the slot
  * [[RequestedModules]] : List of String

and no methods.

The ***SourceTextStaticModuleRecord*** is a CyclicStaticModuleRecord. It additionally holds the static information from the [original SourceTextModuleRecord](https://tc39.es/ecma262/#sourctextmodule-record). It has the slots
  * [[ECMAScriptCode]] : a ParseNode
  * [[ImportEntries]] : List of ImportEntry records
  * [[LocalExportEntries]] : List of ExportEntry records
  * [[IndirectExportEntries]] : List of ExportEntry records
  * [[StarExportEntries]] : List of ExportEntry records

### ModuleInstance

A ***ModuleInstance*** has the slot
  * [[StaticModuleRecord]] : a StaticModuleRecord as specified above

and the following [original ModuleRecord](https://tc39.es/ecma262/#sec-abstract-module-records) slots
  * [[EvalRecord]] : EvalRecord. This is just a renaming of the [[Realm]] slot from the original ModuleRecord. Below, the original RealmRecord is refactored into the EvalRecord.
  * [[Environment]] : LexicalEnvironment, which is unchanged
  * [[Namespace]] : a ModuleNamespace exotic object, which is unchanged
  * [[Meta]] : Object | undefined. Unchanged from the [import.meta](https://tc39.es/proposal-import-meta/) proposal
  * [[HostDefined]] : Any, which is unchanged

It has no slots from the original CyclicModuleRecord

It has the original SourceTextModuleRecord slot
  * [[Context]] : an ECMAScript [execution context](https://tc39.es/ecma262/#sec-execution-contexts).

### ModuleInitialization

We separate into a distinct ***ModuleInitialization*** object the bookkeeping needed to guide module instantiation, linking, initialization, etc. Thus, once the initialization process completes, this bookkeeping state is no longer present. This helps us reason about post-initialization state separately.

A ***ModuleInitialization*** has the slots
  * [[ModuleInstance]] : the ModuleInstance being initialized

It has the following [original CyclicModuleRecord](https://tc39.es/ecma262/#sec-cyclic-module-records) slots.
  * [[Status]] unchanged
  * [[EvaluationError]] unchanged
  * [[DFSIndex]] unchanged
  * [[DFSAncestorIndex]] unchanged

## Refactoring the original RealmRecord into the EvalRecord

Currently, this is mostly a renaming. EvalRecords will be 1-to-1 with Compartments, and so there will be multiple EvalRecords per realm. EvalRecord isn't a great name, but it'll do.

An ***EvalRecord*** has the following [original RealmRecord](https://tc39.es/ecma262/#sec-code-realms) slots
  * [[Intrinsics]] : a Record of all the intrinsics shared by all compartments in this Realm. Unchanged.
  * [[GlobalObject]] : Object, the global object for this compartment. Unchanged
  * [[GlobalEnv]] : a LexicalEnvironment, for code executing in this compartment. Unchanged.
  * [[TemplateMap]] : A List of {[[Site]], [[Array]]} records. Unchanged.
  * [[HostDefined]] : Any, unchanged.

It has the following hook functions, which are typically provided by the host

  * [[ResolveImportedModule]] : (referrer, specifier) -> resolution, from the [original HostResolveImportedModule](https://tc39.es/ecma262/#sec-hostresolveimportedmodule). This is like the `importer` function from [make-importer](https://github.com/Agoric/make-importer), but synchronous?
  * [[ImportModuleDynamically]] : (referrer, specified) -> Promise, from the [original HostImportModuleDynamically](https://tc39.es/ecma262/#sec-hostimportmoduledynamically). This is like the `importer` function from [make-importer](https://github.com/Agoric/make-importer).
