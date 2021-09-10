# lombok



## springboot中引入lombok依赖
```xml
<!-- pom.xml -->
<dependencies>
     <dependency>
         <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>1.16.20</version>
     </dependency>
</dependencies>
```

## 常用注解：
### @Getter和@Setter： 

```java
/* 手写Getter和Setter */
public class Car {

    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;

    public String getMake() {
        return make;
    }

    public void setMake(String make) {
        this.make = make;
    }

    public String getModel() {
        return model;
    }

    public void setModel(String model) {
        this.model = model;
    }

    public String getBodyType() {
        return bodyType;
    }

    public void setBodyType(String bodyType) {
        this.bodyType = bodyType;
    }

    public int getYearOfManufacture() {
        return yearOfManufacture;
    }

    public void setYearOfManufacture(int yearOfManufacture) {
        this.yearOfManufacture = yearOfManufacture;
    }

    public int getCubicCapacity() {
        return cubicCapacity;
    }

    public void setCubicCapacity(int cubicCapacity) {
        this.cubicCapacity = cubicCapacity;
    }

}
```
可以看到以上代码没有用lombot时手动写Getter和Setter，代码非常繁琐且模式化，等价于下面的代码
```java
//POJO实体类
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Car {
    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

}
```
### @AllArgsConstructor：
数据类通常包含一个构造函数，它为每个成员变量接受参数。IDEA 为 Car 生成的构造函数如下所示：
```java
public class Car {
    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

    public Car(String make, String model, String bodyType, int yearOfManufacture, int cubicCapacity) {
        super();
        this.make = make;
        this.model = model;
        this.bodyType = bodyType;
        this.yearOfManufacture = yearOfManufacture;
        this.cubicCapacity = cubicCapacity;
    }

}
```
以上代码等价于
```java
import lombok.AllArgsConstructor;
@AllArgsConstructor
public class Car {

    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

}

```
### @ToString：
在你的数据类上覆盖 toString方法是有助于记录日志的良好实践。IDEA 为 Car类生成的 toString方法如下所示：

```java
public class Car {

    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

   @Override
    public String toString() {
        return "Car [make="
                          + make
                          + ", model="
                          + model 
                          + ", bo dyType=" 
                          + bodyType 
                          + ",yearOfManufacture="
                          + yearOfManufacture 
                          + ", cubicCapacity=" 
                          + cubicCapacity
                          + "]";
    }

}
```
以上代码等价于
```java
import lombok.ToString;

@ToString
public class Car {

    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

}
```
### @Data
如果你想使数据类尽可能精简，可以使用 @Data 注解。 @Data 是 @Getter、 @Setter、 @ToString、 @EqualsAndHashCode 和 @RequiredArgsConstructor 的快捷方式。
```java

import lombok.ToString;
import lombok.RequiredArgsConstructor;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.Setter;

@ToString
@RequiredArgsConstructor
@EqualsAndHashCode(exclude = {"yearOfManufacture", "cubicCapacity"})
@Getter
@Setter
public class Car {

    private String make;

    private String model;

    private String bodyType;
  
    private int yearOfManufacture;
 
    private int cubicCapacity;
}

```
以上代码等价于
```java
import lombok.Data;

@Data
public class Car {

    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

}
```
### @Slf4j(日志)

Lombok另一个伟大的功能是日志记录器。如果没有 Lombok，要实例化标准的 SLF4J日志记录器，通常会有以下内容：
```java
public class SomeService {
    
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);

    public void doStuff() {
        log.debug("doing stuff....");
    }
}
```
这些日志记录器很沉重，并为每个需要日志记录的类添加了不必要的混乱。值得庆幸的是 Lombok提供了一个为你创建日志记录器的注解。你要做的所有事情就是在类上添加注解，这样就可以了，以上代码等价于
```java
import lombok.Slf4j;

@Slf4j
public class SomeService {
    public void doStuff() {
        log.debug("doing stuff....");
    }
}
```
###@NonNull
对方法参数进行 null 检查通常不是一个坏主意，特别是如果该方法形成的 API被其他开发者使用。虽然这些检查很简单，但是他们可能变得冗长，特别是当你有多个参数时。如下所示，额外的代码无助于可读性，并且可能从方法的主要目的分散注意力。
```java
public void nonNullDemo(Employee employee, Account account) {

    if (employee == null) {
        throw new IllegalArgumentException("Employee is marked @NonNull but is null");
    }

    if (account == null) {
        throw new IllegalArgumentException("Account is marked @NonNull but is null");
    }

    // do stuff
}
```
理想情况下，你需要 null 检查——没有干扰的那种。这就是  `@NonNull` 发挥作用的地方。通过用 `@NonNull` 标记参数，Lombok替你为该参数生成 null 检查。你的方法突然变得更加简洁，但没有丢失那些安全性的 null 检查，以上代码等价于
```java
import lombok.NonNull
public void nonNullDemo(@NonNull Employee employee, @NonNull Account account) {

    // just do stuff

}
```
### @Buildler
```java
import lombok.AllArgsConstructor

@AllArgsConstructor
public class Car {
    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;
    private List<LocalDate> serviceDate;
}
```
假设我们要创建一个 Car，但只想设置 make和 model。在 Car上使用标准的全参数构造函数意味着我们只提供 make和 model并设置其他参数为 null。
 ```java

Car car = new Car("Ford", "Mustang", null, null, null, null);
 ```
以上代码等价于
```java
@Buildler
Car car= Car.builder().make("Ford")
        .model("mustang")
        .build();
```