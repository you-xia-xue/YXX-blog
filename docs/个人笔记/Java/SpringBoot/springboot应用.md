# 一个springboot应用
首先我们初始化好的应用部署在IDE内，我们通过`Application.java`来启动项目  
不过由于没有链接数据库所以会启动失败，所以...

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
``` 
#properties
spring.application.name=包名
#为“jdbc:mysql://localhost:3306/数据库名字”可选属性(?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC)
spring.datasource.url=jdbc:mysql://localhost:3306/数据库名?characterEncoding=utf-8
#这两项就是数据库用户名和密码
spring.datasource.username=root
spring.datasource.password=123456

//yaml
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

### DTO层 （传输层）
DTO是客户端传输层，只作为前端传输而来的载体使用
我们需要通过把DTO对象转换成Pojo对象来
## Service 层 （业务层）
## Contorller 层（控制层）
使用Rest风格来处理前端请求
RestFul
GET POST PUT DELET