# XML与Xpath
XML是一种具有某种层次结构的文件，Xpath则是解析这种文件的工具
接下来将会解释XML文件的结构和Xpath的基本使用，并且用Java语言进行操作展示。
## XML结构
XML（可扩展标记语言）文件具有一种层次结构，由标签、元素、属性和文本组成
例如：
```
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book category="Fiction">
    <title lang="en">The Great Gatsby</title>
    <author>F. Scott Fitzgerald</author>
    <price>12.99</price>
  </book>
  <book category="Non-Fiction">
    <title lang="en">The Elements of Style</title>
    <author>William Strunk Jr.</author>
    <price>9.99</price>
    <description>
        <![CDATA[This is a <![CDATA[nested]]> CDATA section.]]>
    </description>
  </book>
</bookstore>
```
1、XML声明：通常出现在XML文档的开头，用于声明XML版本和字符编码。示例中的声明是
```
<?xml version="1.0" encoding="UTF-8"?>。
```
2、根元素：整个XML文档的最顶层元素，这里是<bookstore>。XML文档只能有一个根元素，所有其他元素都是其子元素。
3、元素：元素是XML文档的构建块，它们由开始标签和结束标签包围。例如，<book> 和 </book> 是一对开始标签和结束标签，定义了一个<book>元素。
4、属性：元素可以包含属性，属性提供关于元素的额外信息。在示例中，category 和 lang 是<book>元素的属性。
5、文本：在元素的开始标签和结束标签之间的文本内容，例如，The Great Gatsby 和 F. Scott Fitzgerald 是<title>和<author>元素的文本内容
XML文件的结构可以根据需求进行嵌套和扩展，以表示各种类型的数据和信息。这种层次结构和标记的语法使XML成为一种灵活的数据表示方式，广泛用于数据交换和存储，特别是在Web服务、配置文件和数据传输方面。
6、特殊-CDATA
CDATA（Character Data）部分是XML文档中的一个特殊部分，用于包含不应由XML解析器解释的文本数据
CDATA部分通常用于包含原始文本、脚本、代码片段或其他内容，这些内容可能包含特殊字符或标签，而不希望被解析为XML元素。
CDATA部分以<![CDATA[开始，以]]>结束，如下所示：
```
<description>
        <![CDATA[This is a <![CDATA[nested]]> CDATA section.]]>
</description>
```
7、特殊--DTD
DTD（Document Type Definition）是一种用于定义XML文档结构和验证其有效性的规范。DTD定义了XML文档的元素、属性、实体引用和它们之间的关系，以确保文档遵循指定的结构和规则。以下是关于DTD的一些重要信息：
元素定义：DTD定义了XML文档中可以包含哪些元素以及它们的结构。这包括元素的名称、内容类型（如文本、元素序列、混合内容等）以及它们的层次结构。
1）、属性定义：DTD也可以定义每个元素可以具有的属性、属性的数据类型，以及它们的默认值。属性可以用于提供有关元素的额外信息。
2）、实体引用：DTD支持实体引用，允许在XML文档中使用实体引用来代替常见的字符或文本。例如，&lt; 表示小于号，&gt; 表示大于号。
3）、文档类型声明：在XML文档的开头，可以包括一个文档类型声明，指定要使用的DTD。这有助于验证文档的结构和有效性。
4）、验证：XML解析器可以使用DTD来验证XML文档是否符合规定的结构和规则。如果文档与DTD不一致，解析器会产生错误。
以下是对开头的xml文件的定义
```
<!DOCTYPE bookstore [
  <!ELEMENT bookstore (book+)>
  <!ELEMENT book (title, author, price)>
  <!ELEMENT title (#PCDATA)>
  <!ELEMENT author (#PCDATA)>
  <!ELEMENT price (#PCDATA)>
  <!ATTLIST book category CDATA #REQUIRED>
  <!ATTLIST title lang CDATA #IMPLIED>
]>

```
在上述示例中，DTD定义了一个XML文档，其中包含bookstore元素，该元素包含至少一个book元素。每个book元素具有title、author和price子元素，并且book元素具有一个category属性，title元素具有一个可选的lang属性。
DTD对于确保XML文档的结构和有效性非常有用，但它也有一些限制和不足，包括不支持命名空间和复杂的数据类型。因此，一些应用程序选择使用XML Schema或其他模式语言来定义XML文档的结构和验证。
8、特殊--Schema
XML Schema（XML模式）是一种用于定义XML文档结构、元素、属性和数据类型的规范，它允许更精确和强大的验证和验证XML文档的有效性。与DTD（Document Type Definition）相比，XML Schema提供了更多的功能和灵活性，包括支持命名空间、更复杂的数据类型定义以及更丰富的约束。以下是有关XML Schema的一些关键信息：
1）、结构定义：XML Schema允许您定义XML文档的结构，包括元素、元素的顺序、出现次数和嵌套关系。您可以指定哪些元素是必需的，哪些是可选的，以及它们的嵌套规则。
2）、数据类型：XML Schema定义了可以用于元素和属性值的数据类型，例如字符串、整数、日期等。这允许更详细的数据验证，确保数据的格式和类型正确。
3）、约束：XML Schema支持各种约束，包括最小值、最大值、正则表达式模式、唯一性约束等。这些约束有助于确保文档数据的完整性和一致性。
4）、命名空间支持：XML Schema允许在XML文档中使用命名空间，以确保元素和属性名称的唯一性，并允许更复杂的文档结构。
5）、导入和包含：XML Schema支持模块化，可以将多个XML Schema文件组合在一起，以便更容易管理和维护模式定义。
6）、验证：XML解析器可以使用XML Schema来验证XML文档的有效性。如果文档与模式不匹配，解析器会报告错误。
```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="bookstore">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="book" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="title" type="xs:string"/>
              <xs:element name="author" type="xs:string"/>
              <xs:element name="price" type="xs:decimal"/>
            </xs:sequence>
            <xs:attribute name="category" type="xs:string"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```
在上述示例中，XML Schema定义了一个bookstore元素，它包含了多个book元素，每个book元素具有title、author和price元素，以及一个category属性。这个模式将确保XML文档遵循这一结构，并且price元素的值为十进制数。

## Xpath
XPath（XML路径语言）是一种用于在XML文档中选择元素和节点的查询语言。它提供了一种灵活的方法来定位XML文档中的数据，使您能够根据元素的层次结构、属性、文本内容和其他条件来检索所需的信息。XPath通常与XML文档处理、解析和转换一起使用，以提取数据或执行特定操作。
以下是XPath的一些常见用法和语法：
1、元素选择：您可以使用XPath来选择XML文档中的元素，指定元素的路径。例如，要选择<book>元素，可以使用路径表达式/bookstore/book。
2、属性选择：XPath允许您选择元素的属性。例如，要选择lang属性的值，可以使用路径表达式/bookstore/book/title/@lang。
3、文本内容选择：使用XPath，您可以选择元素的文本内容。例如，要选择<title>元素的文本内容，可以使用路径表达式/bookstore/book/title/text()。
4、通配符：XPath支持通配符，如*用于匹配任何元素。例如，/bookstore/*将选择<book>元素。
5、谓词：XPath允许您使用谓词来进一步筛选元素。例如，/bookstore/book[1]将选择第一个<book>元素。
6、逻辑运算符：您可以在XPath中使用逻辑运算符，如and和or，来组合条件。例如，/bookstore/book[price > 20 and category = 'Fiction']将选择价格大于20且类别为'Fiction'的书。
7、轴：XPath提供了各种轴（如child、parent、ancestor、following-sibling等）来导航文档的层次结构。这使您可以选择相对于当前节点的其他节点。
XPath在多种编程语言中都有相应的库和工具，用于解析和查询XML文档。例如，在Python中，lxml库提供了XPath查询功能。在Java中，您可以使用Java API for XML Processing (JAXP) 来执行XPath查询。

## 扩展--Xlink和XPoint
XLink（XML Linking Language）是一种XML标准，用于定义和表示超链接信息，使其更丰富和复杂。XLink的主要目标是提供更多的灵活性和功能，以支持链接在XML文档中的使用。以下是一些关于XLink的重要信息：
1、超链接类型：XLink引入了不同类型的超链接，包括简单链接（simple link）和扩展链接（extended link）。简单链接允许你创建基本的链接，而扩展链接允许你定义更复杂的链接关系，包括多个资源和其他属性。
2、超链接元素：XLink引入了新的元素来表示超链接信息。其中，<a> 元素用于定义超链接，<locator> 元素用于指定资源的位置，<arc> 元素用于指定链接关系。这些元素可以包含各种属性，以定义链接的性质和行为。
3、超链接属性：XLink引入了一系列属性，如xlink:href（用于指定目标资源的位置）和xlink:type（用于指定链接的类型），这些属性用于描述超链接。
4、命名空间：XLink定义了自己的XML命名空间（通常为http://www.w3.org/1999/xlink），以将XLink的元素和属性与XML文档中的其他元素分开。
XPointer：XLink可以与XPointer一起使用，XPointer是用于定位XML文档中特定部分的标准。这使得可以创建更精确的链接，例如链接到XML文档中的特定段落或元素。
使用场景：XLink可以用于多种用途，包括在XML文档中创建超链接、定义导航菜单、构建复杂的链接结构以及在文档之间建立链接。
尽管XLink提供了更多的灵活性和功能，但它并没有在Web开发中得到广泛采用，因为HTML和其他Web标准通常提供了足够的超链接功能。然而，XLink在特定领域和应用中仍然有其用武之地，特别是在XML文档中需要更复杂链接关系的情况下。
## Xpath解析XML文件
1、在idea中定义xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE bookstore SYSTEM "bookstore.dtd">
<bookstore>
    <book category="Fiction">
        <title lang="en">The Great Gatsby</title>
        <author>F. Scott Fitzgerald</author>
        <price>12.99</price>
    </book>
    <book category="Non-Fiction">
        <title lang="en">The Elements of Style</title>
        <author>William Strunk Jr.</author>
        <price>9.99</price>
        <description>
            <![CDATA[This is a <![CDATA[nested]]> CDATA section.]]>
        </description>
    </book>
</bookstore>
```
同时在当前目录下建立一个bookstore.dtd文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE bookstore [
        <!ELEMENT bookstore (book+)>
        <!ELEMENT book (title, author, price)>
        <!ELEMENT title (#PCDATA)>
        <!ELEMENT author (#PCDATA)>
        <!ELEMENT price (#PCDATA)>
        <!ATTLIST book category CDATA #REQUIRED>
        <!ATTLIST title lang CDATA #IMPLIED>
        ]>

```
需要注意，如果是在idea里建立的，会产生一个错误
URI is not registered (Settings | Languages & Frameworks | Schemas and DTDs)
这时需要手动注册一下这个dtd文件
1）、打开你的IDE，进入 "Settings" 或 "Preferences"。
2）、寻找 "Languages & Frameworks" 或类似的选项。
3）、在 "Languages & Frameworks" 下，选择 "Schemas and DTDs" 或类似的子选项。
4）、在 "Schemas and DTDs" 设置中，你应该看到一个列表，其中包含了已经注册的 URI，以及与之关联的 XML Schema 或 DTD 文件。
如果你的 XML 文档中使用了一个未注册的 URI，你需要将其注册。在设置中，应该有一个选项来添加或注册新的 URI。
5）、通常，你可以点击 "Add" 或 "New" 按钮，然后输入要注册的 URI。
输入 URI 后，你可以为它选择相关联的 XML Schema 文件或 DTD 文件，以便在验证时使用。
确保保存设置。
在URI框中命名为bookstore.dtd，这个需要与xml文件引入的对应
6）、现在，你的IDE应该能够正确识别和验证你的 XML 文档中使用的 URI。
2、选用Xpath工具
在Java中，你可以使用许多不同的库和工具来执行XPath查询和处理XML文档。以下是一些常见的Java XPath工具和库：
1）、Java内置XPath支持：Java标准库提供了对XPath的支持，可以使用javax.xml.xpath包中的类来执行XPath查询。这包括XPathFactory和XPath类。你可以使用XPathFactory来创建一个XPath对象，然后使用它来执行XPath查询。
2）、JDOM：JDOM是一个流行的Java库，用于解析和处理XML文档。它提供了XPath支持，你可以使用org.jdom2.input.SAXBuilder来解析XML文档，然后使用org.jdom2.XPath来执行XPath查询。
3）、DOM4J：DOM4J是另一个流行的Java库，用于处理XML文档。它提供了XPath支持，你可以使用org.dom4j.Document对象的selectNodes()方法执行XPath查询。
4）、Xerces-J：Xerces-J是Apache XML项目的一部分，是一个流行的XML解析器和处理工具包。它支持XPath查询，可以使用org.apache.xpath.XPathAPI来执行查询。
5）、Saxon：Saxon是一个功能强大的XPath和XSLT处理器，它提供了高度优化的XPath引擎，用于XML文档的查询。你可以将Saxon集成到Java应用程序中，并使用其API来执行XPath查询。
6）、XMLUnit：XMLUnit是一个用于测试XML文档的Java库，它包括XPath支持，用于验证和比较XML文档。你可以使用XMLUnit来执行XPath查询以验证XML文档的结构和内


引入依赖
```

        <dependency>
            <groupId>org.jdom</groupId>
            <artifactId>jdom2</artifactId>
            <version>2.0.6</version> <!-- 请使用最新版本 -->
        </dependency>
```
