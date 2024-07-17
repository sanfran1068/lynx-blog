---
layout: post
title: "Jena学习笔记"
date: 2015-09-21 12:00:00
categories: 
- Document
tags:
- JenaAPI
- API
- Ontology
- 基础知识
---

#### Jena初步了解

JenaAPI的框架如下图：
![](https://img.alicdn.com/imgextra/i3/O1CN01r61JlO21WqWYrTMd4_!!6000000006993-2-tps-610-590.png)

<!-- more -->

如上图所示，利用Jena进行语义网处理应用的开发有通过HTTP和Servlet响应的B/S模式，还有通过JenaAPI进行Java编程来处理本体数据。

- RDF API：Jena提供了对RDF文件的创建、读/写以及一些基本操作的接口（方法）；
- SPARQL API：Jena提供一套对本体的三元组的query机制，类似于SQL语言对于数据库的操作，SPARQL对于本体的查询也可以通过简单的查询语句达到目的，这也为B/S模式的本体展示和交互提供了基础；
- Ontology API：
- Fuseki

#### RDF API

##### 名词解释
- Blank Node：一阶逻辑中表示一个无URI的resource
- Literal：可作为一个property的值（value）
- Object：the resource or literal pointed to by the arc（箭头指向的一端）
- Subject：the resource from which the arc leaves（箭头离开的一端）
- Triple：
  - Predicate：三元组中的属性部分（关系）（the property that labels the arc）
  - Property：一个resource的属性，指三元组中的关系
  - Resouce：实体，通常由URI命名的资源

  Statement：指一个fact，即一个三元组（Each arc in an RDF Model）

##### 对RDF的操作（JavaAPI）

- Create graph    

```java
// some definitions
static String personURI = "http://somewhere/JohnSmith";
static String fullName = "John Smith";

// create an empty Model
Model model = ModelFactory.createDefaultModel();

// create the resource
Resource johnSmith = model.createResource(personURI);

// add the property
 johnSmith.addProperty(VCARD.FN, fullName);
```

- Create resource and add the property

```java
//Resouce example
Resource johnSmith = model.createResource(personURI).addProperty(VCARD.FN, fullName);
```

```java
// some definitions
String personURI    = "http://somewhere/JohnSmith";
String givenName    = "John";
String familyName   = "Smith";
String fullName     = givenName + " " + familyName;

// create an empty Model
Model model = ModelFactory.createDefaultModel();

// create the resource
// and add the properties cascading style
Resource johnSmith
  = model.createResource(personURI)
         .addProperty(VCARD.FN, fullName)
         .addProperty(VCARD.N,
                      model.createResource()
                           .addProperty(VCARD.Given, givenName)
                           .addProperty(VCARD.Family, familyName));
```

- Manipulate the stmt

```java
// list the statements in the Model
StmtIterator iter = model.listStatements();

// print out the predicate, subject and object of each statement
while (iter.hasNext()) {
    Statement stmt      = iter.nextStatement();  // get next statement
    Resource  subject   = stmt.getSubject();     // get the subject
    Property  predicate = stmt.getPredicate();   // get the predicate
    RDFNode   object    = stmt.getObject();      // get the object

    System.out.print(subject.toString());
    System.out.print(" " + predicate.toString() + " ");
    if (object instanceof Resource) {
       System.out.print(object.toString());
    } else {
        // object is a literal
        System.out.print(" \"" + object.toString() + "\"");
    }

    System.out.println(" .");
}
```

由于object可以是Resource或者Literal，所以object的类型为RDFNode。

- Writing RDF

```java
// now write the model in XML form to a file
model.write(System.out);
model.write(System.out, "RDF/XML-ABBREV"); // write the model in XML form to a file
model.write(System.out, "N-TRIPLES"); // write the model in N-TRIPLES form to a file
```

- Reading RDF

```java
// create an empty model
 Model model = ModelFactory.createDefaultModel();

 // use the FileManager to find the input file
 InputStream in = FileManager.get().open( inputFileName );
if (in == null) {
    throw new IllegalArgumentException(
                                 "File: " + inputFileName + " not found");
}

// read the RDF/XML file
model.read(in, null);

// write it to standard out
model.write(System.out);
```

#### SPARQL API

需要保存在.rq文件中进行使用。
- Basic query：

```SPARQL
SELECT ?xWHERE{ ?x<http://www.w3.org/2001/vcard-rdf/3.0#FN>  "JohnSmith" }
```
其中<>中为URI，“”中为literal，？x为查询变量

- Basic pattern

```SPARQL
WHERE
  { ?y  <http://www.w3.org/2001/vcard-rdf/3.0#Family>  "Smith" .
    ?y  <http://www.w3.org/2001/vcard-rdf/3.0#Given>  ?givenName .
  }
```

basic patterns：是一组三元组的模式；
prefix and blanknodes：

```SPARQL
PREFIX vcard:      <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?y ?givenName
WHERE
 { ?y vcard:Family "Smith" .
   ?y vcard:Given  ?givenName .
 }
 ```

- Filters

```SPARQL

PREFIX vcard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?g
WHERE
{ ?y vcard:Given ?g .
  FILTER regex(?g, "r", "i") }
```
上面的加粗为FILTER，其中最后一个“”中的i表示case-insensitive

- Optionals（表示如果有这个属性那么就显示在结果中，如果没有就不显示）

```SPARQL
PREFIX info:    <http://somewhere/peopleInfo#>
PREFIX vcard:   <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name ?age
WHERE
{
    ?person vcard:FN  ?name .
    OPTIONAL { ?person info:age ?age }
}
```

optionals with filters：

```SPARQL
PREFIX info:        <http://somewhere/peopleInfo#>
PREFIX vcard:      <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name ?age
WHERE
{
    ?person vcard:FN  ?name .
    OPTIONAL { ?person info:age ?age . FILTER ( ?age > 24 ) }
}
```

- Unions：

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX vCard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name
WHERE
{
   { [] foaf:name ?name } UNION { [] vCard:FN ?name }
}
```

相当于FILTER ( ?p = foaf:name || ?p = vCard:FN)
如果是分开资源那么只要分别用不同变量标注即可：

```SPARQL
SELECT ?name1 ?name2
WHERE
{
   { [] foaf:name ?name1 } UNION { [] vCard:FN ?name2 }
}
```

- SPARQL result forms：

  - SELECT – Return a table of results.
  - CONSTRUCT – Return an RDF graph, based on a template in the query.
  - DESCRIBE – Return an RDF graph, based on what the query processor is configured to return.
  - ASK – Ask a boolean query.
