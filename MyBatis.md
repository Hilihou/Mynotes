# 简介

JDBC--Dbutils--JdbcTemplate

持久化层的框架，整体的解决方案

![](https://tva1.sinaimg.cn/large/008i3skNly1gtla7g8uh5j610c086aav02.jpg)

Hibernate：全自动ORM框架：旨在消除sql，HQL定制sql（学习成本大）

希望：sql交给开发人员来写

![](https://tva1.sinaimg.cn/large/008i3skNly1gtlacmrkf6j611606u74y02.jpg)

全自动框架



---

Mybatis流程：

![image-20210818220357365](https://tva1.sinaimg.cn/large/008i3skNly1gtlajm4vibj614e0bqjsa02.jpg)

sql与java代码分离，sql是开发人员控制

只要掌握sql就行！

半自动框架，轻量级

---

# HelloWorld（Old version）

该方法为旧版本，存在一些问题，仅供学习，实际不用

**Building SqlSessionFactory from XML**

## 一、全局配置文件

有数据源和一些运行环境的信息

接下来创建SqlSession对象

在官方文档的xml文件中修改如下：

``mybatis-config.xml``

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
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test?useSSL=false&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456789"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

用于配置数据库连接，并且和mapper相关联

问题：出现uri报红解决方案

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gtldvhds4fj312l0u041k.jpg" alt="image-20210818235837065" style="zoom:50%;" />

----

## 二、sql映射文件

``StudentMapper.xml``

配置了每一个sql和一些sql的封装规则

问题：id为红色的时候估计是插件的问题，卸载掉就好了

resultType要写bean类的全类名

用于编写sql语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.iweb.StudentMapper">
<!--名称空间namespace 自定义，最好有意义-->
<!-- #{id}从传递过来的参数中取出id值-->
    <select id="selectStudent" resultType="com.iweb.Student">
        select * from student where sno = #{id}
    </select>

</mapper>
```

---

## 三、映射文件在全局配置文件中注册

## 四、写代码

1. 根据全局配置文件得到sqlSessionFactory
2. 使用sqlSessionFactory获取到SqlSession对象，并进行增删改查，一个SqlSession代表一次与数据库的会话，用完一定要关闭。
3. 使用sql的唯一标识来告诉MyBatis执行哪个sql，sql都是保存在配置文件中的。

在测试类中：

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = sqlSessionFactory.openSession();
//这边最好用名称空间+id值来保证唯一
Student o = session.selectOne("com.iweb.StudentMapper.selectStudent", 1);
System.out.println(o);
session.close();
```

出现问题：如果找不到资源可以将conf文件夹设置为资源文件夹

目录结构：<img src="/Users/hillihou/Library/Application Support/typora-user-images/image-20210819001810512.png" alt="image-20210819001810512" style="zoom: 33%;" />

---

# 接口式编程（推荐使用）

1. 首先创建一个接口

```java
package com.iweb;
public interface StudentMapper {
    public Student getStudentBySno(Integer sno);
}
```

2. 然后在映射文件中的namespace中写接口的全类名就可以绑定接口了
3. sql语句的id绑定接口中的方法名

测试代码：

```java
@Test
public void test2(){
    //1.得到sqlSessionFactory
    SqlSessionFactory sqlSessionfactory = getSqlSessionfactory();
    //2.获取sqlSession对象
    SqlSession sqlSession = sqlSessionfactory.openSession();
    //3.获取接口的实现类对象,自动实现，更强的类型检查，明确的返回值
    //会为接口创建一个代理对象，代理对象会执行增删改查的方法
    //用接口将DAO层的规范和实现分离开来，方便今后代码的维护
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    //3.1查询
    Student student = mapper.getStudentBySno(1);
    System.out.println(student);
}
```

原生：DAO------DAOImpl

MyBatis：Mapper------xxMapper.xml

---



- sqlSession是数据库的一次会话，用完必须关闭

- sqlSession和connection一样是非线程安全的，每次都应该去获取新的

- mapper没有实现类，但是mybatis会为接口创建一个代理类对象：

（将接口和xml文件进行绑定）

​		sqlSession.getMapper(StudentMapper.class);

返回的是接口的对象，拿到的是代理类的对象，已经帮忙实现了

- 两个重要的配置文件：

 	1. Mybatis全局配置文件（也可以不用，官方文档有没有全局文件连接的方法）
 	2. sql映射文件（非常重要‼️），保存了每一个sql语句的映射信息，将sql抽取了出来

​	