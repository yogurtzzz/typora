Jackson使用

最简单的使用

新建一个`ObjectMapper`实例

```java
ObjectMapper mapper = new ObjectMapper();

Student student = Student.builder()
    .name("yogurt")
    .age(16)
    .score(95)
    .build();

String json = mapper.writeValueAsString(student);
//json = {"name":"yogurt","age":16,"score":95}

//从json串还原JAVA对象
Student src = mapper.readValue(json,Student.class);

//使用prettyPrint
mapper.enable(SerializationFeature.INDENT_OUTPUT);
String prettyJson = mapper.writeValueAsString(student);
System.out.println(prettyJson);
/*
{
  "name" : "yogurt",
  "age" : 16,
  "score" : 95
}
*/
//也可使用writer进行prettyPrint
ObjectWriter writer = mapper.writer();
writer.with(SerializationFeature.INDENT_OUTPUT)；
String str = writer.writeValueAsString(student);

//可以使用writer或者mapper直接将对象转换为json字符串并写入到文件
ObjectWriter writer = mapper.writer();
//压缩格式
writer.with(JsonGenerator.Feature.ESCAPE_NON_ASCII);
//展开格式
//writer.with(SerializationFeature.INDENT_OUTPUT);
writer.writeValue(new File("E:\\student.json"),student);

//或者
mapper.enable(JsonGenerator.Feature.ESCAPE_NON_ASCII);
mapper.writeValue(new File("E:\\student.json"),student);
//同理也可以从文件里直接读取json串并转换为JAVA对象

//针对某个对象的自定义输出,需要创建一个自定义的JsonSerializer子类
public static class AnimalSerializer extends JsonSerializer<Animal>{
        @Override
        public void serialize(Animal animal, JsonGenerator jgen, SerializerProvider sp) throws IOException {
            jgen.writeStartObject();
            jgen.writeBooleanField("what",true);
            jgen.writeNumberField("num?",1);
            jgen.writeStringField("str?",animal.getScale()+animal.getSpecies());
            jgen.writeFieldName("arr?");
            jgen.writeStartArray();
            jgen.writeString("arr1");
            jgen.writeString("arr2");
            jgen.writeString("arr3");
            jgen.writeEndArray();
            jgen.writeEndObject();
        }
    }

	@Test
    public void test() throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(Animal.class,new AnimalSerializer());
        mapper.registerModule(simpleModule);
        //这一步注册module，可以简化为在Animal类上加注解
        Animal animal = Animal.builder()
                .scale("bigScale")
                .species("Birds")
                .build();
        String customJson = mapper.writeValueAsString(animal);
        System.out.println(customJson);
        //{"what":true,"num?":1,"str?":"bigScaleBirds","arr?":["arr1","arr2","arr3"]}
    }

//直接在Animal类上使用@JsonSerialize注解，指定自定义的Serializer，即可避免给ObjectMapper配置module
	@Getter
    @Setter
    @Builder
    @JsonIgnoreProperties(ignoreUnknown = true)
    @JsonSerialize(using = AnimalSerializer.class)
    public static class Animal{
        private String species;
        private String scale;
    }

//当json字符串里有一些属性，JAVA类中没有的时候，可以使用@JsonIgnoreProperties属性中的ignoreUnknown = true 来屏蔽掉
```





从json字符串，构造出JsonNode节点

```java
String data = "some json string here ...";
ObjectMapper mapper = new ObjectMapper();
JsonNode rootNode = mapper.readTree(data);
//此时得到的是一个ObjectNode
//可以通过JsonNode的path() 方法来获取其子节点
JsonNode request = rootNode.path("request");
//若某个子节点已经是叶子节点，则可以用asText来获取其字符串值
JsonNode status = request.path("status");
String value = status.asText();
//若某个子节点不是叶子节点（其还有子节点），想要获取该节点的json字符串表示，使用toString() 而不是asText()
String jsonStr = request.toString();
//若一个JsonNode是一个ArrayNode （这个节点是一个数组节点）
//那么用迭代器的方式来访问其中的元素
```



若要从Object对象中构造出JsonNode节点

```java
//使用ObjectMapper的valueToTree()方法
```



新建一个JsonNode

```java
ObjectMapper mapper = new ObjectMapper();
ObjectNode objNode = mapper.createObjectNode();
objNode.put("name","yogurt");
objNode.put("age",6);

ArrayNode arrNode = mapper.createArrayNode();
arrNode.add(1);
arrNode.add(2);
arrNode.add(3);
```



关于jackson的一个bug

若属性的命名为，第一个字母是小写，紧接着是大写字母

如 `iLoveYou`，在转换成Object时，属性会自动变为`iloveYou`，导致解析失败

至少是2个小写字母开头，然后接大写字母，就不会出现这种问题

或者在属性上加`@JsonPrperty("iLoveYou")`