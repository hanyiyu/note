# MyBatis入门

**mybatis官方文档：https://mybatis.org/mybatis-3/zh/getting-started.html**



## 搭建环境

### 搭建MySQL数据库

```mysql
CREATE DATABASE `mybatis`;

USE `mybatis`;

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
`id` int(20) NOT NULL,
`name` varchar(30) DEFAULT NULL,
`pwd` varchar(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`name`,`pwd`) values (1,'狂神','123456'),(2,'张三','abcdef'),(3,'李四','987654');
```

### 导入MyBatis相关 jar 包

- GitHub上找

```xml
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis</artifactId>
   <version>3.5.2</version>
</dependency>
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>5.1.47</version>
</dependency>
```

## 创建模块

### 编写MyBatis核心配置文件

- 连接数据库
- 注册每个Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <environments default="development">
       <environment id="development">
           <transactionManager type="JDBC"/>
           <dataSource type="POOLED">
               <property name="driver" value="com.mysql.jdbc.Driver"/>
               <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
               <property name="username" value="root"/>
               <property name="password" value="123456"/>
           </dataSource>
       </environment>
   </environments>
    
    <!-- 注册mapper.xml-->
   <mappers>
       <mapper resource="com/kuang/dao/userMapper.xml"/>
   </mappers>
</configuration>
```

### 编写MyBatis工具类

- 获取sqlSessionFactory对象

  MyBatis是以一个 SqlSessionFactory的实例为核心的。

  **SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder获得。**SqlSessionFactoryBuilder通过XML 配置文件或一个预先配置的Configuration实例来构建出SqlSessionFactory实例。

- **从 SqlSessionFactory 中能获取 SqlSession。SqlSession实例能向数据库执行Sql命令。**SQLSessionFactory工厂生产SqlSession。

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.InputStream;
 
public class MybatisUtils {

   private static SqlSessionFactory sqlSessionFactory;
    
   // 静态代码块
   static {
       try {
           /*获取sqlSessionFactory.
           *  1、通过resources读取配置文件，加载成流
           *  2、SqlSessionFactoryBuilder,加载这个流，构建成工厂sqlSessionFactory
           */
           String resource = "mybatis-config.xml";
           InputStream inputStream = Resources.getResourceAsStream(resource);
           sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
      } catch (IOException e) {
           e.printStackTrace();
      }
  }

   //通过sqlSessionFactory获取SqlSession实例
   public static SqlSession getSession(){
       return sqlSessionFactory.openSession();
  }

}
```

## 编写代码

### 创建实体类

```java
public class User {
   
   private int id;  //id
   private String name;   //姓名
   private String pwd;   //密码
   
   //构造,有参,无参
   //set/get
   //toString()
   
}
```

### 编写Mapper接口类

- 相当于UserDao

```java
import com.kuang.pojo.User;
import java.util.List;

public interface UserMapper {
   List<User> selectUser();
}
```

### 编写Mapper.xml配置文件

- namespace 指定一个Dao/Mapper接口
- 配置文件中namespace中的名称为对应Mapper接口或者Dao接口的完整包名,必须一致！
- 接口实现类由原来的UserDaoImpl转变为一个Mapper配置文件

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.UserMapper">
 <select id="selectUser" resultType="com.kuang.pojo.User">
  select * from user
 </select>
</mapper>
```

## 编写测试类

- Junit 包测试

```java
public class MyTest {
   @Test
   public void selectUser() {
       SqlSession session = MybatisUtils.getSession();
     
       //通过UserMapper接口，返回UserMapper类的实现类mapper
       UserMapper mapper = session.getMapper(UserMapper.class);
       //通过实现类，实现接口里面的方法
       List<User> users = mapper.selectUser();
       //另一种方法，不推荐使用
       //List<User> users = session.selectList("com.kuang.mapper.UserMapper.selectUser");

       for (User user: users){
           System.out.println(user);
      }
       session.close();
  }
}
```

运行测试，成功的查询出来的我们的数据，ok！



**问题说明NullPointerException：Maven静态资源过滤问题,配置文件无法注册**

xml或者properties文件放在java目录下而没有放在resources目录下，配置文件会无法注册。需要在pom文件中加上以下代码：

```xml
<resources>
   <resource>
       <directory>src/main/java</directory>
       <includes>
           <include>**/*.properties</include>
           <include>**/*.xml</include>
       </includes>
       <filtering>false</filtering>
   </resource>
   <resource>
       <directory>src/main/resources</directory>
       <includes>
           <include>**/*.properties</include>
           <include>**/*.xml</include>
       </includes>
       <filtering>false</filtering>
   </resource>
</resources>
```

