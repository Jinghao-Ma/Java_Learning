## XML & JSON

[TOC]



### XML

- 可扩展标记语言（eXtensible Markup Language）
- 一种数据表示格式
  - 可以描述非常复杂的数据结构
  - 用于传输和储存数据
- 纯文本，默认UTF-8编码
- 可嵌套，适合表示结构化数据



**XML格式**

- XML声明

- 文档类型定义（DTD，可选）

- 有且仅有一个根元素

- 可以包含若干子元素

- 元素可以包含属性

- 元素必须正确嵌套

- 空元素用`<tag/>`

- 特殊字符用`&???;`表示

  - | 字符 |   表示   |
    | :--: | :------: |
    | `<`  |  `&lt;`  |
    | `>`  |  `&gt;`  |
    | `&`  | `&amp;`  |
    | `“`  | `&quot;` |
    | `‘`  | `&apos;` |

**两种标准的解析XML的API：**

- DOM：一次性读取XML，在内存中表示为树形结构
- SAX：已以流的形式读取XML，使用事件回调



------



#### DOM （Document Object Model）



**Java解析XML**

- Document：代表整个XML文档
- Element：元素
- Attribute：属性

**Java DOM核心API：**

```java
DocumentBuilderFactory dbf = DOcumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse("/path/to/file.xml");
// get root node
Element root = doc.getDocumentElement();
```

- 将XML解析为DOM
- 可在内存中完整表示XML数据结构
- ==解析速度慢，内存占用大==

##### Code Demo

```java
import org.w3c.dom.Document;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

public class DOM_demo {


    static final String XML_ADDRESS = "C:\\Workstation_Aoc__\\Java\\MavenTest\\pom.xml";

    public static void main(String[] args) throws Exception{
        DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder documentBuilder = builderFactory.newDocumentBuilder();
        Document document = documentBuilder.parse(XML_ADDRESS);

        System.out.println(document.getDocumentElement());
        // print out the path of the XML file
        System.out.println(document.getDocumentURI());
    }
}
```

------

#### SAX (Simple API for XML)

- 基于事件的API



**SAX解析会触发一系列事件：**

- startDocument
- startElement
- ...
- startElement
- characters
- endElement
- ...
- endElement
- endDocument



**SAX解析XML**

- 一种流式解析XML的API
- ==通过事件触发，速度快==
- 调用方通过回调方法获得数据

##### Code Demo

SAX

```java
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

public class SAX_demo {

    static final String XML_ADDRESS = "C:\\Workstation_Aoc__\\Java\\MavenTest\\pom.xml";
	
    /**
     * Almost like DOM creating
     * One addition is Handler
     */
    public static void main(String[] args) throws Exception{
        SAXParserFactory parserFactory = SAXParserFactory.newInstance();
        SAXParser saxParser = parserFactory.newSAXParser();
        saxParser.parse(XML_ADDRESS, new SAXHandler());
    }
}
```

Handler

```java

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;
import org.xml.sax.helpers.DefaultHandler;

public class SAXHandler extends DefaultHandler {

    void print(Object... objects) {
        for (Object object : objects) {
            System.out.println(object);
            System.out.println("  ");
        }
    }
	/**
	 * Override methods
	 */
    @Override
    public void startDocument() throws SAXException {
        print("Start Document");
    }

    @Override
    public void endDocument() throws SAXException {
        print("End Document");
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        print("Start element: ", localName, qName, attributes);
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        print("End element: ", localName, qName);
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        print(" Characters: ", new String(ch, start, length));
    }

    @Override
    public void error(SAXParseException e) throws SAXException {
        print("error: ", e);
    }
}
```



#### Third Library API (Jackson)

**Jackson**

- 开源XML读写工具
- 可在XML和JavaBean之间转换
- API接口简单
- 可定制序列化

##### Code Demo

pom.xml

```xml
<!-- 添加Maven项目的依赖 -->
<dependencies>
	<dependency>
      	<groupId>com.fasterxml.jackson.dataformat</groupId>
      	<artifactId>jackson-dataformat-xml</artifactId>
      	<version>2.9.0</version>
	</dependency>
	<dependency>
      	<groupId>org.codehaus.woodstox</groupId>
      	<artifactId>woodstox-core-asl</artifactId>
      	<version>4.4.1</version>
	</dependency>
    ...
</dependencies> 
```



##### Code Demo

class object

```java
package org.xml.jackson;

import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;

// Root node
@JacksonXmlRootElement(localName = "poms")
public class Poms {

    // Child node
    private String name = "T-MAC";
    private String age = "40";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }
}
```

XML<-->Beam

```java
package org.xml.jackson;

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.dataformat.xml.JacksonXmlModule;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;

import java.io.*;

public class XML2Bean {

    static final String XML_URL = "/home/hisoka/Workstation/IdeaProject/test.xml";

    public static void main(String[] args) throws Exception{
        /**
         * -------XML to JavaBean--------
         * @ xml -- read content from XML_URL
         * @ pom -- transfer xml to Bean(Poms.class)
         */
        Poms poms = null;
        XmlMapper mapper = new XmlMapper();
        String xml = inputStreamToString(new FileInputStream(new File(XML_URL)));
        poms = mapper.readValue(xml, Poms.class);
        System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(poms));

		/**
		 * ---------JavaBean to XML-----------
		 * @ xml1 -- read content from Poms.class JavaBean and transfer to XML
		 */
        XmlMapper mapper1 = new XmlMapper();
        String xml1 = mapper1.writeValueAsString(new Poms());
        System.out.println(xml1);
    }

    static String inputStreamToString(InputStream inputStream) throws IOException {
        StringBuilder builder = new StringBuilder();
        String line;
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))){
            while ((line = reader.readLine()) != null) {
                builder.append(line);
            }
        }
        return builder.toString();
    }
}
```

Output

```sh
<poms>
  <name>KB</name>
  <age>41</age>
</poms>

<poms><name>T-MAC</name><age>40</age></poms>

Process finished with exit code 0
```



- 使用Jackson可以快速在XML和JavaBean之间转换
- 可使用Annotation定制序列化和反序列化

------



### JSON



**JSON（JavaScript Objecct Notation）是一种类似JavaScript对象的数据表示格式**

**JSON格式**

- 只允许用UTF-8编码
- 必须使用双引号
- 特殊字符用 `\`转义



##### Code Demo

Person class

```java
package org.json.jackson;

public class Person {

    private String id;
    private String name;
    private int age;

    public Person() {}

    public Person(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * return data by JSON format
     */
    @Override
    public String toString() {
        return "{\n" +
                "    \"id\": \"" + this.id + "\",\n" +
                "    \"name\": \"" + this.name + "\",\n" +
                "    \"age\": " + this.age + ",\n" +
                "}";
    }
}
```

Jackson in JSON

```java
package org.json.jackson;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.JsonNodeFactory;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.io.*;

public class TestJsonJackson {
	// a path of a existed JSON file
    final static String JSON_PATH = "/home/hisoka/Workstation/IdeaProject/JSONJackson/src/test.json";
	
    /**
     * DataBinding JSON -> Bean
     * a JSON file in path serialize to Person.class
     */
    static void testJsonToObject(Person person, String path) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        person = mapper.readValue(new File(path), Person.class);
        System.out.println(person.toString());
        System.out.println(person.getName());
    }
	/**
     * DataBinding Bean -> JSON
     * an Person Object person serialize to JSON file in path
     * a JSON file will created in path
     */
    static void testObjectToJSON(Person person, String path) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(new File(path), person);
    }
	/**
	 * Stream JSON -> Bean
	 */
    static void testJsonToObjectIO(Person person, String path) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        person = mapper.readValue(new FileInputStream(path), Person.class);
        System.out.println(person.toString());
    }
	/**
	 * Stream Bean -> JSON
	 */
    static void testObjectToJSONIO(Person person, String path) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(new FileOutputStream(path), person);
    }
	
    /**
     * Tree Model to create a JSON file
     * need three objects at first
     * 1. JsonNodeFactory
     * 2. JsonFactory
     * 3. JsonGenerator -- the path in this object 
     *
     * ObjectNode.put(...) -- add a key-value in Node
     * ArrayNode.add(...) -- create a key-arrayValue
     * ObjectNode.set(key_name, ArrayNode) -- append a key-arrayValue in Node
     * ObjectMapper.writeTree(JsonGenerator, ObjectNode) -- create nodes with Tree Model
     */
    static void treeModel() throws IOException {
        JsonNodeFactory factory = new JsonNodeFactory(false);
        JsonFactory jsonFactory = new JsonFactory();
        JsonGenerator generator = jsonFactory.createGenerator(new FileWriter(new File("tree.json")));

        ObjectMapper mapper = new ObjectMapper();
        ObjectNode person = factory.objectNode();
		
        person.put("name", "Dong77");
        person.put("age", 26);

        ArrayNode teammates = factory.arrayNode();
        teammates.add("DK").add("JK").add("SN");
        person.set("teammates", teammates);

        ArrayNode gameData = factory.arrayNode();
        gameData.add("34 points").add("12 ast").add("6 reb").add("3 stl");
        person.set("gameData", gameData);

        mapper.configure(SerializationFeature.INDENT_OUTPUT, true);
        mapper.writeTree(generator, person);
    }


    public static void main(String[] args) throws IOException {
        // testJsonToObject
        /**
         * output:
         * {
         * 	  "id": "52332214323",
    		  "name": "Tracy Mcgrady",
    		  "age": 41,
         * }
         * Tracy Mcgrady
        */
        Person person = null;
        testJsonToObject(person, JSON_PATH);
        testJsonToObjectIO(person, JSON_PATH);

        // testObjectToJSON
        /**
         * result.json
         * {"id":"347598345","name":"Kobe Bryant","age":41}
         */
        Person person1 = new Person("347598345", "Kobe Bryant", 41);
        testObjectToJSON(person1, "result.json");
		/**
         * result2.json
         * {"id":"3499998345","name":"D Wade","age":36}
         */
        Person person2 = new Person("3499998345", "D Wade", 36);
        testObjectToJSONIO(person2, "result2.json");

        /**
         * tree.json
         * {"name":"Dong77","age":26,"teammates":["DK","JK","SN"],"gameData":["34 points","12 ast","6 reb","3 stl"]}
         */
        treeModel();
    }
}
```



