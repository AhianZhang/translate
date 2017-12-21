# 怎么使用 JPA 和 Hibernate 映射 JSON 集合

## 介绍

开源项目： [hibernate-types](https://vladmihalcea.com/2017/09/25/the-hibernate-types-open-source-project-is-born/) 允许你映射 JAVA 对象或者 Jackson JsonNode 作为 JPA 实体属性。

最近，感谢我们很棒的的构建者，我们也已经添加了将类型安全集合作为JSON持久化的支持。在这篇文章，你们会看到如何实现这一目标。

## Maven Dependency

首先，你需要在你的项目中的 `pom.xml` 配置文件中添加以下 Maven 依赖：
```xml
<dependency>
    <groupId>com.vladmihalcea</groupId>
    <artifactId>hibernate-types-52</artifactId>
    <version>${hibernate-types.version}</version>
</dependency>
```

如果你在使用老版本的 Hibernate，请检查 `[Hibernate-types GitHub repository](https://github.com/vladmihalcea/hibernate-types)` 来获取你当前 Hibernate 版本的依赖更多的匹配信息。

## 域模型

让我们假定有以下`Location` 的 Java 对象类型。
```java
public class Location implements Serializable {

    private String country;

    private String city;

    //Getters and setters omitted for brevity

    @Override
    public String toString() {
        return "Location{" +
                "country='" + country + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

和一个 `Event` 实体
```java
@Entity(name = "Event")
@Table(name = "event")
public class Event extends BaseEntity {

    @Type(type = "jsonb")
    @Column(columnDefinition = "jsonb")
    private Location location;

    @Type(
        type = "jsonb",
        parameters = {
            @org.hibernate.annotations.Parameter(
                name = TypeReferenceFactory.FACTORY_CLASS,
                value = "com.vladmihalcea.hibernate.type.json.PostgreSQLGenericJsonBinaryTypeTest$AlternativeLocationsTypeReference"
            )
        }
    )
    @Column(columnDefinition = "jsonb")
    private List<Location> alternativeLocations = new ArrayList<Location>();

    //Getters and setters omitted for brevity
}
```

`BaseEntity` 定义了一些基础属性(例如，@Id, @Version 和少数自定义的 Hibernate 类型，在这里，我们对 JsonBinaryType 很感兴趣)。

```java
@TypeDefs({
    @TypeDef(name = "string-array", typeClass = StringArrayType.class),
    @TypeDef(name = "int-array", typeClass = IntArrayType.class),
    @TypeDef(name = "json", typeClass = JsonStringType.class),
    @TypeDef(name = "jsonb", typeClass = JsonBinaryType.class),
    @TypeDef(name = "jsonb-node", typeClass = JsonNodeBinaryType.class),
    @TypeDef(name = "json-node", typeClass = JsonNodeStringType.class),
})
@MappedSuperclass
public class BaseEntity {

    @Id
    private Long id;

    @Version
    private Integer version;

    //Getters and setters omitted for brevity
}
```

有关使用`@mappedSuperclass` 的更多细节，请查看[这篇文章](https://vladmihalcea.com/2017/11/08/how-to-inherit-properties-from-a-base-class-entity-using-mappedsuperclass-with-jpa-and-hibernate/)。

## TypeReferenceFactory

为了存储`Location`对象到 jsonb PostgreSQL 列中，我们仅需要在 location 属性中声明 `@Type(type = "jsonb")`。

然而，对于 `alternativeLocations` 集合，我们需要提供关联 Jackson `[TypeReference](https://fasterxml.github.io/jackson-core/javadoc/2.2.0/com/fasterxml/jackson/core/type/TypeReference.html)`,这样我们在从关系型数据库读取 Json 对象时能够非常方便的重构安全类型的 Java 集合。

对于这个计划，我们提供了有关 `TypeReferenceFactory` 完全合格的类实现，类似下面这样：

```java
public static class AlternativeLocationsTypeReference 
    implements TypeReferenceFactory {
    
    @Override
    public TypeReference<?> newTypeReference() {
        return new TypeReference<List<Location>>() {};
    }
}
```

就是这样

## 测试时间

当保存下面的 `Event` 实体时：

```java
Location cluj = new Location();
cluj.setCountry("Romania");
cluj.setCity("Cluj-Napoca");

Location newYork = new Location();
newYork.setCountry("US");
newYork.setCity("New-York");

Location london = new Location();
london.setCountry("UK");
london.setCity("London");

Event event = new Event();
event.setId(1L);
event.setLocation(cluj);
event.setAlternativeLocations(
    Arrays.asList(newYork, london)
);

entityManager.persist(event);
```

Hibernate 将会生成下面的 SQL 插入语句：

```java
INSERT INTO event (
    version, 
    alternativeLocations, 
    location, 
    id
) 
VALUES (
    0, 
    [
        {"country":"US","city":"New-York"},
        {"country":"UK","city":"London"}
    ], 
    {"country":"Romania","city":"Cluj-Napoca"}, 
    1
)
```

当检索 `Event` 实体时,`location` 和 `alternativeLocations` 属性恩能够正确取得：

*Event event = entityManager.find(Event.class, eventId);*

```java
assertEquals(
    "Cluj-Napoca", 
    event.getLocation().getCity()
);

assertEquals(2, event.getAlternativeLocations().size());

assertEquals(
    "New-York", 
    event.getAlternativeLocations().get(0).getCity()
);
assertEquals(
    "London", 
    event.getAlternativeLocations().get(1).getCity()
);
```

很酷，对吧？

>如果你喜欢这篇文章，我猜你会同样爱上[我的书](https://leanpub.com/high-performance-java-persistence?utm_source=blog&utm_medium=banner&utm_campaign=article)。
![](https://vladmihalcea.files.wordpress.com/2015/11/hpjp_small.jpg?w=150)

## 总结

`[Hibernate-types](https://github.com/vladmihalcea/hibernate-types)` 不仅仅支持 JSON 类型。你同样可以映射 PostgreSQL 数组类型或者 PostgreSQL 指明枚举类型，可为空的`字符`，甚至是你自己不可变的 Hibernate 定制类型。