关于e.pringStackTrace()

tomcat下会输出到catalina.out

==不要用e.pringStackTrace()打日志会出现锁死问题==

 关于e.pringStackTrace()打印日志可能出现的一些问题

 https://blog.csdn.net/qq_28929589/article/details/82495193?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.no_search_link&spm=1001.2101.3001.4242.1

e.toString()：获得异常种类和错误信息

e.getMessage()：获得错误信息

e.printStackTrace()：在控制台打印出异常种类，错误信息和出错位置等

实例：

```java
public class Test {undefined

 /**
  * @param args
  */
 public static void main(String[] args) {
 // TODO Auto-generated method stub
 try {
                    System.out.println(1 / 0);
                } catch (Exception e) {
                    System.out.println(e.toString());
                    System.out.println("-------------------------------------------------");
                    System.out.println(e.getMessage());
                    System.out.println("-------------------------------------------------");
                    e.printStackTrace();
                }
 }

}
```

```bash
输出结果：

java.lang.ArithmeticException: / by zero
-------------------------------------------------
/ by zero
-------------------------------------------------
java.lang.ArithmeticException: / by zero

at com.envision.Test.main(Test.java:11)
```

```java
//将e.printStackTrace使用logger输出
ByteArrayOutputStream stream = new ByteArrayOutputStream();
e.printStackTrace(new PrintStream(stream));
String exception = stream.toString();
logger.error("ES-search-ERROR",exception);
```



```java
//当出现空指针异常等情况时，e.getMessage()有可能为空,因此输出日志的最佳方式是使用logger，第一个参数自定义错误信息提示，第二个参数直接传入完整的堆栈异常信息(包含异常代码行，异常文件，异常提示)
try{
    .....
}catch(Exception e){
    logger.error("这是自定义的异常错误提示",e)
}
```

