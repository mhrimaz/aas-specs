# Resource Description Framework

The Resource Description Framework ([RDF](https://www.w3.org/TR/rdf11-primer/)) is recommended standard of the W3C to unambiguously model and present semantic data. RDF documents are structured in the form of triples, consisting of subjects, relations and objects. The resulting model is often interpreted as a graph, with the subject and object elements as the nodes and the relations as the graph edges.

RDF is closely related to Web standards, illustrated by the fact that all elements are encoded using (HTTP-)URIs. As a common practice, the provision of additional information at the referenced location of an RDF entity directly allows the interlinking of entities based on the Web. This process, the following of links in order to discover related information, is called dereferencing a resource and is supported by any browser or web client. As the AAS namespace (`https://admin-shell.io/aas/<version>/<revision>/`) is supporting content negotiation. This means that `curl --location 'https://admin-shell.io/aas/3/0/' --header 'Accept: text/turtle'` will return the Ontology of AAS. Connecting the capabilities of RDF with the expressiveness of the Asset Administration Shell is one motivation for the RDF serialization.

Combining both worlds, the RDF mapping of the Asset Administration Shell can provide the basis for complex queries and requests. SPARQL, the standard query language for the Semantic Web, can combine reasoning features with the integration of external data sources. In order to benefit of these abilities, the AAS requires a clear scheme of its RDF representation. In the following, the necessary transformation rules are presented, followed by an illustration of relevant parts of the scheme and an example. The complete [data model](rdf-ontology.ttl) together with the [schema](shacl-schema.ttl) are listed in this repository.

## RDF Mapping Rules

The concepts of the RDF and the derived RDF serialization of the AAS are explained by the mapping rules. These rules are implemented by the [generators](https://github.com/aas-core-works/aas-core-codegen) used to create the ontology and shacl files based on the independent project [aas-core-works](https://github.com/aas-core-works/). The main design rules the following:

### Default Serialization Format
- Turtle (`.ttl`) MUST be used as the default serialization format for both AAS Ontology and SHACL shapes.
- Other RDF serializations (e.g., JSON-LD, RDF/XML, N-Triples) MAY be supported, provided they maintain equivalent semantics to the Turtle representation.

### Data Instance Serialization

- RDF graphs MUST be used to represent AAS data instances.
- Identifiers for AAS elements MUST be globally unique IRIs.

Every entity MUST be encoded as either an IRI or a Literal:

1. **IRIs for Entities and Relations**:
   - RDF uses IRIs to represent both entities and relationships.
   - If no predefined IRI exists for an AAS element, a globally unique IRI MUST be generated.
   - For RDF resources (`rdf:Resource`) that do not have a corresponding element in AAS, the generation of IRIs is delegated to the client.

2. **Client and Implementor Responsibilities**:
   - Clients and implementors MUST NOT rely on the specific structure or format of an IRI.
   - Instead, they MUST depend solely on the elements defined within the AAS specification to ensure interoperability.
   - While a client MAY adopt a specific scheme for generating IRIs (e.g., for performance optimization or other valid reasons), these implementations MUST NOT alter the semantics defined by the AAS ontology.

3. **Typed Literals for Primitive Values**:
   - Primitive values MUST be encoded as **Typed Literals** in RDF.
   - Appropriate `xsd` datatypes (e.g., `xsd:string`, `xsd:integer`, `xsd:dateTime`) MUST be used to ensure semantic clarity.

4. **Resolution of IRIs**:
   - While AAS has its own resolution mechanism for ids, it is not mandatory for IRIs to be **resolvable** or to support **content negotiation**.
   - However, it is considered **best practice** to make IRIs resolvable and to support content negotiation to enhance interoperability and provide richer semantics.

### Round-Tripability of Serializations

The AAS specification supports multiple serialization formats, including **JSON**, **XML**, and **RDF**. All these formats SHOULD be **round-tripable**, meaning that data serialized in one format must be convertible back into the original format without loss of information or change in semantics.

1. **Round-Trip Consistency**:
   - Clients MUST ensure that a **canonicalized JSON** representation of AAS produces a valid RDF graph.
   - The RDF graph MUST be able to be round-tripped back into the same **canonicalized JSON** representation.
   - This round-trip process ensures that no data is lost or altered during the conversion between serialization formats.

2. **Handling Lists in JSON**:
   - Since JSON serializes elements that do not semantically have an order (such as elements within a `Submodel`) as lists, special care MUST be taken when handling such elements.
   - To preserve semantic meaning and ensure consistent round-tripability, a **CanonicalAAS/JSON** representation SHOULD be used for lists where order does not matter.

3. **Client and Implementor Responsibility**:
   - Regardless of the serialization format used, **clients MUST ensure round-tripability** between JSON, XML, and RDF formats.
   - Clients MUST ensure that converting data between these formats does not alter the semantic content or structure of the AAS instance.

This principle guarantees that AAS data remains consistent and accurately represented, regardless of the serialization format used, promoting interoperability across different systems and tools.

### Extension of AAS Instances

Implementors MAY decide to add further information internally to instances of AAS, such as using external vocabularies like **DCAT** for data provenance. However, such extra information MUST NOT be communicated or shared in a way that extends beyond the AAS specification.

1. **Interoperability Considerations**:
   - The inclusion of additional information is permitted for internal use by the implementor but MUST NOT be communicated externally.
   - This restriction is in place to ensure **interoperability** across implementations and to reduce **heterogeneity** in AAS data exchanges.

2. **Client and Library Behavior**:
   - Clients and libraries processing AAS/RDF MUST **ignore** any extra attributes that are outside the scope of AAS.
   - Clients and libraries MUST only raise an error if there is an issue within the scope of AAS, including violations of its semantic model or structural integrity.
   - If extra attributes or non-AAS-related data are encountered, they should be treated as non-relevant and should not interfere with the core processing of AAS data.

This ensures that AAS data instances remain consistent and standardized across implementations, preserving the core semantics of AAS while allowing for internal extensions where necessary.

### Note: Pattern Deviation from the Specification
Since most RDFS engines we tested operated on UTF-16 and could not handle UTF-32, we transpiled the pattern from AASd-130, which uses UTF-32 in the specification, into UTF-16. This is a trade-off between correctness and practicality. See [#362](https://github.com/admin-shell-io/aas-specs/issues/362) for the details.

## Example Overview

RDF is often regarded as a graph model, as it provides the flexibility to interlink entities at any stage. In the following, the [complete example](./examples/Complete_Example.ttl) is originally provided in Turtle but accompanied with visualizations of the represented graph (see Fig. 1): Attributes referencing non-literal values are shown as directed links while Literal values are drawn together with the corresponding entity itself. In order to increase readability, the namespace declaring sections are omitted. The complete example with all namespaces can be found in the [example folder](examples). One can see the additionally inserted triples for `rdf:type`, `rdfs:label`, and `rdfs:comment` as determined by Rule 4. The first attribute states the instance’ class. The second provides its commonly used name, for instance based on the idShort attribute. `rdfs:comment` supplies a short description about the regarded entity, based on the description value. The generally available tools, for instance the open source tool [Protégé](https://protege.stanford.edu), render these attributes and display the correct class hierarchy, render the elements with their labels or supply short explanations based on the comments.

![Simplified graph of the core classes in the example](../rdf/media/aas-rdf-graph.png)

___Figure 1: Simplified graph of the core classes in the example___

A comprehensive set of generated examples is also provided in the subfolder [examples/generated](./examples/generated), always containing a complete and a minimal version of each class. The files have been created using the [aas-core3.0-testgen](https://github.com/aas-core-works/aas-core3.0-testgen) project to simplify the maintenance process and to stick directly to the efforts made at [aas-core-meta](https://github.com/aas-core-works/aas-core-meta).
