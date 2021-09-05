---
title: GraphQL在Spring Boot项目中的实践
date: 2021-09-05
tags:
---

## 什么是GraphQL？

GraphQL 是一个用于 API 的查询语言，是一个使用基于类型系统来执行查询的服务端运行时（类型系统由你的数据定义）。它对标的是REST这种接口风格，它重新定义了一种接口模式，让调用方可以指定需要查询的数据，而且没有任何冗余。

<!-- more --> 

GraphQL相比现在流行的RESTful接口的优势在哪呢？考虑一种场景：一个购物系统，提供在主页展示商品简略信息的接口，这个接口只用查询出商品的名称、价格和介绍图地址，然后当我进入到商品的详情页面时需要用另一个接口查出商品的更多具体信息。同样是查询商品信息这个操作在不同的场景可能就要定义`/product/brief`和`/product/detail/`等多个接口。

而对于GraphQL，查询简略信息的场景，我们可以用这样一个查询语句：

```
{
  product {
    name
    price
    picture
  }
}
```

而在详情页面我们的查询语句是这样：

```
{
  product {
    name
    price
    originalPrice
    picture
    morePicture
    publishTime
    ...
  }
}
```

这样我们的客户端查询更加灵活，服务端不用编写更多的接口应对更多的场景，可复用性更好。这就是GraphQL最大也最直观的优势了。

## 在Spring Boot项目中编写GraphQL接口

首先为项目添加graphql-java的依赖：

```
<!-- GraphQL -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java</artifactId>
    <version>11.0</version>
</dependency>
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-spring-boot-starter-webmvc</artifactId>
    <version>1.0</version>
</dependency>
```

然后后在resource目录下添加一个定义Schema的文件，如`schema.graphqls`。Schema定义了一个Query用于查询，blogById属性返回Blog类型数据，然后定义了几个实体类型。完整的定义如下：

![截屏2021-09-05 20.17.08](https://i.loli.net/2021/09/05/ZwGEDuKAxpodB1n.png)

然后最重要的部分是实现数据获取的部分。

首先定义一个GraphQLProvider类，它的作用是结合`TypeDefinitionRegistry`和`RuntimeWiring`并生成最终的GraphQL，把GraphQL这个Bean注入到Spring容器中就能对外提供一个`/graphql`的HTTP接口。下面分别说说`TypeDefinitionRegistry`和`RuntimeWiring`是什么。

`TypeDefinitionRegistry`比较简单就是从之前`schema.graphqls`读取到的Schema定义；`RuntimeWiring`要知道如何去获取Schema中定义的类型的数据，这里就需要我们自己实现`DataFetcher`了。借用一下官方的图展示一下它们的关系：

![graphql_creation](https://www.graphql-java.com/images/graphql_creation.png)

下面是GraphQLProvider的代码，和官方的图基本一致比较清晰：

```java
@Component
public class GraphQLProvider {
    @Autowired
    GraphQLDataFetchers graphQLDataFetchers;

    private GraphQL graphQL;

    @Bean
    public GraphQL graphQL() {
        return graphQL;
    }

    @PostConstruct
    public void init() throws IOException {
        GraphQLSchema graphQLSchema = buildSchema();
        this.graphQL = GraphQL.newGraphQL(graphQLSchema).build();
    }

    private GraphQLSchema buildSchema() throws IOException {
        TypeDefinitionRegistry typeRegistry = buildRegistry();
        RuntimeWiring runtimeWiring = buildWiring();
        SchemaGenerator schemaGenerator = new SchemaGenerator();
        // 结合 TypeDefinitionRegistry 和 RuntimeWiring 生成 GraphQLSchema
        return schemaGenerator.makeExecutableSchema(typeRegistry, runtimeWiring);
    }

    private TypeDefinitionRegistry buildRegistry() throws IOException {
        URL url = Resources.getResource("schema.graphqls");
        String sdl = Resources.toString(url, Charsets.UTF_8);
        return new SchemaParser().parse(sdl);
    }

    private RuntimeWiring buildWiring() {
        return RuntimeWiring.newRuntimeWiring()
                .type(newTypeWiring("Query")
                        .dataFetcher("blogById", graphQLDataFetchers.getBlogByIdDataFetcher()))
                .type(newTypeWiring("Blog")
                        .dataFetcher("author", graphQLDataFetchers.getAuthorDataFetcher()))
                .build();
    }
}
```

然后看看具体的DataFetchers怎么实现：

```java
@Component
public class GraphQLDataFetchers {
    @Autowired
    BlogService blogService;
    @Autowired
    UserService userService;

    public DataFetcher<Blog> getBlogByIdDataFetcher() {
        return dataFetchingEnvironment -> {
            Integer blogId = dataFetchingEnvironment.getArgument("id");
            return blogService.findById(blogId);
        };
    }

    public DataFetcher<User> getAuthorDataFetcher() {
        return dataFetchingEnvironment -> {
            Blog blog = dataFetchingEnvironment.getSource();
            Integer aid = blog.getAid();
            return userService.findById(aid);
        };
    }
}
```

DataFetcher设计采用了函数式编程的思想，它是一个函数式接口，提供了dataFetchingEnvironment获取查询参数，返回查询结果，中间如何去查到数据由函数内部实现GraphQL框架并不关注。这里我直接使用之前写好的业务层接口BlogService根据id查询Blog，然后根据Blog的aid用UserService查询author。**写到这里其实大致就能看出来GraphQL它的功能主要体现control层，要在原来的RESTful接口上做迁移也只用修改control层的代码。**

写完了之后运行项目看看效果，我这里使用了`Altair GraphQL Client`这个浏览器插件调用GraphQL接口：

![截屏2021-09-05 21.08.34](https://i.loli.net/2021/09/05/XJeKBYbLO8jESI1.png)

这里要注意的是GraphQL接口是基于HTTP协议的，**但是查询语句并不是JSON格式**，但是返回结果是标准的JSON格式，从浏览器控制台可以看到具体的请求格式：

![截屏2021-09-05 21.11.52](https://i.loli.net/2021/09/05/9KQyfkWHLoPih1c.png)

## 对GraphQL的一些看法

前面提到了GraphQL相对传统的RESTful接口的优势非常明显，客户端使用接口更加灵活，服务端接口复用程度高。

如果想把现有的项目迁移到GraphQL上的话需要只需要修改服务端的control层，但是因为GraphQL的查询语句不是传统的JSON格式，所以客户端也需要做相应的更改。

而GraphQL另一个重要的问题在哪呢，看上面对项目GraphQL的接口调用可以发现接口都是以`/graphql`这个路径发布的，原来系统传统的根据路径做鉴权的方法基本都失效了，比如在这个DEMO中我为了调通这个接口，把`/graphql`这个路径加到鉴权的白名单中：

![截屏2021-09-05 21.24.06](https://i.loli.net/2021/09/05/b7O39R4m5PsNcdY.png)

另外对于一个实体不同的用户能访问到的字段也不一样，比如用户隐藏了手机号，那么其他用户是不能在GraphQL中查询到他的手机号的，这样就必须实现字段级的鉴权了。

按照GraphQL官方推荐做法，我们应该把接口鉴权放到业务层，但是原来基于路径的鉴权业务层未鉴权的接口就需要做修改了。

总的来说GraphQL增强了客户端的能力，但是另一方面也算是带来了更多安全隐患，旧系统想要享受GraphQL带来的便利就得在系统的安全策略上做出合理的调整。