# Spring Boot 学习笔记

## 一、RESTFUL 入门案例

### 1、约定规范

|            URL形式            |  结果  |  请求方式  |
| :-------------------------: | :--: | :----: |
| http://localhost:8080/users |  查询  |  GET   |
| http://localhost:8080/users |  新增  |  POST  |
| http://localhost:8080/users |  更新  |  PUT   |
| http://localhost:8080/users |  删除  | DELETE |

###  2、案例详情

```java
@RequestMapping(value = "/books/{id}",method = RequestMethod.GET/POST/PUT/DELETE)
public  String getById(@PathVariable Integer id){
    System.out.println("springboot is running");
    return "springboot is running";
}
```

### 3、注解详情

```java
@RequestMapping //约束请求方式和访问路径，method参数规定CRUD类型
@PathVariable //路径参数，通常用于接收一个参数，比如id
@RequestBody  //请求体参数，json格式;通常用于接收多个参数，json格式提交
@RequestParam //路径参数，表单提交,用于非json格式的提交
```

### 4 快速开发

```java
@RequestMapping("/users") //至于类上方简化方法上方的value参数
@ResponseBody  //置于类上方简化开发
@RestController  //@Controller @ResponseBody 合并
@PostMapping("/{id}")   // 替换 method = RequestMethod.POST ，其余提交方式以此类推
```

## 二、复制工程 

### 1、工程模板

​	相同的内容源码放置在一个模块中，保留工程基础结构，抹除修改痕迹。

​	模板修改方式如下

1. 复制spring boot快速上手工程
2. 修改pom文件中`artifactId`内容
3. 删除target目录
4. 删除配置文件，只保留src和pom.xml
5. `artifactId` 更改为新模块名称



## 三、基础配置 

### 1、服务器端口配置

```properties
server:
  port: 8081
```

### 2、banner配置

```yaml
spring:
  banner:
    location: banner.txt
```

### 3、日志配置

```yaml
logging:
  level:
    root:   (root是包名)
      info
```

### 4、配置文件加载优先级

​	application.properties > application.yml > application.yaml

​	重叠属性按照优先级，不重叠属性全部加载

### 5、yaml属性代码提示消失问题

``` null
file ->Project Structure ->Project Setting ->Facets ->Spring 下选中当前项目 ->选中绿叶子图标的配置文件-> +号调出Choose Custom Configuration Files->选择配置文件路径
```

### 6、配置文件书写要求

- 大小写敏感
- 属性层级关系使用多行描述，每行结尾使用冒号结束
- 使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用Tab键)
- 属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔)
- '#'表示注释
- 多属性使用- 链接

```yaml
likes:
  - game
  - ball
  - music
  - sleep
  - name: zhangsan
    age: 18
  - name: lisi
    age: 20
# 同等于    
likes: [game,ball,music]
users: [{name:zhangsan,age:18},{name:lisi,age:20}]
```

### 7、读取yml单一属性数据

1. 定义一个变量
2. `@Value()` 为变量注入基本数据类型和字符串的格式

```  java
    /*
    读取yml单一属性变量
     */
    @Value("${country}")
    private String country;

    /*
    读取yml对象变量
     */
    @Value("${spring.banner.location}")
    private String banner;

    /*
    读取yml中数组的值（使用索引）
     */
    @Value("${likes[2]}")
    private String likes;

    /*
    读取yml中数组对象的值（使用索引）
    */
    @Value("${users[0].name}")
    private String age;

	/*
	自动全局装载
	*/
	@Autowired
	private Environment env;
	env.getProperty("users[0].age")
    /*
    
    */
```

### 8、 引用数据

``` yaml
baseDir: c:\windows
tmpDir: ${baseDir}\tmp
# "${baseDir}\tmp" 双引号表示的字符串支持转义字符  
```

### 9、读取指定对象数据

要读取如下格式的一段数据

```yaml
# 创建一个类封装下面的数据
# spring 加载数据到对象中
# 让spring知晓加载的信息
# 从spring中获取这段信息
 datasource:
    driver: com.mysql.jdbc.Driver
    url: jdbc:mysql://82.156.167.171/springboot_db
    username: root
    password: Abc@97923
```

JAVA配置如下类用于加载数据到对象中

```java
@Component
@ConfigurationProperties("datasource")
@Data
public class MyDataSource {
    private String driver;
    private String url;
    private String username;
    private String password;

    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

读取数据

```java
    @Autowired
    private MyDataSource myDataSource;
```

## 四、 Spring Boot 整合JUnit

### 1、 boot 测试

1. 注入测试的对象
2. 执行测试的对象的方法

dao中的接口

```java
package com.guojiahe.dao;

public interface BookDao {
    public void save();
}
```

impl中的实现

```java
package com.guojiahe.impl;

import com.guojiahe.dao.BookDao;
import org.springframework.stereotype.Repository;

@Repository
public class BookDaoImpl implements BookDao {
    @Override
    public void save() {
        System.out.println("BookDao");
    }
}
```

test中的测试

```java
    @Autowired
    private BookDao bookDao;

    @Test
    void testJunit(){
        bookDao.save();
    }
```

### 2、 classes属性

需要测试类`@SpringBootTest`注解中使用 `classes` 属性注入引导类名，以防止二者不同包名无法扫描到test的情况

## 五、Spring Boot 整合Mybatis

### 1、基本需求

- 核心配置：数据库连接相关信息（连什么、连谁、权限）

- 映射配置：SQL映射（xml/注解）

- 导入对应的starter

  ```xml
          <dependency>
              <groupId>org.mybatis.spring.boot</groupId>
              <artifactId>mybatis-spring-boot-starter</artifactId>
              <version>2.2.2</version>
          </dependency>

          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <scope>runtime</scope>
          </dependency>
  ```

### 2、yml中配置数据库连接

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://82.156.167.171/springboot_db
    username: root
    password: root
```

### 3、创建实体类、数据层接口、测试

```java
@Data
public class Teacher {
    private Integer id;
    private Integer age;
    private String name;
    private String email;
    private String country;
}
```

```java
@Mapper
public interface TeacherDao {
    @Select("select * from teachers where id =#{id}")
    public Teacher getById(Integer id);
}
```

```java
    @Autowired
    private TeacherDao teacherDao;
    @Test
    void contextLoads() {
        System.out.println(teacherDao.getById(1));
    }
```

## 六、Spring Boot 整合MyBatis-Plus

###1、与MyBatis的区别

- 导入坐标不同
- 数据层实现简化

### 2、导入坐标

[maven仓库](https://mvnrepository.com/)

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
```

### 3、实体类不变，数据层接口

```java
// 继承BaseMapper并传入实体类泛型即可
@Mapper
public interface TeacherDao  extends BaseMapper<Teacher> {
}

```

### 4、测试

```java
    @Autowired
    private TeacherDao teacherDao;
    @Test
    void contextLoads() {
        System.out.println(teacherDao.selectById(2));
    }

    @Test
    void testGetAll(){
        System.out.println(teacherDao.selectList(null));
    }

```

### 5、遇到的错误

由于MyBatis-Plus 自动装填了基本的查询语句，映射数据库时可能发生命名冲突

解决方案1：添加前缀（适用于 前缀_表名类型的表名）

```yaml
# MyBatis-Plus 相关配置
mybatis-plus:
  global-config:
    db-config:
      table-prefix: tbl_
```

解决方案2：实体类前添加` @TableName("表名")` 

## 七、Spring Boot 整合Druid

### 1、导入坐标

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.8</version>
</dependency>
```

### 2、配置

```yaml
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://82.156.167.171/judge
      username: root
      password: root
```

## 八、整合SSMP案例

 ### 1、案例实现方案分析

- 开发实体类 --Lombok快速制作实体类
- Dao开发--整合MP，制作数据层测试类
- Service开发--基于MP做增量开发，制作业务层测试类
- Controller开发--基于Restful开发，使用PostMan测试接口，前后端开发协议制作
- 页面搭建，vue+ElementUI,前后端联调，页面数据处理，页面消息处理，
  - 列表
  - 新增
  - 修改
  - 删除
  - 分页
  - 查询
- 项目异常处理
- 按条件查询
- 功能调整

### 2、创建模块

整合MyBatis-Plus插件、Druid,添加数据库信息

导入lombok

```xml	
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

### 3、初始化数据库表，创建实体类

```java
@TableName("tb_book")
@Data
@NoArgsConstructor
public class Book {
    private Integer id;
    private String type;
    private String name;
    private String description;

}
```

### 4、数据层开发

#### 1、数据层接口

```java
@Mapper   //没有这个注解spring无法识别该类
public interface BookDao {

    /*
    MyBatis 查询方法
     */
    @Select("select * from tb_book where id=#{id}")
    Book getById(Integer id);
}
```

#### 2、基于MP的基本CURD(继承BaseMapper<entity class>)

```yaml
//设置id的自动生成策略
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
```

```java	
    @Test
    void TestGetByIdMP(){
        System.out.println(bookDao.selectById(1));
    }
    @Test
    void TestSaveMP(){
        Book book = new Book();
        book.setType("Test");
        book.setDescription("testDescription");
        book.setName("test");
        bookDao.insert(book);
    }
    @Test
    void TestUpdateMP(){
        Book book = new Book();
        book.setId(15);
        book.setType("Test15");
        book.setDescription("testDescription15");
        book.setName("test15");
        bookDao.updateById(book);
    }
    @Test
    void TestDeleteMP(){
        bookDao.deleteById(16);
    }
    @Test
    void TestGetAllMP(){
        System.out.println(bookDao.selectList(null));
    }
```

#### 3、MP日志处理

```yaml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

#### 4、分页查询

必须使用MP提供的拦截器进行操作

```java
//用这个注解标注这是一个spring配置；类
@Configuration
public class MPConfig {
    @Bean
    //固定写法，拦截后return出去
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

```java
    @Test
    void TestGetPageMP(){
        IPage page =new Page(1,5);
        IPage iPage = bookDao.selectPage(page, null);
        System.out.println(iPage.getPages());
        System.out.println(iPage.getCurrent());
        System.out.println(iPage.getRecords());
        System.out.println(iPage.getSize());
        System.out.println(iPage.getTotal());
    }
```

#### 5、条件查询带分页

```java
    @Test
    void TestGetByMP(){
        /*
        需要创建一个queryWrapper对象,添加查询条件
         */
        // select * from tb_book where name like %spring%
        QueryWrapper<Book> queryWrapper = new QueryWrapper<>();
        queryWrapper.like("name","spring");
        IPage page = new Page(1,5);
        bookDao.selectPage(page,queryWrapper);
    }

```

```java
    @Test
    void TestMistakeGetByMP(){
        /*
        需要创建一个LambdaQueryWrapper对象,添加查询条件时可以直接用实体类中属性名
        lambdaQueryWrapper.like参数中可以对模糊变量进行空值检查
         */
        // select * from tb_book where name like %spring%
        LambdaQueryWrapper<Book> lambdaQueryWrapper = new LambdaQueryWrapper<>();
        lambdaQueryWrapper.like( name!=null,Book::getName,name);
        IPage iPage = new Page(1,5);
        bookDao.selectPage(iPage,lambdaQueryWrapper);
    }
```

### 5、 业务层（service）开发

#### 1、业务层与数据层的区别

以登录操作为例

数据层主要针对数据库操作 ，数据接口名称会是` selectVyUserNameAndPassword(String username,String password)`

业务层主要针对的是业务,接口名称将会是` login(String userName ,String password)`

#### 2、业务层接口创建

创建业务层service包下的业务层接口BookService。不同的业务名称使用不同的方法名即可

```java
public interface BookService {

    Boolean save(Book book);
    Boolean update(Book book);
    Boolean delete(Book book);
    Book getById();
    List<Book> selectAll();
}

```

```java
@Service  //这个注解用来把他定义成一个业务层的bean
public class BookServiceImpl implements BookService {
    
    //注入数据层，转调数据层方法
    @Autowired
    private BookDao bookDao;
    public BookServiceImpl() {
        super();
    }

    @Override
    public Boolean save(Book book) {
        //数据层方法insert返回的值是影响行数，故而返回>0即代表操作成功进行，配置的是逻辑规则
        return bookDao.insert(book)>0;
    }

    @Override
    public Boolean update(Book book) {
        return bookDao.updateById(book)>0;
    }

    @Override
    public Boolean delete(Integer id) {
        return bookDao.deleteById(id)>0;
    }

    @Override
    public Book getById(Integer id) {
        return bookDao.selectById(id);
    }

    @Override
    public List<Book> selectAll() {
        return bookDao.selectList(null);
    }
}

```

### 6、业务层快速开发

#### 1、快速开发方案

- MP提供有业务层通用接口（IService<T>)与业务层通用实现类（ServiceIpml<M,T>）
- 在通用类基础上做功能重载或功能追加
- 重载时不要覆盖原始操作，闭幕安原始提供的功能丢失

#### 2、具体做法

业务层接口

```java
public interface IBookService extends IService<Book> {
   
}
```

业务层实现类

```java
@Service
public class CBookServiceImpl  extends ServiceImpl<BookDao, Book> implements IBookService {
    
}
// ServiceImpl<BookDao, Book>中的两个泛型分别是数据层接口和实体类
```

测试：注意函数名称

```java
@SpringBootTest
public class CBookServiceTestCase {


    @Autowired
    private IBookService iBookService;


    @Test
    void testGetById(){
        iBookService.getById(15);
    }

    @Test
    void show(){
        System.out.println("CBookServiceTestCase");
    }
    @Test
    void testSelectPage(){
        LambdaQueryWrapper<Book> l = new LambdaQueryWrapper();
        l.like(Book::getName,"spring");
        IPage page = new Page(1,5);

        iBookService.page(page,l);
    }
    @Test
    void testUpdate(){
        Book book = new Book();
        book.setId(21);
        book.setName("IService<T>");
        book.setType("IService<T>");
        book.setDescription("IService<T>");
        iBookService.updateById(book);
    }
}
```

#### 3、自定义函数覆盖问题

```java
    @Override
    boolean save(Book book);
//如果@Override报错代表自定义函数没有重名
```

### 7、表现层开发

####1、表现层接口开发

- 基于Restful进行表现层开发
- 使用PostMan测试接口功能

```java
@RestController
@RequestMapping("books")
public class BookController {

    @Autowired
    private IBookService service;

    @GetMapping
    public List<Book> selectAll(){
        return service.list();
    }

    @PutMapping
    public Boolean update(@RequestBody Book book){
        return service.updateById(book);
    }

    //save操作请求体传json，使用@RequestBody
    @PostMapping
    public Boolean save( @RequestBody Book book){
        return service.save(book);
    }
    @DeleteMapping("{id}")
    public Boolean delete( @PathVariable Integer id){
        return service.removeById(id);
    }

    @GetMapping("{id}")
    public Book getById( @PathVariable Integer id){
        return service.getById(id);
    }
    @GetMapping("{current}/{size}")
    public IPage<Book> selectPage(@PathVariable Integer current,@PathVariable Integer size){
        return service.selectPage(current,size);
    }
}
```

重写了分页方法

```java
    @Autowired
    private  BookDao bookDao;
    @Override
    public IPage<Book> selectPage(Integer current, Integer size) {
        IPage<Book> page = new Page(current,size);
        return bookDao.selectPage(page, null);
    }
```

#### 2、统一返回结果

根据返回的结果标准格式进行统一返回结果类的构建

```json
{
  flag:true.
  data:[
    total,
    current,
    maxPage,
    message[
      具体值
    ]
  ]
}
```

简单的构建一个只有返回状态和返回数据的实体类utils.R

```java
@Data
@NoArgsConstructor
public class R {
    private Boolean flag;
    private Object data;
}
```

```java
@RestController
@RequestMapping("books")
public class BookController {

    @Autowired
    private IBookService service;

    @GetMapping
    public R selectAll(){
        return new R(true,service.list());
    }

    @PutMapping
    public R update(@RequestBody Book book){
        return new R(service.updateById(book));
    }

    //save操作请求体传json，使用@RequestBody
    @PostMapping
    public R save( @RequestBody Book book){
        return new R(service.save(book));
    }
    @DeleteMapping("{id}")
    public R delete( @PathVariable Integer id){
        return new R(service.removeById(id));
    }

    @GetMapping("{id}")
    public R getById( @PathVariable Integer id){
        return new R(true,service.getById(id));
    }

    @GetMapping("{current}/{size}")
    public R selectPage(@PathVariable Integer current, @PathVariable Integer size){
        return new R(true,service.selectPage(current,size));
    }
}
```

### 8、前后端协议联调

- 前后端分离结构设计中的页面归属前端服务器

- 单体工程中的页面位置放在static目录中

- 首页结构

  ```javascript
  //钩子函数，VUE对象初始化完成后自动执行
          created() {
          },
  //具体实现
          methods: {
          }
  ```


#### 1、查询全部功能前端实现

首先在页面加载之前` created() ` 中调用加载函数` getAll()` 

接收后台数据

```javascript
    getAll() {
        console.log("页面加载之前执行getAll")
        //发送异步请求,发送的路径一定是controller中的路径,使用.then接收数据
        axios.get("/books").then((res)=>{
            console.log(res.data)
        })
    }
```

数据处理

使用` prop`进行列绑定,页面在` :data='dataList'` 中接收数据，后台将数据发送给` dataList`变量即可

```javascript
//列表
getAll() {
	console.log("页面加载之前执行getAll")
     //发送异步请求,发送的路径一定是controller中的路径,使用.then接收数据
	axios.get("/books").then((res)=>{
    // console.log(res);
    // res.data是response对象中的data，该对象中还包含
    // status,headers,request,config等诸多参数，res.data.data是
    // 统一返回结果类R中自定义的值
 	this.dataList=res.data.data;
	})
},
```

#### 2、分页查询



#### 3、删除

vue使用 `@click="handleDelete(scope.row)`将行数据封装为一条数据,改行数据封装在row中

```javascript
    handleDelete(row) {
        //参数分别为提示信息，标题和提示类别
        this.$confirm("此操作永久删除当前信息，是否继续","警告",{type: "info"}).then(()=>{
            // console.log(row)
            axios.delete("/books/"+row.id).then((res)=>{
                if (res.data.flag){
                    this.$message.success("删除成功")

                }
                else this.$message.error("删除失败")
            }).finally(()=>{
                this.getAll()
            })
        }).catch(()=>{
            this.$message.info("操作取消")
        })
    },
```

#### 4、新增

新增按钮的前端功能：点击弹出新增页面，清空历史添加记录

将新页面的`:visible.sync="dialogFormVisible"`  绑定的值改为true即可

```javascript
 handleCreate() {
 	this.dialogFormVisible = true
    this.formData ={}
 }
```

将前端的数据发送到后台，新增的请求方式使用post,前端绑定的数据存储在`fromData.column`中,将该数据发送到后台即可。同时关闭窗口并刷新列表，并提示用户添加成功与否

```javascript
    handleAdd () {
        //发送请求到后端，添加是post请求,发送数据在post中传参
        axios.post("/books",this.formData).then((res)=>{
            //判断当前操作是否成功
            if (res.data.flag){
                //关闭弹层
                this.dialogFormVisible = false
                this.$message.success("添加成功")
            }
            else{
                this.dialogFormVisible = false
                this.$message.error("添加失败")
            }
        }).finally(()=>{
            //刷新列表
            this.getAll()
        })
    }
```

取消操作

```javascript
cancel(){
	this.dialogFormVisible = false
	this.$message.info("用户取消操作")
},
```

#### 5、修改

实质上是单个查询+新增的组合

先进行单个查询，与删除相同，使用` scope.row`进行单行数据封装,点击编辑按钮时使用`row.id` 做单行查询并封装近表单formData中。

```javascript
axios.get("/books/"+row.id).then((res)=>{
    if (res.data.flag && res.data.data != null){
        this.formData=res.data.data
        this.dialogFormVisible4Edit=true
    }
    else {
          	this.$message.error("数据查询失败，自动刷新")
     }
 }).finally(()=>{
     this.getAll()
	})
```

调用数据修改接口

```javascript
    handleEdit() {
        axios.put("/books",this.formData).then((res)=>{
            if (res.data.flag){
                this.dialogFormVisible4Edit =false
                this.$message.success("修改成功")
            }
            else {
                this.$message.error("修改失败")
            }
        }).finally(()=>{
            this.getAll()
        })
    }
```

### 9、统一异常处理

自定义异常处理类，在统一返回结果中添加消息字段

```java
//异常处理器
//定义controller层处理器
@RestControllerAdvice
public class MyExceptionAdvice{
    //拦截所有异常信息,异常信息可以分别拦截
    @ExceptionHandler(Exception.class)
    public R doException(Exception e){
        //记录日志
        //通知运维
        //通知开发
        e.printStackTrace();
        return new R(false,e.getMessage());

    }
}
```
































