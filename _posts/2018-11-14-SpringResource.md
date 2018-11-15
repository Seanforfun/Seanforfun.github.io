---
layout: post
title:  "Spring| Resource Object"
date:   2018-11-14 14:08
author: Botao Xiao
categories: Spring
comment: true
description: Spring的核心思想就是通过规范和资源配置将程序员从繁琐的编程业务中解放出来，即使SpringBoot强调注解式配置的今天，仍然无法避免的使用properties文件，所以我单独的将Spring Resource拉出来分为一章进行总结。
---
Spring的核心思想就是通过规范和资源配置将程序员从繁琐的编程业务中解放出来，即使SpringBoot强调注解式配置的今天，仍然无法避免的使用properties文件，所以我单独的将Spring Resource拉出来分为一章进行总结。

### Resource Interface
Spring抽象出了一个Resource接口：
```Java
public interface Resource extends InputStreamSource {
	boolean exists();   文件是否存在，存在的文件一般都是可读的
	default boolean isReadable() {
		return exists();
	}
	default boolean isOpen() {
		return false;
	}
	default boolean isFile() {
		return false;
	}
	URL getURL() throws IOException;
	URI getURI() throws IOException;    Return a URI handle for this resource.
	File getFile() throws IOException;  Return a File handle for this resource, so we can apply stream onto this file handler.
	default ReadableByteChannel readableChannel() throws IOException {  Read file using nio, return a channel in nio.
		return Channels.newChannel(getInputStream());
	}
	long contentLength() throws IOException;     Return length of file
	long lastModified() throws IOException;
	Resource createRelative(String relativePath) throws IOException;
	@Nullable
	String getFilename();
	String getDescription(); Return a description for this resource, to be used for error output when working with the resource.
}
```

### Resource的继承体系
![Imgur](https://i.imgur.com/fLsp0xT.png)

### FileSystemResource
```Java
public class ResourceLoader {
    public static void main(String[] args) {
        
         PathResource方法已经过期了，此处我们用FileSystemResource方法替代，获取resource资源
         
        WritableResource resource = new FileSystemResource(FJavaEE_ProjectMultithreadsrcmainresourceslog4j.properties);
        System.out.println(resource.getFilename());
        InputStream inputStream = null;
        try {
            
              获取资源文件的输入流，从输入流中读取数据。
             
            byte[] buffer = new byte[1024];
            inputStream = resource.getInputStream();
            while (inputStream.read(buffer) != -1){
                System.out.println(new String(buffer, UTF-8));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### ClassPathResource 类路径方式加载文件
1. 使用传统的IO流
```Java
public class ResourceLoader {
    public static void main(String[] args) {
        
         使用ClassPathResource，可以使用resource文件的相对路径。
         
        Resource resource = new ClassPathResource(log4j.properties);
        System.out.println(resource.getFilename());
        InputStream inputStream = null;
        try {
            
              获取资源文件的输入流，从输入流中读取数据。
             
            byte[] buffer = new byte[1024];
            inputStream = resource.getInputStream();
            while (inputStream.read(buffer) != -1){
                System.out.println(new String(buffer, UTF-8));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

2. 使用FileCopyUtil类
```Java
public class ResourceLoader {
    public static void main(String[] args) throws IOException {
        
         使用ClassPathResource，可以使用resource文件的相对路径。
         
        Resource resource = new ClassPathResource(log4j.properties);
        EncodedResource encodedResource = new EncodedResource(resource, UTF-8);
        System.out.println(FileCopyUtils.copyToString(encodedResource.getReader()));
    }
}
```

### Spring下统一的资源读取
Spring中有统一的资源文件读取机制，我们使用ResourcePatternResolver类解析资源文件。该类会根据传入文件的前缀判断当前文件是哪一种资源文件类型进行统一判断。
```Java
public class ResourceLoader {
    public static void main(String[] args) throws IOException {
        ResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
         读取文件路径下所有的properties文件
        Resource[] resources = patternResolver.getResources(classpath.properties);
        for(Resource resource  resources)
            System.out.println(resource.getFilename());
    }
}
```

匹配文件的正则表达式
```
 单个字符
： 任意长度的字符
： 匹配多层路径
```

### Reference
1. [精通Spring4.x](httpsbook.douban.comsubject26952826)