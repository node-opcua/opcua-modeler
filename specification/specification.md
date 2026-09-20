---
title: OPC UA YAML DSL
subtitle: A human-friendly, declarative authoring format for OPC UA NodeSet2 information models
subtitleTop: Sterfive Specification
template: spec
brand: Sterfive OPC UA YAML DSL
brandSub: v0.1 (public draft) · 2026-09-19
version: 0.1 (public draft)
status: Working Draft
date: 2026-09-19
editor: Etienne Rossignon, Sterfive SAS
lang: en
copyright: Copyright © 2026 Sterfive SAS. Licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).
---

## Abstract {.unnumbered}

This specification defines the **Sterfive OPC UA YAML DSL**, a declarative,
human-readable authoring format that compiles to conformant OPC UA NodeSet2
XML information models. The DSL is intended as a generic, vendor-neutral
specification and is open to multiple, independent implementations. It
expresses the full OPC UA address-space vocabulary — ObjectTypes,
VariableTypes, DataTypes (basic, enumeration, structure, union, option set),
ReferenceTypes, InterfaceTypes, EventTypes, FiniteStateMachine types,
instances, methods, add-ins and additional references — while shielding the
author from the syntactic burden of NodeSet2 XML and from explicit numeric
NodeId assignment.

The semantics of the produced address space are anchored in the OPC UA
standard, and in particular in [OPC 10000-3] *Address Space Model* and
[OPC 10000-5] *Information Model*. Where this specification constrains a
feature more tightly than the OPC UA standard (for example by requiring a
syntactic convention), the constraint is called out explicitly.

The companion JSON Schema artefact published as
`schemas/nodeset2.schema.json` is the machine-readable representation of the
syntactic rules in this specification and is normative for syntactic
validation. This document is normative for the semantics, the cross-property
invariants and the equivalence between a DSL document and the OPC UA address
space it produces.

## Status of This Document {.unnumbered}

This is a Sterfive Working Draft published for public review. The
specification may be updated, replaced or made obsolete by Sterfive at any
time. Publication as a Sterfive Working Draft does not imply endorsement by,
or affiliation with, the OPC Foundation.

References to OPC UA terminology and node classes are provided for
interoperability. OPC, OPC UA and NodeSet2 are trademarks of the OPC
Foundation; their use in this document is purely descriptive and refers to
the format the DSL targets.

## Changes in This Revision {.unnumbered}

Revision 0.1 (2026-09-19) brings the specification in line with what the
schema and the reference processor accept, found by round-tripping the
OPC Foundation companion NodeSets:

- [Document Structure](#document-structure) restored: every root property
  (`lint`, `profile`, `objects`, `namespaceMetadatas`,
  `outputBinaryTypeDictionary`…), which also brings the section numbers
  cited in this document back in line;
- [Variable attributes](#variable-attributes): `userAccessLevel`,
  `minimumSamplingInterval`, `historizing`, the DataItem Properties, and
  `exposesItsArray` (offered by the schema, rejected by the processor until
  now);
- Structures: `allowSubTypes` on a field; the companion VariableType is
  built with `HasStructuredComponent`, and `variableType` refines it;
- ReferenceTypes: `symmetric`;
- Additional references: `source` is required in both forms (the draft
  said it was implicit per node), and a per-node entry may declare its
  target;
- anchors: `$anchor`; a NamespaceMetadata Object is named by its URI;
- Add-In entries: own members, `description`, nested `addIns`;
- `organizes` may hold Variables and Properties; `accessLevel` is not a
  VariableType attribute; state machines may declare `methods`;
- `imports`: a `sha256` mismatch is an error; initializer paths reach a
  Property with `.`; `optionals` segments may carry an alias;
- an `organizes` member with a modelling rule is an InstanceDeclaration;
- the self-contained profile is no longer future work.

## Introduction

OPC UA address spaces are conventionally distributed as
[NodeSet2](https://reference.opcfoundation.org/Core/Part6/) XML documents.
NodeSet2 XML is faithful to the OPC UA Address Space Model defined in
[OPC 10000-3] but is verbose, tightly coupled to numeric NodeIds, and
laborious to author or review. The Sterfive OPC UA YAML DSL provides a
compact, declarative representation of the same model:

- nodes are addressed by **browse name** and **browse path** rather than by
  NodeId, in line with the symbolic-name convention of [OPC 10000-5] §4.1;
- numeric NodeIds are auto-assigned and persisted in a *symbol-table* CSV
  that survives regeneration;
- composite features such as DataType structures, EventTypes, Add-Ins and
  FiniteStateMachine types are written as concise YAML mappings;
- the document targets OPC UA 1.05 [OPC 10000-3] [OPC 10000-5], with an
  opt-in toggle to also emit the legacy 1.03 BinaryTypeDictionary entries
  for compatibility with older clients.

A DSL document is consumed by a *Conformant Processor* which produces a
NodeSet2 XML document. Implementations **MAY** additionally produce
TypeScript declaration files, Markdown or PDF documentation; those auxiliary
artefacts are out of scope of this specification.

### Positioning: an authoring format, not an interchange format

The DSL is a *source* format, in the tradition of the OPC Foundation's own
ModelDesign.xml: a compact description that a compiler turns into a NodeSet2
document, with the assigned NodeIds kept in a companion CSV so that
regeneration is stable. NodeSet2 XML (and the JSON forms defined by
[OPC 10000-6]) remains the only interchange format: it is what a Conformant
Processor emits, what other tools consume, and what a working group commits
and reviews. A DSL document is never required for interoperability, and a
tool that has no knowledge of this specification loses nothing. The DSL
exists to make the model easier to write, review in version control, and
generate with assistance; it does not compete with NodeSet2 and is not
proposed as a replacement for it.

> [!NOTE]
> **Self-contained profile.** A document and its symbol table are two files,
> which is the right shape for authoring (the CSV is generated, the YAML is
> hand-written) but not for exchange. The companion *Self-Contained Profile*
> specification (draft, `profile: self-contained`) packs a model, its NodeId
> symbol table and what it needs from its dependencies into one YAML stream,
> for the cases where a single file is preferable.

```yaml title="Example 1 — minimal document"
namespaceUri: http://acme.com/UA/TemperatureSensor
version: 1.0.0
publicationDate: 2026-01-01T00:00:00.000Z
namespaces:
  - di
objectTypes:
  - browseName: TemperatureSensorType
    subtypeOf: di:DeviceType
    components:
      - browseName: di:ParameterSet
        components:
          - browseName: Temperature
            typeDefinition: ua:AnalogItemType
            dataType: ua:Double
            engineeringUnits: "degree Celsius"
instances:
  - browseName: Sensor01
    typeDefinition: TemperatureSensorType
    organizedBy: /ua:Objects/di:DeviceSet
```

## Conformance

This document uses normative keywords as defined by RFC 2119 / RFC 8174.
The terms **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY** and **OPTIONAL**, when
typeset in this manner, denote the corresponding requirement levels.

This specification defines two conformance classes:

Conformant Document
:   A YAML 1.2 document whose root mapping satisfies every **MUST**
    requirement in sections 4 through 18.

Conformant Processor
:   An implementation that, given a Conformant Document, produces a
    NodeSet2 XML document [NodeSet2-equivalent](#equivalence-with-nodeset2-xml) to
    the address space described by the document, and rejects with a
    diagnostic any document that violates a **MUST** requirement of this
    specification.

> [!NORMATIVE]
> A Conformant Processor **MUST** implement sections 4–15 in full and
> **MUST** reject documents that contain unknown root properties or unknown
> sub-properties flagged by this specification as having
> `additionalProperties: false`. Sections 16–18 ([Events](#events),
> [State Machines](#state-machines), [Initializers](#initializers)) describe
> optional features. A processor that does not implement an optional
> feature **MUST** reject documents that use it rather than silently ignore
> the feature.

## Terminology

document
:   A YAML 1.2 file whose root is a mapping conforming to this
    specification.

own namespace
:   The namespace whose URI is given by the `namespaceUri` property at the
    document root. New nodes defined by the document are created in this
    namespace.

imported namespace
:   A namespace declared through `namespaces` or `imports`. Its nodes are
    visible to the document but not redefined by it.

browse name
:   A qualified name. Written as a bare PascalCase identifier when in the
    own namespace, or as `alias:Name` otherwise.

placeholder browse name
:   A browse name surrounded by `<>`, e.g. `<ImageId>`. It denotes a member
    created by an `OptionalPlaceholder` or `MandatoryPlaceholder` modelling
    rule.

browse path
:   A solidus-separated sequence of browse names. Browse paths starting
    with `/` are *absolute* and rooted at `ua:RootFolder`; paths starting
    with `$Identifier/` are *anchored* at a node defined in the same
    document; otherwise the path is *relative* to the closest enclosing
    context.

connected object
:   An instance linked to its parent through exactly one of the connection
    properties listed in [§14.1](#connected-object-rule).

instance
:   An OPC UA Object or Variable that lives in the address space at run
    time, as opposed to a Type which is a template.

add-in
:   A reusable Object aggregate, instantiated by reference to its
    `typeDefinition` rather than by writing out its members, with browse-name
    resolution rules described in [§12](#add-ins).

symbol table
:   A CSV file (conventionally named `<model>.symbols.csv`) that stores the
    auto-assigned NodeId of every node in the document. Persistence of
    NodeIds across regeneration is achieved by consulting and updating this
    file.

## Document Structure

The root of a document is a mapping. `namespaceUri` is **REQUIRED**; every
other property is optional. A processor **MUST** reject a root property not
listed here.

| Property                     | Type    | Section / meaning                                                                                                                                     |
| ---------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `namespaceUri`               | string  | The URI of the own namespace.                                                                                                                         |
| `version`                    | string  | The model version, written to the NodeSet2 `Model` element.                                                                                           |
| `publicationDate`            | date    | The model publication date.                                                                                                                           |
| `profile`                    | string  | `self-contained`: the file is a YAML stream carrying the model, its symbol table and its dependencies (see the self-contained profile).               |
| `namespaces`, `imports`      | array   | The namespaces the document builds on ([Namespace Imports](#namespace-imports)).                                                                      |
| `documentation`              | mapping | Rendered documentation only (`title`, `author`, `header`, `options.diagrams`); no effect on the address space.                                        |
| `lint`                       | mapping | Per-document diagnostic policy: `silence` drops the listed codes or code prefixes, `error` escalates warnings to errors ([Diagnostics](#diagnostics)). |
| `outputBinaryTypeDictionary` | boolean | Also emit the OPC UA 1.03 binary and XML TypeDictionary nodes. Default `false`.                                                                       |
| `dataTypes`                  | array   | [DataTypes](#datatypes).                                                                                                                              |
| `referenceTypes`             | array   | [ReferenceTypes](#referencetypes).                                                                                                                    |
| `objectTypes`, `variableTypes` | array | [ObjectTypes and VariableTypes](#objecttypes-and-variabletypes).                                                                                      |
| `interfaceTypes`             | array   | [InterfaceTypes](#interfacetypes).                                                                                                                    |
| `eventTypes`                 | array   | [Events](#events).                                                                                                                                    |
| `stateMachines`              | array   | [State Machines](#state-machines).                                                                                                                    |
| `instances`                  | array   | [Instances](#instances).                                                                                                                              |
| `objects`                    | array   | Low-level Objects, created as written: the members of their `typeDefinition` are **not** instantiated. Use `instances` for an instance of a type.     |
| `namespaceMetadatas`         | array   | The NamespaceMetadata Object of the own namespace; `typeDefinition` defaults to `ua:NamespaceMetadataType` and the parent to `/ua:Objects/ua:Server/ua:Namespaces`. |
| `references`                 | array   | [Additional References](#additional-references).                                                                                                      |
| `initializers`               | array   | [Initializers](#initializers).                                                                                                                        |

The collections are processed in dependency order, not document order
([Processing Model](#processing-model)): a type may reference a type declared
later in the same document.

## Lexical Conventions

### Browse names and placeholders

A bare browse name **MUST** match the regular expression
`([A-Za-z0-9]+:)?(([A-Z][a-zA-Z0-9]*)|(<[A-Z][a-zA-Z0-9°#]*>))`. The
optional `alias:` prefix selects an imported namespace; without it the
browse name belongs to the own namespace. PascalCase **SHOULD** be used for
the local part. English **SHOULD** be preferred over other languages when
authoring browse names.

The browse name of a `namespaceMetadatas` entry is the namespace URI itself
(`http://…`), as [OPC 10000-5] names a NamespaceMetadata Object; a URI
**SHOULD NOT** be used as a browse name anywhere else.

A placeholder browse name is enclosed in `<>`, e.g. `<ItemId>`. It **MUST**
appear only on a member that declares `modellingRule: OptionalPlaceholder`
or `modellingRule: MandatoryPlaceholder`.

The alias `ua` is reserved for the OPC UA standard namespace
`http://opcfoundation.org/UA/`. A document **MUST NOT** redefine `ua`.
Aliases are case-sensitive.

### Browse paths and anchors

A browse path is a string of solidus-separated browse names matching
`(/(alias:)?Name)+` with an optional anchor prefix.

| Form                  | Meaning                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `/ua:Objects/...`     | Absolute path rooted at `ua:RootFolder`.                                                                                             |
| `$Robot1/Axes`        | Anchored path: `$Robot1` resolves to the instance of this document whose anchor is `Robot1` (see below).                            |
| `Robot1/Axes`         | Relative path; the resolver searches first the current parent, then sibling instances of the document by browse name.                |

{.props}

An instance's anchor is its `$anchor` property when given (an identifier,
`[A-Za-z_][A-Za-z0-9_]*`), otherwise the local part of its browse name; a
processor **MUST** append a numeric suffix to make duplicate anchors unique
and **SHOULD** report the duplicate. Authors write `$anchor` to keep a
path stable when a browse name repeats.

A Conformant Processor **MUST** reject a document if any browse path cannot
be unambiguously resolved.

### Namespace aliases

The following aliases are reserved by this specification and resolve to
companion-spec namespaces published by the OPC Foundation. A document
**MUST NOT** use any of them as a custom `imports[].alias`.

`adi`, `amb`, `autoId`, `cnc`, `commercialKitchenEquipment`, `di`, `gds`,
`glass`, `ia`, `ijtBase`, `iolink`, `iolinkIODD`, `irdi`,
`isa95JobControl`, `machinery`, `machineryJobs`, `machineryProcessValues`,
`machineryResult`, `machineTool`, `machineVision`, `metalForming`,
`packML`, `padim`, `robotics`, `standard`, `surfaceTechnologyGeneralTypes`,
`surfaceTechnologyOctMss`, `tightening`, `woodWorking`.

### NodeId assignment

Numeric NodeIds **SHOULD NOT** be authored by hand. A Conformant Processor
**MUST** assign a fresh numeric NodeId to every node that lacks an explicit
`nodeId`, persisting the assignment in the symbol table so it remains
stable across regenerations.

When required (for example to interoperate with an external system that
hard-codes a NodeId), a node **MAY** declare an explicit `nodeId` string
matching one of:

- `s=<string>` — string NodeId.
- `g=<uuid>` — GUID NodeId.
- `b=<base64>` — opaque NodeId.

The namespace prefix `ns=<index>;` **MUST NOT** appear: the namespace is
fixed by the own namespace. When a parent declares a string `nodeId`,
descendants whose NodeId is not explicitly stated **SHOULD** derive a
string NodeId from the parent (e.g. `s=MyMachine-Child1`).

### Localized text

A property typed as LocalizedText accepts either:

- a bare string (locale defaults to none), or
- a mapping `{ locale: <string|null>, text: <string> }`.

## Namespace Imports

The `namespaces` property declares an *ordered* list of well-known
companion-spec aliases (see [§5.3](#namespace-aliases)). Order matters: a
namespace **MUST** appear after every namespace it depends on. For example,
because `adi` depends on `di`, the order `[di, adi]` is required.

The `imports` property declares additional, non-well-known imports. Each
entry **MUST** be a mapping containing `alias` plus **at least one** of `uri`
or `filename`:

| Key               | Type       | Card. | Notes                                                                                                                                   |
| ----------------- | ---------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `alias`           | string     | 1     | Short name used to qualify browse names; case-sensitive; **MUST NOT** collide with a reserved alias.                                     |
| `uri`             | URI string | 0..1  | Absolute namespace URI — the *identity* of the import. Resolution matches it against each candidate's `<Model ModelUri>`.                |
| `filename`        | string     | 0..1  | Path to the NodeSet2 XML, resolved relative to the DSL document. A *hint*: if it is missing or stale and `uri` is given, `uri` is used.  |
| `publicationDate` | date       | 0..1  | Require a nodeset published on or after this date (OPC UA's `PublicationDate >= required` rule). Not a version range.                    |
| `sha256`          | string     | 0..1  | Integrity pin: the sha256 of the exact file expected. A mismatch **MUST** be rejected: a pin that only warned would not pin anything.    |

An import that names only a `uri` is resolved by scanning, in order, the
document's own folder and a `./deps/` folder beside it, reading each candidate's
`<Model ModelUri>` header. Filenames inside `./deps/` are therefore irrelevant —
what identifies a nodeset is what it declares itself to be, which means the same
thing on every machine, whereas a path does not.

When both are given, `filename` is tried first and the loaded `ModelUri` is
**verified** against the declared `uri`; a mismatch is reported as a warning and
the file is still loaded, because `uri` was informative-only in earlier versions
and incorrect values exist in existing models.

A `filename` that is not a NodeSet2 XML is rejected with a diagnostic naming the
file. In particular, a YAML model cannot be imported directly — compile it to a
NodeSet2 first (`opcua-modeler generate`) and import the result.

{.props}

```yaml title="Example 2 — mixed imports"
namespaces:
  - di
  - ia
  - machinery

imports:
  - alias: ts
    uri: http://acme.com/UA/TemperatureSensor/
    filename: ../temperature_sensor.NodeSet2.xml
```

## DataTypes

DataTypes describe the structure of Variant values, as defined in
[OPC 10000-3] §5.8. A DataType definition **MUST** declare `browseName`. It
**MAY** declare any of `subtypeOf`, `isAbstract`, `description`, `nodeId`,
plus the kind-specific properties below. The kind of a DataType is inferred
from the keys present, as defined in §7.1–§7.5.

### Basic DataType

A simple alias of an existing DataType. The mapping **MUST** declare
`subtypeOf` and **MUST NOT** declare `fields`, `enum`, `isUnion` or
`isOptionSet`.

### Enumeration

Enumerations are subtypes of `ua:Enumeration` as described in
[OPC 10000-3] §5.8.3. An enumeration **MAY** be declared with or without an
explicit `subtypeOf: ua:Enumeration`; the latter form is implied when an
`enum` property is present. The `enum` property accepts either:

1. an array of bare strings — values are auto-assigned starting at 0; or
2. an array of mappings each with at least `name` and `value`, optionally
   `description` and `displayName`.

Within a single enumeration the set of `name`s **MUST** be unique and the
set of integer `value`s **MUST** be unique.

```yaml title="Example 3 — enumeration with explicit values and descriptions"
dataTypes:
  - browseName: SoftwareVersionFileType
    subtypeOf: ua:Enumeration
    enum:
      - name: Current
        value: 0
        description:
          text: The currently used version.
      - name: Pending
        value: 1
        description:
          text: The pending version that could be installed.
      - name: Fallback
        value: 2
```

### Structure

A DataType with a `fields` array is a Structure. Each field **MUST** declare
`name` and `dataType`; it **MAY** declare `description`, `valueRank`,
`arrayDimensions`, `isOptional` and `allowSubTypes`.

`allowSubTypes: true` lets the field hold a subtype of its `dataType` (the
NodeSet's `AllowSubTypes`): the Structure becomes a
`StructureWithSubtypedValues` ([OPC 10000-3] §8.51). A Structure **SHOULD NOT**
mix `allowSubTypes` and `isOptional` fields; the two select different
encodings, and `isOptional` wins.

A Structure DataType **MAY** reference itself or a sibling defined later in
the same document (forward references are permitted; see [§19](#processing-model)).

```yaml title="Example 4 — basic structure with array field"
dataTypes:
  - browseName: MyType
    fields:
      - name: Field1
        dataType: ua:Double
        valueRank: 1
```

### Union

A Structure with `isUnion: true` denotes a discriminated Union as defined in
[OPC 10000-3] §5.8.4: at most one field carries a value at run time.
`isOptional` **MUST NOT** appear on a Union field.

### Option set

A Structure with `isOptionSet: true` denotes an OptionSet DataType (a
structured bit-mask) as defined in [OPC 10000-3] §5.8.5. `isUnion` and
`isOptionSet` **MUST NOT** both be true on the same DataType.

### Companion VariableType

A Structure DataType **MAY** request the automatic creation of a companion
VariableType by setting `hasVariableType: true`. The companion VariableType
browse name is derived by replacing the trailing `DataType` suffix with
`Type` (e.g. `MyStuffDataType` → `MyStuffType`). When this convention does
not apply, the processor **MUST** still emit a deterministic browse name and
document the rule.

Each field becomes a Variable of the companion VariableType, referenced by
`HasStructuredComponent` ([OPC 10000-3] §7.22) and typed
`BaseDataVariableType`, unless the field's DataType has a companion
VariableType of its own, which is then instantiated. A field is `Optional`
when the Structure has optional fields and the field is one of them, and
`Mandatory` otherwise. A scalar Structure field whose DataType has no
VariableType carries its own fields the same way, one level deep.

`variableType` refines the generated VariableType, and implies
`hasVariableType: true`. Its `components` and `properties` entries follow two
rules: an entry that names a generated member adjusts it (`modellingRule`,
`dataType`, `valueRank`, and nested `components` for that member's own
fields), and any other entry adds a member.

```yaml title="Example 4b — the generated ResultType refined as OPC 40001-101 defines it"
dataTypes:
  - browseName: ResultDataType
    hasVariableType: true
    fields:
      - name: ResultMetaData
        dataType: ResultMetaDataType
      - name: ResultContent
        dataType: ua:BaseDataType
        valueRank: 1
    variableType:
      components:
        - browseName: ResultContent       # a generated field: only the rule changes
          modellingRule: Optional
        - browseName: ReducedResultContent # no field: a member of its own
          dataType: ua:BaseDataType
          valueRank: 1
          modellingRule: Optional
```

## ReferenceTypes

A ReferenceType definition **MUST** declare `browseName` and **SHOULD**
declare `subtypeOf` (default `ua:NonHierarchicalReferences` when omitted).
It **MAY** declare `inverseName`, `symmetric`, `isAbstract` and
`description`. The reserved hierarchical references of OPC UA Part 3
(`HasChild`, `HasComponent`, …) **MUST NOT** be redefined.

`symmetric: true` declares a ReferenceType that means the same in both
directions ([OPC 10000-3] §5.3.3), such as DI's `ConnectsTo`; it is written
`Symmetric="true"` in the NodeSet. A symmetric ReferenceType **MUST NOT**
declare `inverseName`.

When `symmetric` is omitted, a ReferenceType with an `inverseName` other
than its own browse name is not symmetric; one without an `inverseName`, or
whose `inverseName` is its own browse name, is symmetric, except that a
`Has…` name without one gets the `Is…Of` inverse name and a warning.
`symmetric: false` without `inverseName` declares a non-symmetric type
with no inverse name, as some companion specifications do (machineVision's
`FromTransition`), although [OPC 10000-3] expects an inverse name on a
concrete non-symmetric type.

## ObjectTypes and VariableTypes

An entry of `objectTypes` describes one OPC UA ObjectType. The mapping
**MUST** declare `browseName`; if `subtypeOf` is omitted the type inherits
from `ua:BaseObjectType`. `variableTypes` entries follow the same shape
with the addition of `dataType`, `valueRank`, `arrayDimensions` and `value`
(the access attributes of [§9.2](#variable-attributes) belong to Variables,
not to VariableTypes); an unspecified `subtypeOf` defaults to
`ua:BaseDataVariableType` and `dataType` is required for non-abstract
VariableTypes.

| Property                    | Type                                | Notes                                                                                  |
| --------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------- |
| `browseName`                | browse name                         | Required.                                                                              |
| `subtypeOf`                 | type reference                      | Default per node-class.                                                                |
| `isAbstract`                | boolean                             | Default `false`.                                                                       |
| `description`               | localized text                      | Markdown permitted in the long form.                                                   |
| `documentation`             | string                              | Markdown shown in the rendered docs only.                                              |
| `nodeId`                    | string NodeId                       | See [§5.4](#nodeid-assignment).                                                        |
| `defaultInstanceBrowseName` | browse name                         | See [§9.4](#default-instance-browse-name).                                             |
| `properties`                | array                               | `HasProperty` children. See [§9.1](#aggregate-members).                                |
| `components`                | array                               | `HasComponent` children.                                                               |
| `methods`                   | array                               | UAMethod members. See [§13](#methods).                                                 |
| `organizes`                 | array                               | FolderType-like children (`Organizes`).                                                |
| `orderedComponents`         | array                               | `HasOrderedComponent` children. See [§9.3](#advanced-component-containment).           |
| `physicalComponents`        | array                               | `HasPhysicalComponent` children.                                                       |
| `containedComponents`       | array                               | `HasContainedComponent` children.                                                      |
| `attachedComponents`        | array                               | `HasAttachedComponent` children.                                                       |
| `interfaces`                | array                               | InterfaceTypes implemented by this type. See [§11](#interfacetypes).                   |
| `addIns`                    | array                               | Add-Ins. See [§12](#add-ins).                                                          |
| `references`                | array                               | Per-node additional references. See [§15](#additional-references).                     |
| `generatedEvents`           | array of EventType reference        | See [§16](#events).                                                                    |
| `eventSources`              | array of browse path                | Materialised as `HasEventSource` references.                                           |
| `eventSourceOf`             | browse path                         | Inverse of `eventSources`.                                                             |
| `subtypes`                  | array of ObjectType / VariableType  | Inline subtypes. See [§9.5](#inline-subtypes).                                         |

{.props}

### Aggregate members

Aggregate properties translate to standard hierarchical ReferenceTypes
defined in [OPC 10000-3] §7.

| Property            | OPC UA ReferenceType                       | Permitted children                                                                          |
| ------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------- |
| `properties`        | `HasProperty`                              | Variables only (NodeClass *Variable* typed by `PropertyType` per [OPC 10000-3] §5.6.2).     |
| `components`        | `HasComponent`                             | Variables, Objects or Methods.                                                              |
| `orderedComponents` | `HasOrderedComponent`                      | Variables or Objects.                                                                       |
| `methods`           | `HasComponent` (NodeClass *Method*)        | UAMethod children.                                                                          |
| `organizes`         | `Organizes`                                | Objects (typically `FolderType`-rooted), Variables or Properties (the DI FunctionalGroup pattern). One with a modelling rule is an InstanceDeclaration like any aggregate: an instance gets it, and `optionals` may name members below it. |

{.props}

Each child **MUST** declare `browseName`. A child whose `typeDefinition` is
omitted inherits the default for its node-class (`ua:BaseDataVariableType`
for Variables, `ua:BaseObjectType` for Objects). A child whose `nodeClass`
is omitted has its node-class inferred from `typeDefinition` when possible,
then from the presence of `dataType` (Variable) or its absence (Object).
Authors **MAY** nevertheless write `nodeClass: Object | Variable | Method`
explicitly, and a processor **MUST** reject inconsistencies between the
explicit node-class and the inferred one.

```yaml title="Example 5 — mandatory and placeholder members"
objectTypes:
  - browseName: MySensorType
    components:
      - browseName: MyMandatoryComponent
        typeDefinition: ua:BaseObjectType
      - browseName: MyOptionalComponent
        typeDefinition: ua:BaseObjectType
        modellingRule: Optional
      - browseName: MyComponentWithNoModelingRule
        typeDefinition: ua:BaseObjectType
        modellingRule: false
      - browseName: <MyOptionalPlaceholderComponent>
        typeDefinition: ua:BaseObjectType
        modellingRule: OptionalPlaceholder
      - browseName: <MyMandatoryPlaceholderComponent>
        typeDefinition: ua:BaseObjectType
        modellingRule: MandatoryPlaceholder
```

### Variable attributes

A Variable, whether a type member or an instance, **MAY** declare the
Variable Attributes of [OPC 10000-3] §5.6.2:

| Property                  | Type          | Meaning                                                                                                   |
| ------------------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| `dataType`                | DataType ref  | The DataType; required unless inherited.                                                                  |
| `valueRank`               | integer       | Default `-1` (scalar).                                                                                    |
| `arrayDimensions`         | integer array | One entry per dimension; `0` means unknown length.                                                        |
| `value`                   | Variant       | The initial value.                                                                                        |
| `accessLevel`             | flag list     | `\|`-separated: `CurrentRead`, `CurrentWrite`, `HistoryRead`, `HistoryWrite`, `SemanticChange`, `StatusWrite`, `TimestampWrite`. |
| `userAccessLevel`         | flag list     | Same syntax; it can only narrow `accessLevel`, never widen it.                                            |
| `minimumSamplingInterval` | number        | The fastest sampling rate the server supports, in milliseconds; `0` means continuously updated.          |
| `historizing`             | boolean       | Whether the server collects history for the value.                                                        |

{.props}

A Variable typed by `ua:DataItemType` or one of its subtypes (`ua:AnalogItemType`,
`ua:AnalogUnitType`, …) **MAY** also declare the values of that type's
Properties: `definition` (string), `valuePrecision` (number),
`engineeringUnits` (a UNECE unit, by symbol or name), `euRange` and
`instrumentRange` (each `[low, high]` or `{ low, high }`). Declaring
`instrumentRange` activates the Optional `InstrumentRange` Property.

`exposesItsArray: true` on a member Variable of a type makes it the element
of its parent's array: it is the `ExposesItsArray` modelling rule
([OPC 10000-3] §6.4.4.4.5), written as a property of the member. A processor
**MUST** reject it on an instance, on a VariableType entry, next to another
`modellingRule`, and under a parent that is not a Variable or VariableType
holding an array (`valueRank` other than `-1`).

### Advanced component containment

The `orderedComponents`, `physicalComponents`, `containedComponents` and
`attachedComponents` arrays declare child references using the OPC UA
reference types `HasOrderedComponent`, `HasPhysicalComponent`,
`HasContainedComponent` and `HasAttachedComponent` respectively. The order
of items in `orderedComponents` is significant and **MUST** be preserved by
a Conformant Processor when emitting NodeSet2 XML.

```yaml title="Example 6 — advanced component containment"
objectTypes:
  - browseName: MyObjectType
    orderedComponents:
      - browseName: First
        nodeClass: Object
        typeDefinition: ua:BaseObjectType
      - browseName: Second
        nodeClass: Object
        typeDefinition: ua:BaseObjectType
    physicalComponents:
      - browseName: Phys1
        nodeClass: Object
    containedComponents:
      - browseName: Contained
        nodeClass: Object
    attachedComponents:
      - browseName: Attached1
        nodeClass: Object
```

### Default instance browse name

An ObjectType **MAY** declare a `defaultInstanceBrowseName`. When this type
is consumed as an [Add-In](#add-ins) with no explicit `browseName`, the
processor **MUST** use the default to name the instantiated child.

### Inline subtypes

An ObjectType or VariableType **MAY** declare an inline `subtypes` array.
Each entry is a full Type definition whose `subtypeOf` defaults to the
enclosing type. This convenience is equivalent to declaring the subtypes at
the document root with explicit `subtypeOf`.

## Modelling Rules and Optionals

Modelling rules describe the cardinality and constraints of a member of a
Type. This specification adopts the standard modelling rules defined in
[OPC 10000-3] §6.4.4.4 and adds one purely syntactic convenience (`false`).
The `modellingRule` property of a child member **MUST** take one of the
following values:

- `Mandatory` — the child is always present in every instance of the
  enclosing type. This is the default when the property is omitted on a
  Type-side member.
- `Optional` — the child may be present; an instance opts in by listing the
  member browse name in `optionals`, or a subtype tightens it through
  `promotedToMandatory`.
- `MandatoryPlaceholder` — at least one instance with an author-chosen
  browse name **MUST** be supplied.
- `OptionalPlaceholder` — zero or more instances with author-chosen browse
  names **MAY** be supplied.
- `ExposesItsArray` — defined by [OPC 10000-3] §6.4.4.4.5; permitted only on
  a Variable member whose parent Variable or VariableType holds an array
  (`valueRank` other than `-1`). `exposesItsArray: true` is a shorthand for it
  ([§9.2](#variable-attributes)).
- `false` (the YAML boolean) — a DSL convenience meaning "no modelling rule
  applies". The Conformant Processor **MUST NOT** emit a `HasModellingRule`
  reference for the member. It is equivalent to omitting the property
  altogether.

> [!NOTE]
> [OPC 10000-3] §6.4.4.4 defines modelling rules as an extensible concept:
> vendors may introduce additional modelling rules beyond those listed
> above. A Conformant Processor **MAY** accept additional values, but
> **MUST** reject a document if it cannot materialise them.

A modelling rule **MUST NOT** appear on a member of an instance, since
instance members do not declare cardinality.

### Placeholder browse-name convention

[OPC 10000-3] §6.4.4.4.6 and §6.4.4.4.7 recommend that the browse name and
display name of `OptionalPlaceholder` and `MandatoryPlaceholder` instance
declarations be enclosed in angle brackets. This specification tightens the
recommendation into a requirement: the browse name of a member declaring a
placeholder modelling rule **MUST** be a placeholder browse name (matching
`<Name>`), and conversely a placeholder browse name **MUST** be paired with
a placeholder modelling rule.

### Subtype and instantiation overrides

A subtype **MAY** tighten the modelling rule of an inherited member
following the rules of [OPC 10000-3] Table 21, reproduced here for
convenience:

| Rule on supertype       | Permitted rule on subtype                            |
| ----------------------- | ---------------------------------------------------- |
| `Mandatory`             | `Mandatory`                                          |
| `Optional`              | `Mandatory` or `Optional`                            |
| `MandatoryPlaceholder`  | `MandatoryPlaceholder`                               |
| `OptionalPlaceholder`   | `MandatoryPlaceholder` or `OptionalPlaceholder`      |

{.props}

A Conformant Processor **MUST** reject any override that violates this
table. Loosening (e.g. `Mandatory` → `Optional`) is not permitted.

### `optionals`

An `optionals` array enumerates browse names of inherited optional members
that *this* type or instance opts in to. Nested members **MAY** be expressed
using the dot-separated form `"Property.ChildProperty"`, mirroring the
symbolic-name convention of [OPC 10000-5] §4.1. Each segment **MAY** carry
an `alias:` prefix (`di:Identification.di:SerialNumber`), including on a
placeholder member. A Conformant Processor **MUST** resolve names by walking
the fully-inherited InstanceDeclaration hierarchy, and **MUST** report a
name that matches no member rather than ignore it.

On an instance, the array need not list an optional member the document
already assigns a value to: assigning a value implies presence, per
[§18.1](#presence-implied-by-initialization). Listing it anyway is
redundant, not an error.

### `promotedToMandatory`

A subtype, an instance, or an interface application **MAY** promote
inherited `Optional` members to `Mandatory`, or `OptionalPlaceholder`
members to `MandatoryPlaceholder`, by listing their browse names in
`promotedToMandatory`. The transformations **MUST** respect Table 21 above.
Promotion is transitive across ancestors and across implemented interfaces.

```yaml title="Example 7 — promotedToMandatory through an interface implementation"
objectTypes:
  - browseName: PlateType
    interfaces:
      - browseName: di:IVendorNameplateType
        promotedToMandatory:
          - di:DeviceClass
          - di:SerialNumber
          - di:SoftwareRevision
    defaultInstanceBrowseName: Identification
```

## InterfaceTypes

An `interfaceTypes` entry has the same shape as an ObjectType but its
`subtypeOf` defaults to `ua:BaseInterfaceType`. InterfaceTypes are subtypes
of `BaseInterfaceType` per [OPC 10000-5] and form their own hierarchy
through `subtypeOf`.

The `interfaces` property of an ObjectType, VariableType or instance lists
the InterfaceTypes that the node implements. Each entry **MUST** be a
mapping with:

- `browseName` — required; the InterfaceType reference.
- `promotedToMandatory` — optional; promotes interface members per
  [§10.4](#promotedtomandatory).
- `optionals` — optional; opts in to interface optional members per
  [§10.3](#optionals).

A Conformant Processor **MUST** add a `HasInterface` reference (defined in
[OPC 10000-5]) from the implementing Type to the InterfaceType.

### Lean materialisation

This specification adopts a *lean* interpretation of interface application
that follows OPC UA Part 3 v1.05 §6.3.3.1 to the letter:

> "Subtypes inherit the fully-inherited parent type's *InstanceDeclarations*.
> **As long as those *InstanceDeclarations* are not overridden they are not
> referenced by the subtype.**"

Combined with §4.10.2, which describes interface members as living in the
"*fully-inherited* InstanceDeclarationHierarchy of the ObjectType" (a
conceptual collection that already includes inherited and interface-derived
members), the rule a Conformant Processor **MUST** apply when an ObjectType
/ VariableType implements an interface is:

- **Mandatory** interface members are materialised directly on the
  implementing Type, with their original `Mandatory` modelling rule.
- **Optional** interface members listed in the YAML `optionals:` on the
  interface application are materialised directly on the Type, preserving
  their `Optional` modelling rule.
- Optional interface members listed in `promotedToMandatory:` are
  materialised directly on the Type with their modelling rule promoted to
  `Mandatory`.
- Every other Optional interface member is *not* referenced from the
  implementing Type. It remains reachable through the `HasInterface`
  reference and the interface's own InstanceDeclarationHierarchy.

This deliberately diverges from the eager-copy behaviour of some upstream
node-opcua releases. The lean approach keeps the interface as the single
source of truth for its members, prevents redundant nodes on every
implementer, and makes overrides — the only legitimate reason for a Type to
reference an interface member directly — visually unambiguous in the
generated NodeSet2 XML.

### Instance-time interface walk

Because the implementing Type is lean, instance materialisation **MUST**
walk the Type's `HasInterface` references (transitively up the `HasSubtype`
chain) and apply each interface to the instance under the lean rules of
§11.1. Specifically, for every interface implemented (directly or by
inheritance) by the instance's `typeDefinition`:

- Mandatory members of the interface **MUST** be materialised on the
  instance (a Server creating an instance must reproduce the
  fully-inherited InstanceDeclarationHierarchy — Part 3 §6.4).
- Optional members listed in the instance's `optionals:` **MUST** be
  materialised on the instance.
- Other Optional members **MUST NOT** be materialised.

The `HasInterface` reference itself stays on the TypeDefinitionNode; an
instance **MUST NOT** add a duplicate `HasInterface` reference (Part 3
§4.10.2: "Instances should not have a HasInterface Reference to an
Interface if it was already applied to the TypeDefinitionNode").

### Instance with its own `interfaces:` block

An instance **MAY** declare its own `interfaces:` block to apply an
interface that the TypeDefinitionNode does not already implement. In that
case the instance:

- **MUST** add a `HasInterface` reference to the interface; and
- **MUST** materialise members per the lean rules of §11.1, treating the
  per-entry `optionals:` and `promotedToMandatory:` lists as overrides on
  the instance.

```yaml title="Example 8 — interface application with promotion and per-instance optionals"
interfaceTypes:
  - browseName: ILevelType
    properties:
      - browseName: L1Prop
        dataType: ua:String
  - browseName: ILevel2Type
    subtypeOf: ILevelType
    properties:
      - browseName: L2Prop1
        dataType: ua:String
      - browseName: L2Prop2
        dataType: ua:String
        modellingRule: Optional

objectTypes:
  - browseName: XYZDeviceType
    interfaces:
      - browseName: ILevel2Type

instances:
  - browseName: XYZInstance1
    typeDefinition: XYZDeviceType
    organizedBy: /ua:Objects
    optionals: [L2Prop2]
```

## Add-Ins

An Add-In is an Object aggregate referenced by its `typeDefinition` rather
than enumerated member-by-member. The `addIns` property of a Type or
instance **MAY** list Add-Ins; each entry is a mapping with at least
`typeDefinition` and optionally `browseName`, `description`,
`modellingRule` and `optionals`. An entry **MAY** also declare members of
its own beyond those of its type (`components`, `properties`, `methods`)
and nested `addIns`, attached to it by `HasAddIn` (OPC 40001-1's
`MachineryBuildingBlocks` folder carrying LifetimeCounters and Monitoring).

The browse name of the materialised Add-In instance is determined as
follows, in order:

1. the explicit `browseName` of the Add-In entry, if present;
2. otherwise the `defaultInstanceBrowseName` of the Add-In's
   `typeDefinition`;
3. otherwise the `browseName` of the `typeDefinition` with the trailing
   `Type` suffix removed.

A Conformant Processor **MUST** reject a document if rule 3 is reached but
the resulting browse name is empty or collides with an existing sibling.

```yaml title="Example 9 — multiple Add-Ins of the same type with explicit browse names"
objectTypes:
  - browseName: ThermometerComponentType
    components:
      - browseName: Temperature
        dataType: ua:Float
    defaultInstanceBrowseName: Thermometer

instances:
  - browseName: Equipment
    typeDefinition: ua:BaseObjectType
    organizedBy: /ua:Objects
    addIns:
      - typeDefinition: ThermometerComponentType
        browseName: MyThermometer1
      - typeDefinition: ThermometerComponentType
        browseName: MyThermometer2
```

## Methods

A method declaration **MUST** declare `browseName`. `inputArguments` and
`outputArguments` are ordered arrays of mappings; each entry **MUST**
declare `name` and `dataType` and **MAY** declare `description`,
`valueRank` (default `-1`) and `arrayDimensions`.

On a Type-side member, a method **MAY** declare `modellingRule`; on an
instance-side member it **MUST NOT**. A method **MAY** declare an
`executable` boolean (default `true`).

## Instances

An entry of the top-level `instances` array describes an Object or Variable
instance. The mapping **MUST** declare `browseName` and `typeDefinition`,
and **MUST** declare its parent through exactly one of the
[connection properties](#connected-object-rule). An instance **MAY**
additionally declare any of:

| Property                                                    | Notes                                                                                                                         |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `nodeId`                                                    | Explicit string NodeId; see [§5.4](#nodeid-assignment).                                                                       |
| `description`, `displayName`                                | Localized text overrides.                                                                                                     |
| `optionals`                                                 | Inherited optional members to instantiate.                                                                                    |
| `interfaces`                                                | InterfaceTypes implemented by this instance.                                                                                  |
| `addIns`                                                    | Add-Ins to instantiate; see [§12](#add-ins).                                                                                  |
| `properties`, `components`, `organizes`, `methods`          | Inline children that override or augment the instantiated members; see [§14.2](#inline-children-and-overrides).               |
| `propertyValues`                                            | Values for the instance's members, addressed by nesting; see [§14.4](#propertyvalues).                                        |
| `initializers`                                              | Values for the instance's members, addressed by browse path; see [§18](#initializers).                                        |
| `value`, `accessLevel`                                      | Permitted only when the instance is a Variable (i.e. `typeDefinition` resolves to a VariableType).                            |

{.props}

### Connected-object rule

Every Object or Variable in an OPC UA address space is reached from the
Server's Objects, Types or Views folder through hierarchical References
([OPC 10000-3] §7.5). Accordingly an instance in this DSL **MUST** declare
its parent through *exactly one* of the following properties; the JSON
Schema `ConnectedObject` definition enforces this with a `oneOf`:

| Property               | OPC UA reference          |
| ---------------------- | ------------------------- |
| `organizedBy`          | `Organizes`               |
| `componentOf`          | `HasComponent`            |
| `propertyOf`           | `HasProperty`             |
| `orderedComponentOf`   | `HasOrderedComponent`     |
| `physicalComponentOf`  | `HasPhysicalComponent`    |
| `containedComponentOf` | `HasContainedComponent`   |

{.props}

The value of each connection property is a browse path (absolute, anchored
or relative). Two top-level instances **MAY** use different connection
properties.

### Inline children and overrides

An instance **MAY** include `properties`, `components`, `organizes` or
`methods` arrays. The processor **MUST** match each inline child by
`browseName` against the members materialised from `typeDefinition` (and
from implemented interfaces / add-ins):

1. If a matching member exists, the inline child *overrides* it and **MAY**
   set `value`, `description`, `displayName`, `dataType` (when narrowing is
   permitted by OPC UA), `valueRank`, `arrayDimensions`, `accessLevel`,
   sub-aggregates and inline `references`. The inline child **MUST NOT**
   declare a `modellingRule`.
2. If no matching member exists, the inline child is treated as a new
   member added to this instance. The processor **MUST** require
   `typeDefinition` in this case for non-Method children.

```yaml title="Example 10 — overriding a nested component on an instance"
objectTypes:
  - browseName: MySubObjectType
    components:
      - browseName: ComponentA
        nodeClass: Object
      - browseName: ComponentVarB
        nodeClass: Variable
        dataType: ua:Double
        valueRank: -2          # any
  - browseName: MyObjectType
    components:
      - browseName: SubObject1
        typeDefinition: MySubObjectType
        components:
          - browseName: ComponentVarB
            dataType: ua:Double
            valueRank: 1        # narrowed to array
```

### Browse-name uniqueness

Two distinct nodes **MAY** share the same browse name as long as they do
not share the same hierarchical parent. A processor **MUST** reject
documents in which two sibling nodes share the same browse name. This rule
applies uniformly to top-level instances, inline children and members
materialised through Add-Ins or interfaces.

```yaml title="Example 11 — same browse name in two distinct folders"
instances:
  - browseName: MyInstance
    typeDefinition: MyObjectType
    organizedBy: /ua:Objects

  - browseName: SomeLittleThing
    typeDefinition: LittleThingType
    componentOf: MyInstance/Folder1

  - browseName: SomeLittleThing
    typeDefinition: LittleThingType
    componentOf: MyInstance/Folder2
```

### PropertyValues

`propertyValues` assigns values to an instance's members by *nesting* rather
than by path. It is a mapping whose shape mirrors the instance's aggregate
structure: each key is a member browse name, alias-qualified per
[§5.3](#namespace-aliases), and each value is either the value to assign or a
nested mapping addressing that member's own children.

```yaml title="Example 12 — nesting mirrors the aggregate structure"
instances:
  - browseName: Pump1
    typeDefinition: di:DeviceType
    organizedBy: /ua:Objects
    propertyValues:
      di:Manufacturer: Sterfive
      di:ParameterSet:
        Pressure: 4.2
```

A Conformant Processor **MUST** resolve each key against the children of the
node the enclosing mapping addresses, and **MUST** reject a key that names no
such member rather than creating one — a misspelled member is an authoring
error, not a new node. A leaf **MUST** address a Variable; assigning to an
Object is an error.

#### `$value`

A member that carries both a value and children of its own is addressed with
the reserved key `$value`, which assigns to the enclosing Variable itself:

```yaml title="Example 13 — a value alongside its own sub-properties"
    propertyValues:
      Level:
        $value: 42.0
        ua:EngineeringUnits: "%"
```

`$value` **MUST** be rejected when the enclosing node is not a Variable.

#### Value literals

A mapping is normally a sub-tree, which makes the two OPC UA structures that
a model most often writes inline ambiguous. A Conformant Processor **MUST**
therefore treat a mapping as a VALUE, not a sub-tree, when — and only when —
it matches one of these shapes against the target Variable's DataType:

- `{ text, locale? }` where the basic DataType is `LocalizedText`;
- `{ low, high }` where the DataType is `Range`.

The test **MUST** be applied before recursing, and **MUST NOT** be relaxed to
a subset or superset of those keys: `{ txt: "Acme" }` is not a LocalizedText
with a typo, it is a reference to a child named `txt`, and **MUST** produce
the same "no such member" error any other unknown key would. Every other
mapping recurses.

#### Relationship to `initializers`

`propertyValues` and [`initializers`](#initializers) express the same thing —
values for existing Variables — and differ only in how the target is
addressed: by nesting, or by a browse path. A document **MAY** use either or
both. Both are subject to [§18.1](#presence-implied-by-initialization): a
value written at any depth of a `propertyValues` tree opts every `Optional`
node on its path in.

## Additional References

The top-level `references` array, and the per-node `references` sub-arrays
on Types, instances and add-ins, declare references that are not
expressible through aggregates or connection properties (typically
non-hierarchical or cross-namespace references).

Each entry **MUST** be a mapping with:

- `referenceType` — required; browse name of a ReferenceType.
- `source` — required, in both forms; in the per-node form a browse path is
  resolved from the node that holds the array, and `/` is that node itself.
  There is no implicit source.
- `target` — required; browse path of the target node.
- `isForward` — optional boolean; default `true`.

In the per-node form, `target` **MAY** be a node declaration instead of a
browse path. The node is then created under the owner (which becomes its
ParentNodeId), linked by `referenceType` alone: the way to declare a member
owned through a ReferenceType with no dedicated key. DI does this with
`ConnectsTo`, a non-hierarchical ReferenceType. `source` **MUST** be `/`;
`referenceType` **MUST NOT** be `HasTypeDefinition`, `HasModellingRule`,
`HasSubtype`, `HasEncoding` or `GeneratesEvent`.

```yaml title="Example 13b — a placeholder declared through ConnectsTo (OPC 10000-100 DI)"
objectTypes:
  - browseName: NetworkType
    references:
      - referenceType: ConnectsTo
        source: /
        target:
          browseName: <CPIdentifier>
          typeDefinition: ConnectionPointType
          modellingRule: OptionalPlaceholder
```

## Events

Event types are declared in the top-level `eventTypes` array; each entry
has the shape of an ObjectType with default `subtypeOf: ua:BaseEventType`.

The `generatedEvents` property of any Type lists the event types it raises;
the processor **MUST** emit a `GeneratesEvent` reference for each entry.
The `eventSources` / `eventSourceOf` properties materialise
`HasEventSource` references.

```yaml title="Example 14 — event source declaration"
objectTypes:
  - browseName: MyObjectType
    generatedEvents:
      - MyEventType

eventTypes:
  - browseName: MyEventType
```

## State Machines

A `stateMachines` entry produces a FiniteStateMachine type as defined in
[OPC 10000-5] §B (FiniteStateMachineType, StateType, TransitionType). The
default `subtypeOf` is `ua:FiniteStateMachineType`. The mapping **MUST**
declare:

- `browseName` — the type browse name;
- `states` — either an array of state browse names (numeric values
  auto-assigned starting at 0) or an array of mappings
  `{ name, value, description?, isInitialState? }`;
- `transitions` — an array of mappings, each with at minimum `from` and
  `to`; an optional `browseName`, `value` and `description` **MAY** be
  provided. Default transition browse names follow the pattern
  `<FromState>To<ToState>`.

A state machine **MAY** additionally declare `subtypeOf`, `isAbstract`,
`transitionHaveEffect` (the EventType raised on each transition),
`components`, `methods` (e.g. the Start, Suspend, Resume, Halt and Reset
methods of a ProgramStateMachine) and `promotedToMandatory`.

```yaml title="Example 15 — FiniteStateMachine type"
stateMachines:
  - browseName: MyFiniteStateMachine
    states: [LowLow, Low, High, HighHigh]
    transitions:
      - from: LowLow
        to: Low
      - from: Low
        to: High
      - from: High
        to: HighHigh
      - from: HighHigh
        to: High
      - from: High
        to: Low
      - from: Low
        to: LowLow
```

## Initializers

The `initializers` array declares default values for existing variables,
identified by `variable` (a browse path, in which `.` **MAY** replace `/` to
reach a `HasProperty` child, as in `di:ParameterSet/Temperature.ua:EngineeringUnits`).
Each entry **MUST** declare
`variable` and `value`; `engineeringUnits`, `range` and `description` are
optional and apply only to AnalogItem-like variables.

An `initializers` block **MAY** appear on an instance, where `variable` is a
path relative to that instance, or at the top level of the document, where
the first segment of `variable` **MUST** name a symbol — an instance
declared by the document, written with or without the `$` sigil. The two
forms are equivalent: `$Robot1/FlangeLoad` written at the top level and
`FlangeLoad` written inside `Robot1` denote the same Variable and **MUST**
have the same effect.

Initializers are applied *after* all node materialisation has completed and
*before* NodeSet2 XML serialisation. They **MUST NOT** create or remove
nodes.

### Presence implied by initialization

Assigning a value implies the presence of the node that receives it. Where
`variable` resolves, through the fully-inherited InstanceDeclaration
hierarchy, to a member whose modelling rule is `Optional` — or reaches such
a member through an `Optional` ancestor — a Conformant Processor **MUST**
treat every `Optional` node on that path as though it had been listed in
the target instance's [`optionals`](#optionals) array.

This is a materialisation-phase rule, not an exception to the paragraph
above: the nodes are created in [phase 4](#processing-model) like any other
opted-in optional member, and phase 6 still only writes values to nodes that
already exist. A processor that defers the decision to phase 6 cannot
implement it, because `instantiate()` has already run.

The same rule governs [`propertyValues`](#propertyvalues), where a value
written at any depth of the tree opts its path in.

An entry that names a member the type does not declare **MUST NOT** be
opted in — it is an authoring error, and the diagnostic a processor already
emits for an unresolvable `variable` path is the correct response.

## Processing Model

A Conformant Processor **MUST** process a document in the following phases:

1. **Parse and schema-validate.** Reject documents that fail YAML parsing
   or the JSON Schema validation.
2. **Resolve namespaces.** Load every namespace declared in `namespaces`
   and `imports`; assign each a stable namespace index.
3. **Topological sort.** Build a dependency graph over types defined in the
   document; resolve forward references; reject cycles unless the cycle is
   a self-reference permitted by OPC UA (e.g. a Structure field whose
   `dataType` is its enclosing type).
4. **Materialise nodes.** Create ReferenceTypes, DataTypes, ObjectTypes,
   VariableTypes, EventTypes, InterfaceTypes, FiniteStateMachine types,
   then top-level instances. For each Type, copy inherited members, apply
   `interfaces`, `addIns`, `promotedToMandatory`, `optionals` and inline
   children (in that order). Before an instance is materialised, its
   `optionals` set is extended with the `Optional` members implied by the
   values the document assigns to it — from its own `initializers` and
   `propertyValues`, and from the top-level `initializers` entries anchored
   at it (see [§18.1](#presence-implied-by-initialization)). The top-level
   block is only readable this early because its paths are rooted at
   symbols, not at nodes.
5. **Apply additional references.** Resolve and add every entry of the
   top-level and per-node `references` arrays.
6. **Apply initializers.** Set Variant values on existing Variable nodes.
7. **Persist NodeIds.** Update the symbol table so that all auto-assigned
   NodeIds remain stable across regenerations.
8. **Serialise.** Emit a NodeSet2 XML document that is
   [NodeSet2-equivalent](#equivalence-with-nodeset2-xml) to the address
   space described by the document.

## Equivalence with NodeSet2 XML

A NodeSet2 XML document is described by [OPC 10000-6]. Two address spaces
are *NodeSet2-equivalent* if and only if, after canonicalisation, they
declare:

1. the same set of nodes (matched by NodeId after namespace-index
   remapping); each node has the same NodeClass and the same set of
   Attributes as defined in [OPC 10000-3] §5;
2. identical Attribute values for each node, modulo whitespace in
   LocalizedText fields;
3. identical hierarchical and non-hierarchical References, modulo ordering
   of `References` elements except where ordering is mandated by the
   ReferenceType (e.g. `HasOrderedComponent`);
4. identical InstanceDeclaration hierarchies for every TypeDefinitionNode,
   preserving every `HasModellingRule` reference required by the modelling
   rules of [§10](#modelling-rules-and-optionals).

A Conformant Processor **MUST** produce, for any Conformant Document, a
NodeSet2 XML document that is NodeSet2-equivalent to the address space
described by the document.

## Diagnostics

A Conformant Processor **MUST** reject a document and emit a diagnostic
when:

- the JSON Schema validation fails;
- a referenced browse name, browse path or namespace alias cannot be
  resolved;
- two sibling nodes share the same browse name;
- an instance does not satisfy the
  [connected-object rule](#connected-object-rule);
- a `modellingRule` is used on an instance-side member;
- a placeholder browse name is not paired with a placeholder modelling rule;
- an enumeration field has a duplicate `name` or `value`;
- a Structure declares both `isUnion: true` and `isOptionSet: true`;
- an Add-In's resolved browse name is empty or collides with an existing
  sibling;
- a state-machine transition references an unknown state;
- a property is encountered that is not defined by this specification on a
  section that has `additionalProperties: false`.

A processor **SHOULD** emit a non-fatal warning when:

- an unused `imports` entry, an unused symbol-table NodeId or an empty
  optional array is encountered;
- a node uses an explicit string NodeId that conflicts with an existing
  entry in the symbol table.

## Security Considerations

The DSL is a static description of an OPC UA model and does not by itself
transmit data over the network. However:

- Documents may reference local files via `imports[].filename`.
  Implementations **SHOULD** reject paths that escape the project root
  unless the user has explicitly opted in.
- The Markdown `documentation` block is rendered into external artefacts.
  Implementations **MUST** sanitise any embedded HTML according to the
  rules of the target output format before rendering it to a browser.
- Initializers ([§18](#initializers)) accept only literal scalar or array
  values, so they do not in themselves execute code; implementations that
  extend the format with programmable initializers **MUST** sandbox the
  evaluation context.

## References

[OPC 10000-3]
:   OPC Foundation, *OPC Unified Architecture — Part 3: Address Space
    Model*, version 1.05.
    <https://reference.opcfoundation.org/Core/Part3/v105/docs/>

[OPC 10000-5]
:   OPC Foundation, *OPC Unified Architecture — Part 5: Information Model*,
    version 1.05.
    <https://reference.opcfoundation.org/core/Part5/v105/docs/>

[OPC 10000-6]
:   OPC Foundation, *OPC Unified Architecture — Part 6: Mappings* (defines
    the NodeSet2 / UANodeSet XML schema).
    <https://reference.opcfoundation.org/Core/Part6/>

[YAML 1.2.2]
:   YAML Language Development Team, *YAML Ain't Markup Language (YAML™)
    version 1.2 (revision 1.2.2)*.
    <https://yaml.org/spec/1.2.2/>

[JSON Schema 2020-12]
:   JSON Schema Organisation, *JSON Schema Specification, Draft 2020-12*.
    <https://json-schema.org/draft/2020-12/json-schema-core.html>

[RFC 2119]
:   Bradner, S., *Key words for use in RFCs to Indicate Requirement
    Levels*, IETF, 1997.
    <https://www.rfc-editor.org/rfc/rfc2119>

[RFC 8174]
:   Leiba, B., *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key
    Words*, IETF, 2017.
    <https://www.rfc-editor.org/rfc/rfc8174>

[RFC 3339]
:   Klyne, G., Newman, C., *Date and Time on the Internet: Timestamps*,
    IETF, 2002.
    <https://www.rfc-editor.org/rfc/rfc3339>
