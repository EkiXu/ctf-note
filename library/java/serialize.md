# java序列化

## JSON

### Fastjson

fastjson 在autoType反序列化时会调用反序列化类的一些``is|getter|setter``方法，从而触发反序列化

### Jackson


Jackson是一个非常流行且高效的基于Java的库，用于将Java对象序列化或映射到JSON和XML，也可以将JSON和XML转换为Java对象。在Druid也有对其的依赖。


jaskson有个~~bug~~特性


在这个漏洞中利用了Jackson的一个（~~特性~~）BUG

这个Bug出在Jackson的两个注释上

1. @JsonCreator

   在于对用JsonCreator注解修饰的方法来说，方法的所有参数都会解析成CreatorProperty类型，对于没有使用JsonProperty注解修饰的参数来说,会创建一个name为””的CreatorProperty，在用户传入键为””的json对象时就会被解析到对应的参数上。

2. @JacksonInject 
   假设json字段有一些缺少的属性，抓换成实体类的时候没有的属性将为null,但是我们在某些需求当中需要将为null的属性都设置为默认值，这时候我们就可以用到这个注解了，它的功能就是在反序列化的时候将没有的字段设置为我们设置好的默认值。

具体可以编写下面的示例代码

```java
package com.example.demo;


import com.fasterxml.jackson.annotation.JacksonInject;
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;


public class DemoApplication {

    public static void main(String[] args) throws JsonProcessingException {

        String json= "{\"name\":\"Jack\",\"\":\"Nofield\"}";

        ObjectMapper mapper = new ObjectMapper();
        Student result = mapper.readValue(json, Student.class);
        System.out.print(mapper.writeValueAsString(result));
    }

}

class Student {
    @JsonCreator
    public Student(
            @JsonProperty("name")String name,
            @JacksonInject String id
    ){
        this.name=name;
        this.id=id;
    }

    private String id;
    private String name;
    public String getName() {return name;}

    public String getId() {return id;}
    public void setId(String id) {this.id = id;}

}
```

运行输出

```
{"name":"Jack","id":"Nofield"}
```

可以看到``json``串中键为空的值被赋值到了``id``属性上,这是因为``JsonCreator``给``id``设置的键为空，``JsonInject``将``json``串中空键对应的值``Nofield``赋给了``id``

