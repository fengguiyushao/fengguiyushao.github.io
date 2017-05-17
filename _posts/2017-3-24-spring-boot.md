---
layout:     post
title:      "Spring Boot搭建环境"
date:       2017-03-24 12:00:00
author:     "robinzhou"
catalog: true
tags:
    - java
    - spring
  	- web
---



### Spring Boot搭建环境

Spring Boot为开发人员简化了新Spring应用的初始搭建以及开发过程，本文详细描述了使用Spring Boot+Restful+Web MVC+H2搭建环境。



### Gradle配置

需要添加如下dependence

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.2.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-actuator")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("junit:junit")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
}
```



### Spring Boot

首先需要一个启动文件SampleApplication

```java
@SpringBootApplication()
public class SampleApplication {


    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleApplication.class, args);
    }
}
```

配置一个GreetingController

```java
@Controller
public class GreetingController {

    @RequestMapping("/greeting")
    public String greeting(@RequestParam(value = "name", required = false, defaultValue = "World") String name, Model model) {
        model.addAttribute("name", name);
        return "greeting";
    }

}
```

在`resources/templates`中添加模板文件`greeting.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```

至此，Spring Boot已经配置完成，可以通过http://localhost:8080/greeting访问项目。

如果需要更改web项目监听的端口，在配置文件`application.properties`中设置端口号

```properties
server.port = 80
```



### H2配置

首先定义Bean类

```java
@Entity
@Table(name="DemoInfo")
public class DemoInfo {

    @Id
    @GeneratedValue
    private long id;

    private String title;

    private String content;

    public DemoInfo() {

    }

    public DemoInfo(String title, String content) {
        this.title = title;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
```

处理Request的Controller代码如下，支持保存和查询，以及条件查询

```java
@RestController
public class DemoInfoController {

    @Autowired
    private DemoInfoRepository demoInfoRepository;

    /**
     * 保存数据.
     * @return
     */
    @RequestMapping("/save")
    public String save(){
        // 内存数据库操作
        demoInfoRepository.save(new DemoInfo("title1", "content1"));
        demoInfoRepository.save(new DemoInfo("title2", "content2"));
        demoInfoRepository.save(new DemoInfo("title3", "content3"));
        demoInfoRepository.save(new DemoInfo("title4", "content4"));
        demoInfoRepository.save(new DemoInfo("title5", "content5"));
        return "save ok";
    }

    /**
     * 获取所有数据.
     * @return
     */
    @RequestMapping("/findAll")
    public Iterable<DemoInfo> findAll(){
        // 内存数据库操作
        return demoInfoRepository.findAll();
    }

    @RequestMapping("findByTitle")
    public List<DemoInfo> findByTitle(@RequestParam(value = "title", defaultValue = "title") String title) {
        return demoInfoRepository.findByLikeTitle(title);
    }
}
```

持久化使用的是JPA，需要定义`Repository`，继承`CrudRepository`

```java
public interface DemoInfoRepository extends CrudRepository<DemoInfo, Long>, CustomRepository {
}
```

自定义`CustomRepository`来支持条件查询

```java
public interface CustomRepository {

    List<DemoInfo> findByLikeTitle(String title);
}
```

条件查询`findByLikeTitle`需要自己实现查询逻辑，因此需要继承`CustomRepository`

```java
public class DemoInfoRepositoryImpl implements CustomRepository {

    @PersistenceContext
    private EntityManager em;

    @Override
    public List<DemoInfo> findByLikeTitle(String title) {
        CriteriaBuilder builder = em.getCriteriaBuilder();
        CriteriaQuery<DemoInfo> query = builder.createQuery(DemoInfo.class);
        Root<DemoInfo> root = query.from(DemoInfo.class);
        query.where(builder.like(root.get("title"), "%" + title + "%"));
        return em.createQuery(query.select(root)).getResultList();
//        return em.createNativeQuery("select * from DemoInfo where 1=1").getResultList();
    }
}
```



至此Spring Boot搭建的一个简单Demo完成了。