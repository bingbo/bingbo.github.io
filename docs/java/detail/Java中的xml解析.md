## DOM解析

DOM是用与平台和语言无关的方式表示XML文档的官方W3C标准。DOM是以层次结构组织的节点或信息片断的集合。这个层次结构允许开发人员在树中寻找特定信息。分析该结构通常需要加载整个文档和构造层次结构，然后才能做任何工作。由于它是基于信息层次的，因而DOM被认为是基于树或基于对象的。

 【优点】

 * 允许应用程序对数据和结构做出更改。
 * 访问是双向的，可以在任何时候在树中上下导航，获取和操作任意部分的数据。

 【缺点】

 * 通常需要加载整个XML文档来构造层次结构，消耗资源大。

```java
/**
 DOM是用与平台和语言无关的方式表示XML文档的官方W3C标准。DOM是以层次结构组织的节点或信息片断的集合。这个层次结构允许开发人员在树中寻找特定信息。分析该结构通常需要加载整个文档和构造层次结构，然后才能做任何工作。由于它是基于信息层次的，因而DOM被认为是基于树或基于对象的。

 【优点】
 ①允许应用程序对数据和结构做出更改。
 ②访问是双向的，可以在任何时候在树中上下导航，获取和操作任意部分的数据。
 【缺点】
 ①通常需要加载整个XML文档来构造层次结构，消耗资源大。* Created by bing on 2017/1/4.
 */
public class DOMXML {

    public static void main(String[] args) throws Exception{
        DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = builderFactory.newDocumentBuilder();
        Document document = builder.parse(new File("src/main/java/com/ibingbo/xml/books.xml"));

        //get root element
        Element rootElement = document.getDocumentElement();
//        NodeList children = rootElement.getChildNodes();
//        for (int i=0;i<children.getLength();i++) {
//            Node node = children.item(i);
//            if (node.getNodeType() == Node.ELEMENT_NODE) {
//                Element child = (Element) node;
//                System.out.println(child.getTagName());
//                System.out.println(" id: "+child.getAttribute("id"));
//                if (child.hasChildNodes()) {
//                    NodeList nodes=child.getChildNodes();
//                    for (int j=0;j<nodes.getLength();j++) {
//                        Node node1 = nodes.item(j);
//                        if (node1.getNodeType() == Node.ELEMENT_NODE) {
//                            Element element = (Element)node1;
//                            System.out.println(" "+element.getTagName() + ": "+element.getTextContent());
//                        }
//                    }
//                }
//            }
//
//        }
        parse(rootElement);

    }

    private static void parse(Element root) {
        System.out.println(root.getTagName());
        System.out.println(root.getTextContent());
        if (root.hasAttributes()) {
            NamedNodeMap map = root.getAttributes();
            for (int i=0;i<map.getLength();i++) {
                Node attribute = map.item(i);
                if (attribute.getNodeType() == Node.ATTRIBUTE_NODE) {
                    System.out.println(attribute.getNodeName()+" : "+attribute.getTextContent());
                }
            }
        }
        if (root.hasChildNodes()) {
            NodeList nodeList = root.getChildNodes();
            for (int j=0;j<nodeList.getLength();j++) {
                Node node = nodeList.item(j);
                if (node.getNodeType() == Node.ELEMENT_NODE) {
                    parse((Element)node);
                }
            }
        }
    }
}
```

## SAX解析

* SAX处理的优点非常类似于流媒体的优点。分析能够立即开始，而不是等待所有的数据被处理。而且，由于应用程序只是在读取数据时检查数据，因此不需要将数据存储在内存中。这对于大型文档来说是个巨大的优点。事实上，应用程序甚至不必解析整个文档；它可以在某个条件得到满足时停止解析。一般来说，SAX还比它的替代者DOM快许多。

 选择DOM还是选择SAX？ 对于需要自己编写代码来处理XML文档的开发人员来说， 选择DOM还是SAX解析模型是一个非常重要的设计决策。 DOM采用建立树形结构的方式访问XML文档，而SAX采用的是事件模型。

 DOM解析器把XML文档转化为一个包含其内容的树，并可以对树进行遍历。用DOM解析模型的优点是编程容易，开发人员只需要调用建树的指令，然后利用navigation APIs访问所需的树节点来完成任务。可以很容易的添加和修改树中的元素。然而由于使用DOM解析器的时候需要处理整个XML文档，所以对性能和内存的要求比较高，尤其是遇到很大的XML文件的时候。由于它的遍历能力，DOM解析器常用于XML文档需要频繁的改变的服务中。

 SAX解析器采用了基于事件的模型，它在解析XML文档的时候可以触发一系列的事件，当发现给定的tag的时候，它可以激活一个回调方法，告诉该方法制定的标签已经找到。SAX对内存的要求通常会比较低，因为它让开发人员自己来决定所要处理的tag.特别是当开发人员只需要处理文档中所包含的部分数据时，SAX这种扩展能力得到了更好的体现。但用SAX解析器的时候编码工作会比较困难，而且很难同时访问同一个文档中的多处不同数据。

【优势】

* 不需要等待所有数据都被处理，分析就能立即开始 。
* 只在读取数据时检查数据，不需要保存在内存中。
* 可以在某个条件得到满足时停止解析，不必解析整个文档。
* 效率和性能较高，能解析大于系统内存的文档。

【缺点】
* 需要应用程序自己负责TAG的处理逻辑（例如维护父/子关系等），文档越复杂程序就越复杂。
* 单向导航，无法定位文档层次，很难同时访问同一文档的不同部分数据，不支持XPath。

如果XML文档较大且不考虑移植性问题建议采用DOM4J；如果XML文档较小则建议采用JDOM；如果需要及时处理而不需要保存数据则考虑SAX。

```java
/**
 * SAX处理的优点非常类似于流媒体的优点。分析能够立即开始，而不是等待所有的数据被处理。而且，由于应用程序只是在读取数据时检查数据，因此不需要将数据存储在内存中。这对于大型文档来说是个巨大的优点。事实上，应用程序甚至不必解析整个文档；它可以在某个条件得到满足时停止解析。一般来说，SAX还比它的替代者DOM快许多。

 选择DOM还是选择SAX？ 对于需要自己编写代码来处理XML文档的开发人员来说， 选择DOM还是SAX解析模型是一个非常重要的设计决策。 DOM采用建立树形结构的方式访问XML文档，而SAX采用的是事件模型。

 DOM解析器把XML文档转化为一个包含其内容的树，并可以对树进行遍历。用DOM解析模型的优点是编程容易，开发人员只需要调用建树的指令，然后利用navigation APIs访问所需的树节点来完成任务。可以很容易的添加和修改树中的元素。然而由于使用DOM解析器的时候需要处理整个XML文档，所以对性能和内存的要求比较高，尤其是遇到很大的XML文件的时候。由于它的遍历能力，DOM解析器常用于XML文档需要频繁的改变的服务中。

 SAX解析器采用了基于事件的模型，它在解析XML文档的时候可以触发一系列的事件，当发现给定的tag的时候，它可以激活一个回调方法，告诉该方法制定的标签已经找到。SAX对内存的要求通常会比较低，因为它让开发人员自己来决定所要处理的tag.特别是当开发人员只需要处理文档中所包含的部分数据时，SAX这种扩展能力得到了更好的体现。但用SAX解析器的时候编码工作会比较困难，而且很难同时访问同一个文档中的多处不同数据。

 【优势】
 ①不需要等待所有数据都被处理，分析就能立即开始。
 ②只在读取数据时检查数据，不需要保存在内存中。
 ③可以在某个条件得到满足时停止解析，不必解析整个文档。
 ④效率和性能较高，能解析大于系统内存的文档。

 【缺点】
 ①需要应用程序自己负责TAG的处理逻辑（例如维护父/子关系等），文档越复杂程序就越复杂。
 ②单向导航，无法定位文档层次，很难同时访问同一文档的不同部分数据，不支持XPath。


 如果XML文档较大且不考虑移植性问题建议采用DOM4J；如果XML文档较小则建议采用JDOM；如果需要及时处理而不需要保存数据则考虑SAX。
 * Created by bing on 2017/1/4.
 */
public class SAXXML {

    public static void main(String[] args) throws Exception {
        XMLReader xmlReader = XMLReaderFactory.createXMLReader();
        xmlReader.setContentHandler(new BookHandler());
        xmlReader.parse("src/main/java/com/ibingbo/xml/books.xml");
    }

    private static class BookHandler extends DefaultHandler{

        boolean hasAttribute = false;
        Attributes attributes = null;
        @Override
        public void startDocument() throws SAXException {
            System.out.println("start document ...");
        }

        @Override
        public void endDocument() throws SAXException {
            System.out.println("end document ...");
        }

        @Override
        public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
            System.out.println(qName);
            if (attributes.getLength() > 0) {
                for (int i=0;i<attributes.getLength();i++) {
                    System.out.println(attributes.getQName(i) + " : " + attributes.getValue(i));
                }
            }
        }

        @Override
        public void endElement(String uri, String localName, String qName) throws SAXException {

        }

        @Override
        public void characters(char[] ch, int start, int length) throws SAXException {
            System.out.println(new String(ch, start, length));
        }
    }

}
```

## STAX解析

StAX把重点放在流上，StAX使应用程序能够把 XML 作为一个事件流来处理；其实SAX方式也是基于事件流的XML处理方法，但二者不同之处在于，SAX是基于观察者模式，我们需要提供事件处理程序并注册到解析器，解析器在指定事件发生时回调我们提供的程序；而StAX允许我们的程序把事件逐个”拉“出来，这样StAX就有更大的灵活性，对于我们不感兴趣的事件就没有必要将其”拉“出来处理。
 StAX提供了两套API用来处理XML，分别提供了不同程度的抽象。基于指针的 API 把 XML 作为一个标记（或事件）流来处理；应用程序可以检查解析器的状态，获得解析的上一个标记的信息，然后再处理下一个标记，依此类推。这是一种低层 API，尽管效率高，但是没有提供底层 XML 结构的抽象。基于迭代器的 API 把 XML 作为一系列事件对象来处理。应用程序只需要确定解析事件的类型，将其转换成对应的具体类型，然后利用其方法获得属于该事件的信息
 
```java
/**
 * StAX把重点放在流上，StAX使应用程序能够把 XML 作为一个事件流来处理；其实SAX方式也是基于事件流的XML处理方法，但二者不同之处在于，SAX是基于观察者模式，我们需要提供事件处理程序并注册到解析器，解析器在指定事件发生时回调我们提供的程序；而StAX允许我们的程序把事件逐个”拉“出来，这样StAX就有更大的灵活性，对于我们不感兴趣的事件就没有必要将其”拉“出来处理。
 StAX提供了两套API用来处理XML，分别提供了不同程度的抽象。基于指针的 API 把 XML 作为一个标记（或事件）流来处理；应用程序可以检查解析器的状态，获得解析的上一个标记的信息，然后再处理下一个标记，依此类推。这是一种低层 API，尽管效率高，但是没有提供底层 XML 结构的抽象。基于迭代器的 API 把 XML 作为一系列事件对象来处理。应用程序只需要确定解析事件的类型，将其转换成对应的具体类型，然后利用其方法获得属于该事件的信息
 * Created by bing on 2017/1/4.
 */
public class STAXXML {

    public static void main(String[] args) throws Exception {
        XMLInputFactory xmlInputFactory = XMLInputFactory.newInstance();
        xmlInputFactory.setEventAllocator(new XMLEventAllocatorImpl());
//        XMLStreamReader xmlStreamReader = XMLStreamReaderFactory.create("books.xml",new FileInputStream("src/main/java/com/ibingbo/xml/books.xml"),false);
        //基于流、游标的读取
        XMLStreamReader xmlStreamReader = xmlInputFactory.createXMLStreamReader(new FileInputStream("src/main/java/com/ibingbo/xml/books.xml"));
        //parse(xmlStreamReader);

        //基于事件的读取
        XMLEventReader eventReader = xmlInputFactory.createXMLEventReader(new FileInputStream("src/main/java/com/ibingbo/xml/books.xml"));
        //可以封装过滤器
        eventReader = xmlInputFactory.createFilteredReader(eventReader, new EventFilter() {
            @Override
            public boolean accept(XMLEvent event) {
                return true;
            }
        });
        parse(eventReader);
    }

    private static void parse(XMLEventReader eventReader) throws Exception {
        while (eventReader.hasNext()) {
            XMLEvent event = eventReader.nextEvent();
            if (event.isStartDocument()) {
                System.out.println("start document ...");
                System.out.println(event.getSchemaType());
                System.out.println(event.isNamespace());
            } else if (event.isStartElement()) {
                System.out.println(event.asStartElement().getName());
                Iterator<Attribute> iterator = event.asStartElement().getAttributes();
                if (null != iterator) {
                    while (iterator.hasNext()) {
                        Attribute attribute = iterator.next();
                        System.out.println(attribute.getName() + " : " + attribute.getValue());
                    }
                }
            } else if (event.isAttribute()) {
                System.out.println(event.asCharacters().getData());
            } else if (event.isCharacters()) {
                System.out.println(event.asCharacters().getData());
            }
        }
    }
    private static void parse(XMLStreamReader xmlStreamReader) throws Exception {
        int eventType = xmlStreamReader.getEventType();
        while (xmlStreamReader.hasNext()) {
            eventType = xmlStreamReader.next();
            switch (eventType) {
                case XMLStreamConstants.ATTRIBUTE:

                    break;
                case XMLStreamConstants.START_DOCUMENT:
                    System.out.print("start document ..");
                    System.out.println(xmlStreamReader.getEncoding());
                    System.out.println(xmlStreamReader.getVersion());
                    System.out.println(xmlStreamReader.isStandalone());
                    break;
                case XMLStreamConstants.START_ELEMENT:
                    System.out.println(xmlStreamReader.getLocalName());
                    int count = xmlStreamReader.getAttributeCount();
                    if (count > 0) {
                        for (int i = 0; i < count; i++) {
                            System.out.println(xmlStreamReader.getAttributeLocalName(i) + " : " + xmlStreamReader.getAttributeValue(i));
                        }
                    }
                    break;
                case XMLStreamConstants.CDATA:
                case XMLStreamConstants.COMMENT:
                case XMLStreamConstants.CHARACTERS:
                case XMLStreamConstants.SPACE:
                    System.out.println(xmlStreamReader.getText());
                    break;
                default:
                    break;
            }

        }
    }


}
```

## Digester解析

Digester 就是一种用来把一个 XML 转化为一个与该 XML 结构类似的 JavaBean。你可以把 XML 根元素想象成一个 JavaBean， 该根元素的 attribute 就是这个 JavaBean 的各种 Field，当该根元素有其他子 tag 时，又要把这个子 tag 想象成一个个新的 XML，将其视为一个新的 JavaBean， 并作为一个 Field 加入到父 Bean 当中，然后以此类推，通过循环的方式将整个 XML 进行解析
 
```java
/**
 * Digester 就是一种用来把一个 XML 转化为一个与该 XML 结构类似的 JavaBean。你可以把 XML 根元素想象成一个 JavaBean， 该根元素的 attribute 就是这个 JavaBean 的各种 Field，当该根元素有其他子 tag 时，又要把这个子 tag 想象成一个个新的 XML，将其视为一个新的 JavaBean， 并作为一个 Field 加入到父 Bean 当中，然后以此类推，通过循环的方式将整个 XML 进行解析
 * Created by bing on 2017/1/4.
 */
public class DigesterXML {

    public static void main(String[] args) throws Exception {
        File file = new File("src/main/java/com/ibingbo/xml/books.xml");
        Digester digester = new Digester();
        //创建Books对象
        digester.addObjectCreate("books", Books.class);
        //添加Books对象的一些属性
        digester.addSetProperties("books");

        //创建Book对象
        digester.addObjectCreate("books/book", Book.class);
        //添加Book对象的一些属性
        digester.addSetProperties("books/book");

        //设置Book对象的指定属性的值
        digester.addBeanPropertySetter("books/book/title", "title");
        digester.addBeanPropertySetter("books/book/author", "author");

        //下一个Book对象，解析并添加到Books中，通过addBook方法
        digester.addSetNext("books/book", "addBook", "com.xml.Book");

        Books books = (Books) digester.parse(file);

        System.out.println(books);
    }
}
```

```java
public class Books {

    private List<Book> books = new ArrayList<>();

    public List<Book> getBooks() {
        return books;
    }

    public void setBooks(List<Book> books) {
        this.books = books;
    }

    public void addBook(Book book) {
        this.books.add(book);
    }

    @Override
    public String toString() {
        return "Books{" +
                "books=" + books +
                '}';
    }
}

public class Book {
    private String id;
    private String title;
    private String author;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}
```
