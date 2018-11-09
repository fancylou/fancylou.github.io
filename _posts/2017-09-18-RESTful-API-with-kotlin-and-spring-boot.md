---
layout: post
title: "RESTful API with kotlin and Spring Boot"
categories: kotlin RESTful
date: 2017-09-18 13:54:00
---



Kotlin语言结合Spring Boot 开发RESTful请求的的web项目，Spring 官方提供了一个超级简单的生成项目工具，[Spring Initialzr ](http://start.spring.io/)只要选择相应的构建工具、开发语言和需要依赖的包就行了，非常方便。

[SpringBootKotlinDemo](https://github.com/fancylou/SpringBootKotlinDemo) 我把生成好的项目放到了Github上面，非常简单的一个例子。使用到了[HSQLDB](http://hsqldb.org/)和[Spring Data JPA](http://projects.spring.io/spring-data-jpa/)，例子中我建了一个Student的实体类，然后使用JPA的API，通过StudentRepository这个接口将对实体信息进行实例化。

使用Kotlin先建一个实体类，Student：

```kotlin
@Entity
class Student(
        var name: String = "",
        var sex: String = "",
        @Id @GeneratedValue(strategy = GenerationType.AUTO)
        var id: Long = 0
)
```

一如既往的简单明了，什么getter、setter见鬼去吧。通过注解`@Entity`让JPA识别实体类，还有`@Id` 、 `@GeneratedValue` 注解标注了该实体类的主键，以及主键生成方式。



<!-- more -->



然后给Student创建一个Repository接口，很简单只要集成Spring的CrudRepository接口就好了，基本的增删改查方法就都有了。

```kotlin
interface StudentRepository : CrudRepository<Student, Long> {
    fun findByName(name: String): List<Student>
}
```

这个接口上面只要根据名称定义一些函数就可以执行了，比如上面的`findByName` 就会通过Student里面的name属性去查询数据，这个是JPA相关的问题了。

有了上面数据库的操作了就可以定义RESTful请求了，新建一个StudentController，在Spring里面习惯叫Controller

```kotlin
@RestController
@RequestMapping("/students")
class StudentController(val repository: StudentRepository) {


    @GetMapping
    fun findAll() = repository.findAll()

    @PostMapping
    fun addStudent(@RequestBody student:Student) = repository.save(student)

    @PutMapping("/{id}")
    fun updateStudent(@PathVariable id: Long, @RequestBody student: Student) {
        assert(id == student.id)
        repository.save(student)
    }

    @GetMapping("/name/{name}")
    fun findByName(@PathVariable name:String) = repository.findByName(name)

    @GetMapping("/{id}")
    fun findOne(@PathVariable id: Long) = repository.findOne(id)

    @DeleteMapping("/{id}")
    fun deleteById(@PathVariable id: Long) = repository.delete(id)
    
}
```

类上面的  `@RequestMapping("/students")` 注解就表示这个类的所有链接都要加上/students这个前缀。然后里面方法上面的 `@GetMapping`  `@PostMapping` `@PutMapping`  `@DeleteMapping`  分别就代表了RESTful请求的 GET、POST、PUT、DELETE请求类型。

OK，使用kotlin写的一个简单的依赖Spring的RESTful API项目就有了，这时候想要测试请求，可以通过SpringBoot启动，测试。需要定义一个SpringBootApplication

```kotlin
@SpringBootApplication
open class SpringKotlinApplication {

    @Bean
    open fun init(repository: StudentRepository) = CommandLineRunner {
        repository.save(Student("Fancy","man"))
        repository.save(Student("Lily","women"))
    }
}

fun main(args: Array<String>) {
   SpringApplication.run(SpringKotlinApplication::class.java, *args)
}
```

里面的 `init` 方法是初始化了一些数据进去，主要定义一个 `main` 函数，通过这个函数启动项目。

这个时候就可以使用RESTful请求工具对数据进行增删改查了。

GET findAll

![GET](http://img.muliba.net/blog/post/20170918/students-all.png)

GET findById

![GET](http://img.muliba.net/blog/post/20170918/students-one.png)

GET findByName

![GET](http://img.muliba.net/blog/post/20170918/students-name.png)

PUT update name

![PUT](http://img.muliba.net/blog/post/20170918/students-put-1.png)

GET findById

![GET](http://img.muliba.net/blog/post/20170918/students-put-2.png)

