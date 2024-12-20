---
layout: post
title:  "Java Service Provider Interface"
date:   2016-11-15 14:42:21 +0800
---

# 标准定义权

说来说去，还是“标准”的解释权在谁手里面，要么在服务调用方，要么在服务提供方。

标准在服务提供方没什么好说的，服务提供方提供什么内容，调用方用什么内容，常见的Web HTTP请求就是此类。

标准在服务调用方手里，大致可以再归纳归纳：

1、同步调用，接口调用方按自己定义的标准将内容给到服务提供方

- 要么都给（比如轮询服务提供方）

- 要么根据条件只给某些个服务提供方

一个接口多个实现类（DI、Observer Design Pattern），SPI，通过HTTP调用不同的网关接口都属于此类。根据条件只给某些个服务提供方想比于都给服务提供方需要额外的协调器，比如 Spring 的某些用来做DI的组件、SPI里的 ServiceLoader、Observer Design Pattern里转发各类 event 给 event listener 的 event manager。

2、异步调用，常见的MQ组件属于此类，消息发出去谁爱消费谁消费，发什么内容也是生产者决定

# SPI说明及样例

回到SPI，语法层面的规定就是classpath ``META-INF/services``目录里，服务提供方要有个以接口全名称为文件名，内容为实现类全名称的文件，多个服务实现类就多行；最后服务调用方通过 ``ServiceLoader`` 去加载。

看个简单例子（https://github.com/c4rlosmonteiro/image-processor-service-spi），项目结构：

```
src/main
	java/com/github/c4rlosmonteiro/imageprocessorservicespi
		provider
			JPGImageProcessorProvider.java
			PNGImageProcessorProvider.java
		service
			ImageProcessorService.java
		BasicImageType.java
		ImageProcessorApp.java
		Main.java
resources/META-INF/services
	com.github.c4rlosmontero.imageprocessorservicespi.service.ImageProcessorService
```

关键代码

一个Service定义

```
public interface ImageProcessorService {
    String getType();
    void process();
}
```

两个Service Provider

```
public class JPGImageProcessorProvider implements ImageProcessorService {
    @Override
    public String getType() {
        return BasicImageType.JPG.name();
    }

    @Override
    public void process() {
        System.out.println("Processing JPG image");
    }
}
```

```
public final class PNGImageProcessorProvider implements ImageProcessorService {
    @Override
    public String getType() {
        return BasicImageType.PNG.name();
    }

    @Override
    public void process() {
        System.out.println("Processing PNG image");
    }
}
```

协调器 Service Loader

```
public class ImageProcessorApp {

    private final List<ImageProcessorService> imageProcessorProviders;

    public ImageProcessorApp() {
        final ServiceLoader<ImageProcessorService> imageProcessorServiceLoader = ServiceLoader.load(ImageProcessorService.class);
        final Iterator<ImageProcessorService> imageProcessorServiceIterator = imageProcessorServiceLoader.iterator();

        this.imageProcessorProviders = new ArrayList<>();

        while (imageProcessorServiceIterator.hasNext()) {
            imageProcessorProviders.add(imageProcessorServiceIterator.next());
        }
    }

    public void processImages(final String type) {
        for (final ImageProcessorService imageProcessorProvider : imageProcessorProviders) {
            if (Objects.equals(imageProcessorProvider.getType(), type)) {
                imageProcessorProvider.process();
                return;
            }
        }

        throw new RuntimeException("No provider found to process image type: " + type
                + ". Create a new provider for it!");
    }
}
```

to Run

```
public class Main {
    public static void main(final String [] args) {
        final ImageProcessorApp imageProcessorApp = new ImageProcessorApp();
        imageProcessorApp.processImages(BasicImageType.PNG.name());
        imageProcessorApp.processImages(BasicImageType.JPG.name());
        imageProcessorApp.processImages("MY_CUSTOM_OR_NOT_SUPPORTED_IMAGE_TYPE");
    }
}
```

上面只是个样例，一般Service Provider的实现会在Jar包里提供

# 实例 JDBC Driver

Service 定义

```
Location: JDK Jar

interface java.sql.Driver
```

MySQL Service Provider

```
Location: mysql-connector-java-version.jar

File: META-INF/services/java.sql.Driver
Content: com.mysql.cj.jdbc.Driver
```

Service Loader

```
Location: JDK Jar java.sql.DriverManager#loadInitialDrivers

ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
Iterator<Driver> driversIterator = loadedDrivers.iterator();
try{
    while(driversIterator.hasNext()) {
        driversIterator.next();
    }
}
```

这个load基本上没做啥，拉起了实现类后，实现类里通过 static block 将自己做了register

```
com.mysql.cj.jdbc.Driver

static {
    try {
        java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
    }
}
```





