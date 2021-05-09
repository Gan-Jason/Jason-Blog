---
title: "JAX-RS相关的RESTful"
date: 2019-07-29T19:00:50+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","JAX-RS","RESTful","Web"]
author: "Jason·Gan"
---

* Java EE 6 引入了对 JSR-311 的支持。JSR-311（JAX-RS：Java API for RESTful Web Services）旨在定义一个统一的规范，使得 Java 程序员可以使用一套固定的接口来开发 REST 应用，避免了依赖于第三方框架。同时，JAX-RS 使用 POJO 编程模型和基于标注的配置，并集成了 JAXB，从而可以有效缩短 REST 应用的开发周期。  

## Resource 类和 Resource 方法  
* Web 资源作为一个 Resource 类来实现，对资源的请求由 Resource 方法来处理。Resource 类或 Resource 方法被打上了 Path 标注，Path 标注的值是一个相对的 URI 路径，用于对资源进行定位，路径中可以包含任意的正则表达式以匹配资源。和大多数 JAX-RS 标注一样，Path 标注是可继承的，子类或实现类可以继承超类或接口中的 Path 标注。  
* Resource 类是 POJO，使用 JAX-RS 标注来实现相应的 Web 资源。Resource 类分为根 Resource 类和子 Resource 类，区别在于子 Resource 类没有打在类上的 Path 标注。Resource 类的实例方法打上了 Path 标注，则为 Resource 方法或子 Resource 定位器，区别在于子 Resource 定位器上没有任何 @GET、@POST、@PUT、@DELETE 或者自定义的 @HttpMethod。清单 1展示了示例应用中使用的根 Resource 类及其 Resource 方法。  
```  

@Path("/") 
public class BookkeepingService { 
   ...... 
   @Path("/person/") 
   @POST 
   @Consumes("application/json") 
   public Response createPerson(Person person) { 
       ...... 
   } 
 
   @Path("/person/") 
   @PUT 
   @Consumes("application/json") 
   public Response updatePerson(Person person) { 
       ...... 
   } 
 
   @Path("/person/{id:\\d+}/") 
   @DELETE 
   public Response deletePerson(@PathParam("id") 
   int id) { 
       ...... 
   } 
 
   @Path("/person/{id:\\d+}/") 
   @GET 
   @Produces("application/json") 
   public Person readPerson(@PathParam("id") 
   int id) { 
       ...... 
   } 
 
   @Path("/persons/") 
   @GET 
   @Produces("application/json") 
   public Person[] readAllPersons() { 
       ...... 
   } 
 
   @Path("/person/{name}/") 
   @GET 
   @Produces("application/json") 
   public Person readPersonByName(@PathParam("name") 
   String name) { 
       ...... 
} 
```  

## 参数与返回值类型  
Resource 方法合法的参数类型包括：
* 原生类型:
    * 构造函数接收单个字符串参数或者包含接收单个字符串参数的静态方法 valueOf的任意类型
    * List<T>，Set<T>，SortedSet<T>（T 为以上的 2 种类型）
    * 用于映射请求体的实体参数
* Resource方法合法的返回值类型包括：
    * void：状态码 204 和空响应体
    * Response：Response 的 status 属性指定了状态码，entity 属性映射为响应体
    * GenericEntity：GenericEntity 的 entity 属性映射为响应体，entity 属性为空则状态码为 204，非空则状态码为 200
    * 其它类型：返回的对象实例映射为响应体，实例为空则状态码为 204，非空则状态码为 200
* 对于错误处理，Resource方法可以抛出非受控异常 WebApplicationException 或者返回包含了适当的错误码集合的Response对象。  

## Context注解  
* 通过 Context 标注，根Resource类的实例字段可以被注入如下类型的上下文资源：
    * Request、UriInfo、HttpHeaders、Providers、SecurityContext
    * HttpServletRequest、HttpServletResponse、ServletContext、ServletConfig 

## CRUD  
* JAX-RS 定义了 @POST、@GET、@PUT 和 @DELETE，分别对应 4 种 HTTP 方法，用于对资源进行创建、检索、更新和删除的操作。  
* 请求方法中的参数类型和返回参数类型可以通过@Consumes和@Produces注解进行注入  

## JAX-RS 与 JPA 的结合使用  
* 由于 JAX-RS 和 JPA 同样都使用了基于 POJO 和标注的编程模型，因而很易于结合在一起使用。示例应用中的 Web 资源 ( 如账目 ) 同时也是持久化到数据库中的实体，同一个 POJO 类上既有 JAXB 的标注，也有 JPA 的标注 ( 或者还有 Gson 的标注 ) ，这使得应用中类的个数得以减少。如 清单 7所示，Account 类可以在 JAX-RS 与 JPA 之间得到复用，它不但可以被 JAX-RS 绑定为请求体 / 响应体的 XML/JSON 数据，也可以被 JPA 持久化到关系型数据库中。  
* 举个例子：  
```  

@Entity 
@Table(name = "TABLE_ACCOUNT") 
@XmlRootElement 
public class Account { 
   @Id 
   @GeneratedValue(strategy = GenerationType.IDENTITY) 
   @Column(name = "COL_ID") 
   @Expose 
   private int id; 
 
   @ManyToOne 
   @JoinColumn(name = "COL_PERSON") 
   @Expose 
   private Person person; 
 
   @Column(name = "COL_AMOUNT") 
   @Expose 
   private BigDecimal amount; 
 
   @Column(name = "COL_DATE") 
   @Expose 
   private Date date; 
 
   @ManyToOne 
   @JoinColumn(name = "COL_CATEGORY") 
   @Expose 
   private Category category; 
 
   @Column(name = "COL_COMMENT") 
   @Expose 
   private String comment; 
......
```