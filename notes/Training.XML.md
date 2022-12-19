---
id: ffa2xpjmkf8e9it8ka580le
title: XML
desc: 'Looking at the parsing of XML files'
updated: 1671187756687
created: 1671101173103
---
> <H2>Learning objectives</H2>
- What is XML
- What is XSD
- How to use JAXB to generate XSD files
- How to use *StAX, Dom4j, and JAXB* to parse XML files

## XML
XML is the shorthand for *eXtensible Markup Language* and is a markup language like HTML with the core design purpose of **storage** and **transportation** of data, it was also designed to be self-descriptive.

XML does ***not*** do anything!
It is simply information wrapped in tags, software must be written to send, receive, store or display it.

The difference between XML and HTML, XML was designed to *carry data, focusing on what data is* whereas HTML was designed to *display data, focusing on how data looks*.
XML tags aren't predefined like HTML tags are.
With XML, the author defines the tags and the document structure.

### XSD
XML Schema describes the structure of an XML document, XSD stands for *XML Schema Definition*.
Its purpose is defining the legal building blocks of an XML document:
- elements and attributes that can appear in a document
- the number of (and order of) child elements
- data types for elements and attributes
- default and fixed values for elements and attributes

Its useful for describing allowable content, validating correctness, defining facets (*restrictions on data*) and data patterns (*data formats*) and converting between data types.

### JAXB
JAXB is *Java Architecture for XML Binding*, it provides a fast and convenient way to **marshal** (write) Java objects into XML and **unmarshal** (read) XML into objects, supports a binding framework that maps XML elements and attributes to Java fields and properties using Java annotations.

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
Above sows a simple Java object to illustrate marshalling and unmarshalling. It contains these annotations:
- `@XmlRootElement`: name of root XML element, derived from class name, can also specify the name of the root element using the name attribute.
- `@XmlType`: define order in which fields are written in XML file
- `@XmlElement`: define actual XML element name that will be used
- `@XmLAttribute`: define id field is mapped as attribute and not an element
- `@XmlTransient`: annotate fields that we don't want included in XML

The *Marshaller* uses UTF-8 encoding when generating XML data, next we'll see how to generate XML files from Java objects.
```java
public void marshal() throws JAXBException, IOException {
    Book book = new Book();
    book.setId(1L);
    book.setName("Book1");
    book.setAuthor("Author1");
    book.setDate(new Date());

    JAXBContext context = JAXBContext.newInstance(Book.class);
    Marshaller mar= context.createMarshaller();
    mar.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE);
    mar.marshal(book, new File("./book.xml"));
}
```
The `javax.xml.bind.JAXBContext` class provides entry point to JAXB API. By default, JAXB doesn't format the XML document, saving space and prevents any whitespace accidentally interpreted as significant.

To have JAXB format the output, we set `Marshaller.JAXB_FORMATTED_OUTPUT` property to *true*. Marshal method uses object and output file to store generated XML as parameters.

Running the above code allows us to check the result in `book.xml`
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="1">
    <title>Book1</title>
    <date>2016-11-12T11:25:12.227+07:00</date>
</book>
```

To use the *Unmarshaller*:
```java
public Book unmarshall() throws JAXBException, IOException {
    JAXBContext context = JAXBContext.newInstance(Book.class);
    return (Book) context.createUnmarshaller()
      .unmarshal(new FileReader("./book.xml"));
}
```

If handling complex types that may not be directly available in JAXB we can write an adapter to show how to manage specific types.
Using the override on marshal, unmarshal and extending the XmlAdapter you can manage these types.
```java
public class DateAdapter extends XmlAdapter<String, Date> {

    private static final ThreadLocal<DateFormat> dateFormat 
      = new ThreadLocal<DateFormat>() {

        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    @Override
    public Date unmarshal(String v) throws Exception {
        return dateFormat.get().parse(v);
    }

    @Override
    public String marshal(Date v) throws Exception {
        return dateFormat.get().format(v);
    }
}

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

    @XmlTransient
    public void setAuthor(String author) {
        this.author = author;
    }

    @XmlElement(name = "title")
    public void setName(String name) {
        this.name = name;
    }

    @XmlJavaTypeAdapter(DateAdapter.class)
    public void setDate(Date date) {
        this.date = date;
    }
}
```