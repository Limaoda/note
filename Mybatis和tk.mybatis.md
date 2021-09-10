# Mybatis和tk.mybatis

### Mybatis

##### 引入依赖：

```xml
<!-- pom.xml -->
<dependencies>  
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.2.0</version>
  </dependency>
</dependencies>
```

##### 基本使用：

@Mapper注释用来表示该接口类的实现类对象交给mybatis底层创建，然后交由Spring框架管理。

```java
//UserDao.java(Dao)
package com.example.web_demo.mapper;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Update;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Select;
import org.springframework.data.repository.query.Param;
import com.example.web_demo.bean.User;

import java.util.List;

@Mapper
public interface UserDao{

    /**
     * 用户数据新增
     */
    @Insert("insert into users(id,username,account,password,phone) values (#{id},#{username},#{account},#{password},#{phone})")
    void addUser(User user);

    /**
     * 用户数据修改
     */
    @Update("update users set username=#{username},account=#{account},phone=#{phone},password=#{password} where id=#{id}")
    void updateUser(User user);

    /**
     * 用户数据删除
     */
    @Delete("delete from users where id=#{id}")
    void deleteUser(int id);

    /**
     * 根据用户名称查询用户信息
     *
     */
    @Select("SELECT id,username,account,password,phone FROM users where username=#{username}")
    User findByName(@Param("username") String username);

    /**
     * 登录查询
     *
     */
    @Select("SELECT id,username,account,password,phone FROM users where account=#{account} AND password=#{password}")
    User findByAccountAndPwd(@Param("account,password") String account,String password);

    /**
     * 查询所有
     */
    @Select("SELECT id,username,account,password,phone FROM users")
    List<User> findAll();
}
```



```java
//UserServiceImpl.java(Server)
package com.example.web_demo.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.web_demo.bean.User;
import com.example.web_demo.mapper.UserDao;
import java.util.List;

//service接口的实现类
@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private UserDao userDao;
    @Override
    public boolean addUser(User user) {
        try{
            userDao.addUser(user);
            return true;
        }catch(Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public boolean updateUser(User user) {
        try{
            userDao.updateUser(user);
            return true;
        }catch(Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public boolean deleteUser(int id) {
        try{
            boolean res = userDao.deleteUser(id);
            System.out.println(res);
            return true;
        }catch(Exception e){
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public User findUserByName(String usernmae) {
        User res = userDao.findByName(usernmae);
        System.out.println(res.toString());
        return res;
    }
    @Override
    public User findUserByAccountAndPwd(String account, String password) {
        User res = userDao.findByAccountAndPwd(account,password);
        System.out.println(res.toString());
        return res;
    }
    @Override
    public List<User> findAll() {
        return userDao.selectAll();
    }
}

```

可以看到上面直接调用了userDao接口的方法，我们并没有去写userDao接口的实现类，但由于在接口上添加了@Mapper这个注解，spring已经自动帮我们生成了一个代理实现类，因此我们可以直接使用接口中的方法，我们可以调试一下查看spring帮我们代理的实现类

```java
//UserServiceImpl.java(Server)
@Override
    public boolean deleteUser(int id) {
        try{
            boolean res = userDao.deleteUser(id);
            System.out.println(res);  //这一行打一个断点
            return true;
        }catch(Exception e){
            e.printStackTrace();
            return false;
        }
    }

//打断点进入debug后，鼠标移动到userDao这个变量出现一个包含userDao=$Proxy的界面，这个$Proxy就是Mybatis为我们创建的实现类，Proxy是代理的意思后面跟的编号是随机的，也就是UserDao这个接口的实现类是一个代理对象。这个代理对象被存储到spring容器中通过@Autowired自动注入到这个接口属性对象。所以我们才能调用这个方法。
```

##### 常用注解：

Mybatis的注解位于org.apache.ibatis.annotations这个包下面。常用的注解如下

###### 普通映射：

**Select**：映射查询的SQL语句

 ```java
     @Select("SELECT id,username,account,password,phone FROM users where username=#{username}")
     User findByName(@Param("username") String username);
 ```

**Insert**：映射插入的SQL语句

```java
    @Insert("insert into users(id,username,account,password,phone) values (#{id},#{username},#{account},#{password},#{phone})")
    void addUser(User user);
```

**Update**：映射更新的SQL语句

```java
    @Update("update users set username=#{username},account=#{account},phone=#{phone},password=#{password} where id=#{id}")
    void updateUser(User user);
```

**Delete**：映射删除的SQL语句

```java
    @Delete("delete from users where id=#{id}")
    void deleteUser(int id);
```

###### 结果集映射：

作用：当**数据库字段名与实体类对应的属性名不一致**时，可以使用`@Results`映射来将其对应起来。**column**为数据库字段名，**porperty**为实体类属性名，**jdbcType**为数据库字段数据类型，**id**为是否为主键。

**@Result**：

|  **属性**   |                           **描述**                           | 值类型  |
| :---------: | :----------------------------------------------------------: | ------- |
|     id      |                        是否为主键字段                        | boolean |
|   column    |                      数据库查询出的列名                      | string  |
|  property   |                       需要装配的属性名                       | string  |
|     one     |            需要使用的@One注解 `@Result(one=@One)`            | other   |
|    many     |          需要使用的@Many注解 `@Result(many=@many)`           | other   |
|  jdbcType   |                      数据库字段数据类型                      | other   |
|  javaType   | 一个完整的类名，或者是一个类型别名。如果你匹配的是一个JavaBean，那MyBatis 通常会自行检测到。然后，如果你是要映射到一个HashMap，那你需要指定javaType 要达到的目的。 | other   |
| typeHandler | 使用这个属性可以覆写类型处理器。这项值可以是一个完整的类名，也可以是一个类型别名。 | other   |

**@Results**：多个结果（Result）映射列表

基本用法如下：

首先，我们修改一下数据库users表的字段名

修改前[^注释]

![image-20210820143240056](C:\Users\HuQiaoDong\AppData\Roaming\Typora\typora-user-images\image-20210820143240056.png)

修改后[^注释]

![image-20210820144618799](C:\Users\HuQiaoDong\AppData\Roaming\Typora\typora-user-images\image-20210820144618799.png)

```java
//UserDao.java(Dao)
@Select("SELECT id,user_name,account,password,phone FROM users where user_name=#{username}")
@Results({
    @Result(column="id", property="id", jdbcType=JdbcType.INTEGER, id=true),
    @Result(column="user_name", property="username", jdbcType=JdbcType.VARCHAR),
    @Result(column="account", property="account", jdbcType=JdbcType.CHAR),
    @Result(column="password", property="password", jdbcType=JdbcType.CHAR),
    @Result(column="phone", property="phone", jdbcType=JdbcType.VARCHAR),
})
User findByName(@Param("username") String username);
```

如上所示的数据库字段名**user_name**与实体类属性名**username**，就通过这种方式建立了映射关系。名字相同的可以省略。

> @param：当映射器方法需要多个参数时，这个注解可以被应用于映射器方法参数来给每个参数取一个名字。否则，多参数将会以它们的顺序位置和SQL语句中的表达式进行映射，这是默认的，使用@param("username")，SQL中的参数应该被命名为#{username}



###### 关系映射：

**@One**(一对一查询)：代替了 `<association>` 标签，是多表查询的关键，在注解中用来指定子查询返回单一对象。

1. 注解属性

- select: 指定多表查询的sqlmapper
- fetchType: 会覆盖全局的配置参数lazyLoadingEnable

2. 使用格式

```java
@Result(column="", property="",one=@One(select=""))
```

使用实例：

```java
//UserDao.java(Dao)
@Select("SELECT id,user_name,account,password,phone FROM users where user_name=#{username}")
@Results({
    @Result(column="id", property="id", jdbcType=JdbcType.INTEGER, id=true),
    @Result(column="user_name", property="username", jdbcType=JdbcType.VARCHAR,one=@One(select="com.example.web_demo.mapper.UserWordsDao.findWordsByName")),
    @Result(column="account", property="account", jdbcType=JdbcType.CHAR),
    @Result(column="password", property="password", jdbcType=JdbcType.CHAR),
    @Result(column="phone", property="phone", jdbcType=JdbcType.VARCHAR),
})
User findByName(@Param("username") String username);
```





### tk.mybatis

##### 引入依赖：

```xml
<!-- pom.xml -->
<dependencies>       
  <dependency>
       <groupId>tk.mybatis</groupId>
       <artifactId>mapper</artifactId>
       <version>4.1.5</version>
  </dependency>
  <dependency>
       <groupId>tk.mybatis</groupId>
       <artifactId>mapper-spring-boot-starter</artifactId>
       <version>2.1.4</version>
  </dependency>
</dependencies>
```

```java
//UserDao.java(Dao)
package com.example.web_demo.mapper;
//import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.type.JdbcType;
import com.example.web_demo.bean.User;
import tk.mybatis.mapper.common.Mapper;
import java.util.List;

//UserDao接口类需要继承tk.mybatis中的Mapper类，但因为需要在实体类中使用到Mybatis的实体类代理生成，因此还是需要在接口类上加入Mybatis本身的@Mapper注解，两者有冲突，因此需要使用@org.apache.ibatis.annotations.Mapper这种方式
@org.apache.ibatis.annotations.Mapper
public interface UserDao extends Mapper<User>{

}
```

现在UserDao接口已经继承了tk.mybatis的Mapper类，我们还需要去数据库映射实体类中指定主键和查询表，然后就可以使用这个Mapper类的很多方法进行CRUD

```java
//User.java(POJO)
package com.example.web_demo.bean;

import javax.persistence.Id;
import javax.persistence.Table;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@ToString
@Getter
@Setter
//@Table指定数据库表
@Table(name = "users") 
public class User {
    /**
     * 编号 @Id注解指定为主键
     */
    @Id
    private int id;
    /**
     * 用户名
     */
    private String username;
    /**
     * 账号
     */
    private String account;
    /**
     * 密码
     */
    private String password;
    /**
     * 手机号
     */
    private String phone;

}
```



```java
//UserServerImpl.java(Server)
package com.example.web_demo.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.web_demo.bean.User;
import com.example.web_demo.mapper.UserDao;
import java.util.List;

@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private UserDao userDao;
    @Override
    public boolean addUser(User user) {
        try{
            //insertSelective这个方法是tk.mybatis提供的插入方法
            userDao.insertSelective(user);;
            return true;
        }catch(Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public boolean updateUser(User user) {
        try{
            //updateByPrimaryKey这个方法是tk.mybatis提供的更新方法，根据主键更新表
            userDao.updateByPrimaryKey(user);
            return true;
        }catch(Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public boolean deleteUser(int id) {
        try{
            userDao.deleteByPrimaryKey(id);
            return true;
        }catch(Exception e){
            e.printStackTrace();
            return false;
        }
    }
    @Override
    public User findUserByName(String usernmae) {
        User res = userDao.findByName(usernmae);
//        if (res.getPassword())
        System.out.println(res.toString());
        return res;
    }
    @Override
    public User findUserByAccountAndPwd(String account, String password) {
        User res = userDao.findByAccountAndPwd(account,password);
//        if (res.getPassword())
        System.out.println(res.toString());
        return res;
    }
    @Override
    public List<User> findAll() {
        return userDao.selectAll();
    }
}
```

