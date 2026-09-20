# OPC UA Modeler

**Write OPC UA information models in YAML. Get standard NodeSet2.xml.**

OPC UA Modeler, by [Sterfive](https://www.sterfive.com/en/product/opcua-modeler) — the maintainers of
[node-opcua](https://github.com/node-opcua/node-opcua) — is a modelling toolchain for OPC UA information models and
companion specifications. You describe your types and instances in a small, readable YAML language; the compiler checks
the model against the companion specifications it builds on and generates the `NodeSet2.xml` that any OPC UA server
loads.

```yaml
namespaceUri: http://acme.example/UA/Pump/
version: 1.0.0
namespaces:
  - di
objectTypes:
  - browseName: PumpType
    subtypeOf: di:DeviceType
    components:
      - browseName: di:ParameterSet
        components:
          - browseName: FlowRate
            dataType: ua:Double
          - browseName: SetPoint
            dataType: ua:Double
            accessLevel: CurrentRead | CurrentWrite
```

That is a complete model: a pump derived from the Device Integration (DI) companion specification. The generated
NodeSet2.xml carries the namespace table, the `RequiredModel` entries for OPC UA and DI, stable NodeIds, and every
mandatory member the base types imply.

## Where to use it

| | |
|---|---|
| **In your browser** | [opcua-modeler.sterfive.io](https://opcua-modeler.sterfive.io) — editor with completion, live diagnostics, diagrams, and a button that starts a real OPC UA server on your model |
| **In Visual Studio Code** | [Marketplace extension](https://marketplace.visualstudio.com/items?itemName=sterfive.opcua-modeler) |
| **On the command line and in CI** | [CLI quickstart](https://opcua-modeler.doc.sterfive.com/docs/start/cli) |
| **From an AI agent** | [MCP server](https://github.com/node-opcua/node-opcua-modeler-mcp-server) — `npx -y node-opcua-modeler-mcp-server` |

Documentation: **[opcua-modeler.doc.sterfive.com](https://opcua-modeler.doc.sterfive.com/)**.
New to the file format? Start with
[How to create a NodeSet2.xml file](https://opcua-modeler.doc.sterfive.com/docs/guides/create-nodeset2-xml).

## What is in this repository

This repository holds the **open parts** of OPC UA Modeler. The compiler, the language server and the web application
are commercial products and are not here.

| Folder | Content | Licence |
|---|---|---|
| [`specification/`](./specification/specification.md) | The **OPC UA YAML DSL specification**: the language, its semantics and its mapping to NodeSet2 XML. Written so that independent implementations are possible. | CC BY 4.0 |
| [`schema/`](./schema) | The **JSON Schema** of the language, for validation and editor completion | MIT |
| [`examples/`](./examples) | **Example models**, the same ones the documentation uses and compiles on every build | MIT |

### Completion in any editor

Any editor that speaks the YAML language server gets completion and validation from the schema alone. Put this on the
first line of your model:

```yaml
# yaml-language-server: $schema=https://opcua-modeler.doc.sterfive.com/nodeset2.schema.json
```

The schema checks the shape of the document. Resolving types across companion specifications, modelling rules and
NodeId allocation is what the compiler adds.

## Why model as code

An information model has versions, reviewers and a release date, like the software that serves it. As a text file it
gets what that software already has: readable diffs, review on a merge request, merges between colleagues, and a
pipeline that refuses a change breaking a modelling rule. Comparisons with the graphical tools:
[UaModeler, SiOME, UA-ModelCompiler](https://www.sterfive.com/en/blog/opcua-modeler-vs-uamodeler) and the
[OPC Foundation's UA NodeSet Editor](https://www.sterfive.com/en/blog/opcua-modeler-vs-ua-nodeset-editor).

Already have a NodeSet2.xml? `opcua-modeler reverse` turns it into YAML.

## Questions, bugs, ideas

Open an [issue](../../issues). Questions about the language and the specification are as welcome as bug reports. For
commercial support and licences, [contact Sterfive](https://www.sterfive.com/en/contact).

## How this repository is maintained

It is a **mirror**, generated from Sterfive's internal repository by a script. A pull request cannot be merged here as
it is: open an issue, or a pull request as a way to show a proposed change, and it will be applied at the source and
flow back on the next sync.

## Licence

Specification: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Schema and examples: [MIT](./LICENSE-MIT).
See [LICENSE](./LICENSE). "OPC UA" is a trademark of the OPC Foundation.
