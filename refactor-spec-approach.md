# Towards a better factoring of Ecma262

All references to Ecma262 or The EcmaScript Specification, unless stated otherwise, is to [EcmaScript 2020](https://tc39.es/ecma262). This document outlines changes to its organization that should make no observable difference. As such, these would not be semantic changes, and so not need to go through the proposal approval process. However, they are substantial, and would result in a serious needs-consensus PR (pull request).

The purpose of the changes explained here are to prepare the ground for this SES proposal so that it can state its semantic changes more understandably.

## Refactoring the original ModuleRecord abstractions

The [original ModuleRecord abstractions](https://tc39.es/ecma262/#sec-abstract-module-records) mix three concerns
  * The static information that corresponds to the separately linkable units of compilation. In the SourceTextModuleRecord example, this would be the information that could be derived from the source text of one module by itself, with no inter-module analysis. We separate these into ***StaticModuleRecord*** abstractions.
  * The additional ***ModuleContext*** state needed, in addition to a ModuleStaticRecord, to have a fully linked and initialized stateful ***ModuleInstance***.
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

### ModuleContext

A ***ModuleContext*** has the original ModuleRecord slots
  * [[Compartment]] : Compartment exotic object. This is just a renaming of the [[Realm]] slot from the original ModuleRecord. Below, the internal RealmRecord type is refactored into the reified Compartment exotic object.
  * [[Environment]] : LexicalEnvironment, which is unchanged
  * [[Namespace]] : a ModuleNamespace exotic object, which is unchanged
  * [[HostDefined]] : Any, which is unchanged

It has no slots from the original CyclicModuleRecord

It has the original SourceTextModuleRecord slot
  * [[Context]] : an ECMAScript [execution context](https://tc39.es/ecma262/#sec-execution-contexts).

### ModuleInstance

A ***ModuleInstance*** has the slots
  * [[StaticModuleRecord]] : a StaticModuleRecord as specified above
  * [[ModuleContext]] : a ModuleContext as specified above

### ModuleInitialization

We separate into a separate ***ModuleInitialization*** object the bookkeeping needed to guide module instantiation, linking, initialization, etc. Thus, once the initialization process completes, this bookkeeping state is no longer present. This helps us reason about post-initialization state separately.

A ***ModuleInitialization*** has the slots
  * [[ModuleInstance]] : the ModuleInstance being initialized
  * all the fields from the [original CyclicModuleRecord](https://tc39.es/ecma262/#sec-cyclic-module-records).

## Refactoring the original RealmRecord

The [original internal RealmRecord type](https://tc39.es/ecma262/#sec-code-realms) is now the reified Compartment exotic object, an instance of the new `Compartment` abstraction. However, this section by itself does not require that compartment constructors, prototypes, instances, or methods actually be exposed. If they're not, then we still have made no observable changes to EcmaScript. If this is too radical, we can instead make compartment instances be an internal spec object; make the constructor and methods below into internal functions, and then later define the refied `Compartment` abstractions below in terms of these internal objects and functions.

### The Compartment Constructor

  * `new Compartment(endowments?, importer?, options?)`

### Properties of the Compartment constructor

None

### Properties of the Compartment Prototype Object

  * `get global`
  * `evaluate(src: stringable, options?)`
  * `import(specifier)`

### Properties of Compartment Instances

Compartment instances have the [original RealmRecord](https://tc39.es/ecma262/#sec-code-realms) slots
  * [[Intrinsics]] : a Record of all the intrinsics shared by all compartments in this Realm.
  * [[GlobalObject]] : Object, the global object for this compartment.
  * [[GlobalEnv]] : a LexicalEnvironment, for code executing in this compartment.
  * [[TemplateMap]] : unchanged
  * [[HostDefined]] : Any, unchanged.
