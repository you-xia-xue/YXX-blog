# 一个springboot应用
首先我们初始化好的应用部署在IDE内  
我们可以先准备以下的结构   
```
.
├── pojo
├── respository
├── service
├── controller
└── Application.java
```
没有后缀的就是包名（层），这四层逐层调用，这就是一个Springboot架构，我们从pojo逐层实现。

我们通过`Application.java`来启动项目，不过由于没有链接数据库所以会启动失败，所以...

## 链接一个数据库！
当然的,我们需要一个数据库  
此处用MySQL的数据库和驱动  
使用这样的数据库(注意命名规范)
```
└── user_db/
    └── tb_user/
        ├── user_id
        ├── user_name
        ├── user_password
        └── user_email
```
然后就是链接这样的数据库  

在application.properties/yaml文件内配置
``` properties
spring.application.name=包名
#为“jdbc:mysql://localhost:3306/数据库名字”可选属性(?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC)
spring.datasource.url=jdbc:mysql://localhost:3306/数据库名?characterEncoding=utf-8
#这两项就是数据库用户名和密码
spring.datasource.username=root
spring.datasource.password=123456
```
``` yaml
spring:
  application:
    name: 包名
  datasource:
    url: jdbc:mysql://localhost:3306/test?characterEncoding=utf-8
    username: root
    password: 123456
```
通常来讲，链接了之后就可以直接启动了！  
## 建立映射Pojo层 （持久化映射层）
这样就可以声明一个实体类了
``` java
//表示映射的表名
@Table(name="tb_user")
//表示该类作为和表映射的实体类
@Entity
public class User {
    @Id//声明主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//配置主键的生成策略
    @Column(name="user_id")//配置属性和字段的映射关系
    private Integer userId;
    @Column(name = "user_name")
    private String userName;
    @Column(name = "password")
    private String password;
    @Column(name = "email")
    private String email;
        public Integer getUserId() {
        return userId;
    }
    public void setUserId(Integer userId) {
        this.userId = userId;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
}
```
### DTO （数据传输对象）
DTO是客户端传输对象，只作为前端传输而来的载体使用    
整体结构和Pojo一样，所以我们可以把DTO类放在.Pojo.DTO包名里    
不过在业务层里我们需要通过把DTO对象转换成Pojo对象给操作层使用，也可以在对DTO进行额外的方法进行校验  
## Respository层（数据操控）
这里提供直接操作数据库的接口方法，通过Pojo对象输出SQL代码
## Service 层 （业务层）
这里使用数据操作层的接口，进行业务逻辑，将特定结果返回给控制层
## Contorller 层（控制层）
使用Rest风格来处理前端请求
RestFul
GET POST PUT DELET
在这一层，使用注解构建能处理前端请求的接口，在接口内部使用业务层的接口，然后将特定返回给前端
## 小结
我们可以看出来，我们在这一层向下一层返回的结果可有和上一层没有任何联系。即使业务层出了问题，控制层依然可以胡说八道