---
layout: post
title:  "E3try-with-resources"
date:   2019-03-03 15:02:23
comments: false
categories: java
tag: java
description: 《Effective Java, Third Edition》一书英文版已经出版，这本书的第二版想必很多人都读过，号称Java四大名著之一，不过第二版2009年出版，到现在已经将近8年的时间，但随着Java 6，7，8，甚至9的发布，Java语言发生了深刻的变化。                                                        
---
* content
{:toc}
### introduction

《Effective Java, Third Edition》的学习

## 条目9：使用`try-with-resources`语句替代`try-finally`语句

Java类库中包含许多必须通过调用close方法手动关闭的资源。 比如`InputStream，OutputStream`和`java.sql.Connection`。 客户经常忽视关闭资源，其性能结果可想而知。 尽管这些资源中有很多使用`finalizer`机制作为安全网，但finalizer机制却不能很好地工作（条目 8）。
从以往来看，`try-finally`语句是保证资源正确关闭的最佳方式，即使是在程序抛出异常或返回的情况下：

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```


这可能看起来并不坏，但是当添加第二个资源时，情况会变得更糟：
```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
即使是用`try-finally`语句关闭资源的正确代码，如前面两个代码示例所示，也有一个微妙的缺陷------**异常屏蔽**:

```java

/**
 * 异常屏蔽测试
 */
public class Connection implements AutoCloseable {
    public void sendData() throws Exception {
        throw new Exception("send data");
    }
    @Override
    public void close() throws Exception {
        throw new MyException("close");
    }
}
```

```java
public class TryWithResource {
    public static void main(String[] args) {
        try {
            test();
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
    private static void test() throws Exception {
        Connection conn = null;
        try {
            conn = new Connection();
            conn.sendData();
        }
        finally {
            if (conn != null) {
                conn.close();
            }
        }
    }
}
```

运行之后我们发现：

```java
basic.exception.MyException: close
	at basic.exception.Connection.close(Connection.java:10)
	at basic.exception.TryWithResource.test(TryWithResource.java:82)
	at basic.exception.TryWithResource.main(TryWithResource.java:7)
	......
```

好的，问题来了，由于我们一次只能抛出一个异常，所以在最上层看到的是最后一个抛出的异常——也就是`close`方法抛出的`MyException`，而`sendData`抛出的`Exception`被忽略了。这就是所谓的异常屏蔽。由于异常信息的丢失，异常屏蔽可能会导致某些`bug`变得极其难以发现，程序员们不得不加班加点地找bug，如此毒瘤，怎能不除！幸好，为了解决这个问题，从Java 1.7开始，大佬们为`Throwable`类新增了`addSuppressed`方法，支持将一个异常附加到另一个异常身上，从而避免异常屏蔽。那么被屏蔽的异常信息会通过怎样的格式输出呢？我们再运行一遍刚才用`try-with-resource`包裹的`main`方法：

```java
java.lang.Exception: send data

	at basic.exception.Connection.sendData(Connection.java:5)
	at basic.exception.TryWithResource.main(TryWithResource.java:14)
	......
	Suppressed: basic.exception.MyException: close
		at basic.exception.Connection.close(Connection.java:10)
		at basic.exception.TryWithResource.main(TryWithResource.java:15)
		... 5 more
```

可以看到，异常信息中多了一个`Suppressed`的提示，告诉我们这个异常其实由两个异常组成，`MyException`是被`Suppressed`的异常。可喜可贺！

		


 







