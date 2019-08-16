# JSONx

> **JSON Schema for the enterprise**

## Abstract

The <ins>JSONx</ins> project provides the <ins>JSON Schema Definition Language</ins> for JSON, which is a <ins>schema language</ins> designed in close resemblance to the [XMLSchema][xmlschema] specification. The <ins>schema language</ins> extends the capabilities found in JSON documents, by allowing for the description of the structure and to constrain the contents of JSON documents. The <ins>schema language</ins> is represented in two different but equally translatable vocabularies: a JSON vocabulary, and an XML vocabulary.

This document introduces the <ins>JSONx</ins> project, and presents a directory of links to its constituent parts and related resources.

## Table of Contents

<samp>&nbsp;&nbsp;</samp>1 [Introduction](#1-introduction)<br>
<samp>&nbsp;&nbsp;&nbsp;&nbsp;</samp>1.1 [Conventions Used in This Document](#11-conventions-used-in-this-document)<br>
<samp>&nbsp;&nbsp;</samp>2 [Use-Cases](#2-use-cases)<br>
<samp>&nbsp;&nbsp;&nbsp;&nbsp;</samp>2.1 [Consumer Driven Contracts](#21-consumer-driven-contracts)<br>
<samp>&nbsp;&nbsp;</samp>3 [<ins>JSON Schema Definition Language</ins>][#jsd]<br>
<samp>&nbsp;&nbsp;</samp>4 [<ins>JSONx Framework for Java</ins>](#4-jsonx-framework-for-java)<br>
<samp>&nbsp;&nbsp;</samp>5 [Contributing](#5-contributing)<br>
<samp>&nbsp;&nbsp;</samp>6 [License](#6-license)

## <b>1</b> Introduction

The <ins>JSONx</ins> project was created to help developers address common problems and use-cases when working with JSON documents. The <ins>JSONx</ins> project offers <ins>structural</ins> and <ins>functional</ins> patterns that systematically reduce errors and pain-points commonly encountered when developing software that interfaces with JSON. The <ins>structural</ins> patterns are defined in the [<ins>JSON Schema Definition Language</ins>][schema], which is a programming-language-agnostic <ins>schema language</ins> used to describe constraints and document the meaning, usage and relationships of the constituent parts of JSON documents. The <ins>functional</ins> patterns are reference implementations of the specification of the <ins>schema language</ins>, providing utilities that address common use-cases for applications that use JSON in one way or another. Common use-cases include:

1. Definition of a normative contract between a producer and consumer of JSON documents.
1. Validation of JSON documents conforming to a respective <ins>schema document</ins>.
1. Java class binding API for JSON documents conforming to a respective <ins>schema document</ins>.

### <b>1.1</b> Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://www.ietf.org/rfc/rfc2119.txt).

## <b>2</b> Use-Cases

The following sections lists common use-cases where <ins>JSONx</ins> project can be used.

### <b>2.1</b> Consumer Driven Contracts

The <ins>JSONx</ins> project was created specifically with [<ins>Consumer Driven Contracts</ins>][cdc] in mind. With the [<ins>JSON Schema Definition Language (JSD)</ins>][#jsd], one can create a <ins>Consumer Driven Contract (CDC)</ins> with a model that includes the capacity to evolve based on schema versioning. Additionally, the <ins>JSD</ins> can be used by producers and consumers to validate documents in a communication protocol.

The following example illustrates a simple protocol that uses the CDC approach, and consists of the actors:

1. **Producer**: Representing the provider of the <ins>ProductSearch</ins> service.
1. **Consumer1**: The first consumer of the <ins>ProductSearch</ins> service.
1. **Consumer2**: The second consumer of the <ins>ProductSearch</ins> service.

Consider a simple <ins>ProductSearch</ins> service, which allows consumer applications to search a product catalogue.

Version **v1** of the protocol defines the contract:

* **Request**

  ```
  GET /ProductSearch?name=<name>
  ```

* **Response**

  ```diff
  {
    "Version": "v1",
    "CatalogueID": <number>,
    "Name": <string>,
    "Price": <string>,
    "Manufacturer": <string>,
    "InStock": <boolean>
  }
  ```

The schema that describes the **Response** contract is:

<!-- tabs:start -->

###### **JSD**

```json
{
  "jx:ns": "http://www.jsonx.org/schema-0.3.jsd",
  "jx:schemaLocation": "http://www.jsonx.org/schema-0.3.jsd http://www.jsonx.org/schema.jsd",

  "product": { "jx:type": "object", "abstract": true, "properties": {
    "CatalogueID": { "jx:type": "number", "range": "[1,]", "scale": 0, "nullable": false},
    "Name": { "jx:type": "string", "pattern": "\\S|\\S.*\\S", "nullable": false },
    "Price": { "jx:type": "string", "pattern": "\\$\\d+\\.\\d{2}", "nullable": false },
    "Manufacturer": { "jx:type": "string", "pattern": "\\S|\\S.*\\S", "nullable": false },
    "InStock": { "jx:type": "boolean", "nullable": false} } },

  "product1": { "jx:type": "object", "extends": "product", "properties": {
    "Version": { "jx:type": "string", "pattern": "v1", "nullable": false } } }
}
```

###### **JSDx**

```xml
<schema
  xmlns="http://www.jsonx.org/schema-0.3.xsd"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.jsonx.org/schema-0.3.xsd http://www.jsonx.org/schema.xsd">

  <object name="product" abstract="true">
    <property name="CatalogueID" xsi:type="number" range="[1,]" scale="0" nullable="false"/>
    <property name="Name" xsi:type="string" pattern="\S|\S.*\S" nullable="false"/>
    <property name="Price" xsi:type="string" pattern="\$\d+\.\d{2}" nullable="false"/>
    <property name="Manufacturer" xsi:type="string" pattern="\S|\S.*\S" nullable="false"/>
    <property name="InStock" xsi:type="boolean" nullable="false"/>
  </object>

  <object name="product1" extends="product">
    <property name="Version" xsi:type="string" pattern="v1" nullable="false"/>
  </object>

</schema>
```

<!-- tabs:end -->

<sub>_**Note:** The [Converter][#converter] utility automatically converts between <ins>JSD</ins> and <ins>JSDx</ins>._</sub>

All actors -- **Producer**, **Consumer1**, and **Consumer2** -- agree on the contract, and implement and integrate the protocol into their systems. To assert receipt of contract-compliant documents, all actors use the contract definition to automatically validate received and sent messages.

After many months of running in production, **Consumer2** issues a request to the **Producer** to provide additional information in the response. Specifically, **Consumer2** requests for the addition of another field in the JSON response:

```diff
{
- "Version": "v1.0",
+ "Version": "v2.0",
  "CatalogueID": <number>,
  "Name": <string>,
  "Price": <string>,
  "Manufacturer": <string>,
  "InStock": <boolean>,
+ "Description": <string>
}
```

To satisfy **Consumer2**'s request, the contract is updated to support version **v2** of the **Response**:

<!-- tabs:start -->

###### **JSD**

```diff
{
  "jx:ns": "http://www.jsonx.org/schema-0.3.jsd",
  "jx:schemaLocation": "http://www.jsonx.org/schema-0.3.jsd http://www.jsonx.org/schema.jsd",

  "product": { "jx:type": "object", "abstract": true, "properties": {
    "CatalogueID": { "jx:type": "number", "range": "[1,]", "scale": 0, "nullable": false},
    "Name": { "jx:type": "string", "pattern": "\\S|\\S.*\\S", "nullable": false },
    "Price": { "jx:type": "string", "pattern": "\\$\\d+\\.\\d{2}", "nullable": false },
    "Manufacturer": { "jx:type": "string", "pattern": "\\S|\\S.*\\S", "nullable": false },
    "InStock": { "jx:type": "boolean", "nullable": false} } },

  "product1": { "jx:type": "object", "extends": "product", "properties": {
    "Version": { "jx:type": "string", "pattern": "v1", "nullable": false } } }

+ "product2": { "jx:type": "object", "extends": "product", "properties": {
+   "Version": { "jx:type": "string", "pattern": "v2", "nullable": false },
+   "Description": { "jx:type": "string", "pattern": "\\S|\\S.*\\S", "nullable": false } } }
}
```

###### **JSDx**

```diff
<schema
  xmlns="http://www.jsonx.org/schema-0.3.xsd"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.jsonx.org/schema-0.3.xsd http://www.jsonx.org/schema.xsd">

  <object name="product" abstract="true">
    <property name="CatalogueID" xsi:type="number" range="[1,]" scale="0" nullable="false"/>
    <property name="Name" xsi:type="string" pattern="\S|\S.*\S" nullable="false"/>
    <property name="Price" xsi:type="string" pattern="\$\d+\.\d{2}" nullable="false"/>
    <property name="Manufacturer" xsi:type="string" pattern="\S|\S.*\S" nullable="false"/>
    <property name="InStock" xsi:type="boolean" nullable="false"/>
  </object>

  <object name="product1" extends="product">
    <property name="Version" xsi:type="string" pattern="v1" nullable="false"/>
  </object>

+ <object name="product2" extends="product">
+   <property name="Version" xsi:type="string" pattern="v2" nullable="false"/>
+   <property name="Description" xsi:type="string" pattern="\S|\S.*\S" nullable="false"/>
+ </object>

</schema>
```

<!-- tabs:end -->

<sub>_**Note:** The [Converter][#converter] utility automatically converts between <ins>JSD</ins> and <ins>JSDx</ins>._</sub>

With this approach, the **v2** evolution of the contract satisfies **Customer2**. And, since the contract also retains support for **v1**, integration with **Customer1** is unaffected.

_For the application code, see **[<ins>Sample: Consumer Driven Contracts</ins>][sample-cdc]**._

## <b>3</b> <ins>JSON Schema Definition Language</ins>

Describes JSON documents using schema components to constrain and document the meaning, usage and relationships of their constituent parts: value types and their content.

_For a detailed specification of the <ins>schema language</ins>, see **[<ins>JSON Schema Definition Language</ins>][schema]**._

## <b>4</b> <ins>JSONx Framework for Java</ins>

Provides a reference implementation of a processor, validator, and binding API for the [<ins>JSON Schema Definition Language (JSD)</ins>][schema]. The framework also provides a collection of <ins>structural</ins> and <ins>functional</ins> patterns intended to help developers work with JSON documents.

_For a detailed specification of the <ins>JSONx Framework for Java</ins> and its modules, see **[<ins>JSONx Framework for Java</ins>][java]**._

## <b>5</b> Contributing

Pull requests are welcome. For major changes, please [open an issue](../../issues) first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## <b>6</b> License

This project is licensed under the MIT License - see the [LICENSE.txt](LICENSE.txt) file for details.

[#invoice-example]: #43-getting-started
[#jsd]: #3-json-schema-definition-language

[cdc]: http://martinfowler.com/articles/consumerDrivenContracts.html
[java]: ../../../java
[oxygenxml]: https://www.oxygenxml.com/xml_editor/download_oxygenxml_editor.html
[sample-cdc]: ../../../java/tree/master/sample/cdc
[schema]: ../../../schema
[xmlschema]: http://www.w3.org/2001/XMLSchema