---
id: t6unaqxqmwpxmi5fz529xis
title: XML Parsers
desc: ''
updated: 1672919295531
created: 1672404390746
---
JAXB is *Java Architecture for XML Binding* and allows a fast and convenient way to **marshal** (write) Java Objects to XML and **unmarshal** (read) XML into objects. 
Supports a binding framework that maps XML elements and attributes to Java fields and properties through the use of Java annotations.
It is a maven plugin.

Through the use of Java annotations for augmenting generated classes with aidditonal info it allows preparation of the classes for JAXB runtime.
First on the top level class (or classes) you need to add an `@XmlRootElement` with the optional of (name ="") which defaults as the name.
You can set the order of the properties (or props) with `@XmlType(propOrder ={"<prop1>", "<prop2>",..., "<propn>"})`.
`@XmlAttribute` and `@XmlElement` are different respectively as an attribute is contained in an element i.e. `<book id="420">` and 
```
<book>
    <id>420</id>
</book>
```
And `@XmlTransient` is for annotation of fields we don't want included in the XML.

You need setter methods for all of the classes associated with the XML file, otherwise it won't get populated correctly.
```Java
@XmlRootElement(name = "book")
@XmlType(propOrder = { "id", "name", "date" })
public class Book {
    private Long id;
    private String name;
    private String author;
    private Date date;

    @XmlAttribute
    public void setId(Long id) {
        this.id = id;
    }

    @XmlElement(name = "title")
    public void setName(String name) {
        this.name = name;
    }

    @XmlTransient
    public void setAuthor(String author) {
        this.author = author;
    }
    
    // constructor, getters and setters
}
```
It's extremely simple to marshal and unmarshal one object with children, but if you want to be able to effectively do it for potential lists of objects it becomes slightly complex.
Thinking about the PriceRecord, PriceBand (contained in record) 
> JAXB:<br>
[[JAXB intro | https://www.javacodegeeks.com/2013/02/jaxb-tutorial-getting-started.html]] <br>
[[Tutorial around JAXB | https://www.vogella.com/tutorials/JAXB/article.html#jaxb]]<br>
[[Another tutorial showing multiple objects | https://howtodoinjava.com/jaxb/jaxb-exmaple-marshalling-and-unmarshalling-list-or-set-of-objects/]]
<br>

> STaX:<br>
[[STaX writing XML | https://mkyong.com/java/how-to-write-xml-file-in-java-stax-writer/]]
<br>
[[STaX reading XML | https://mkyong.com/java/how-to-read-xml-file-in-java-stax-parser/]]
<br>
[[STaX Oracle Docs | https://docs.oracle.com/javase/tutorial/jaxp/stax/why.html]]

#### Dom4j
- XPath stands for XML Path Language
- XPath uses "path like" syntax to identifyand navigate nodes in an XML document
- XPath contains over 200 built-in functions
- XPath is a major element in the XSLT standard and a W3C recommendation