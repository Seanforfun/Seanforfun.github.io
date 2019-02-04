---
layout: post
title:  "Spring| Annotations"
date:   2018-11-15 19:43:59
author: Botao Xiao
categories: Spring
comment: true
description: 总结Spring中出现的所有注解。
---
| Annotation Name | Region | Description | Comment |
| :------ | :------ | :------ | :------ |
| @Bean | METHOD, ANNOTATION_TYPE | Indicates that a method produces a bean to be managed by the Spring container. | |
| @Component | ANNOTATION_TYPE | Indicates that an annotated class is a "component". Such classes are considered as candidates for auto-detection when using annotation-based configuration and classpath scanning. | |
| @Configuration | TYPE | Indicates current class is a configuration class. Auto-scanned by Spring container. | |
| @Scope | TYPE, METHOD | When used as a type-level annotation in conjunction with indicates the name of a scope to use for instances of the annotated type.| |
| @ResponseBody | Method | Normally, a response from a controller will be considered as a redirect path, be adding @ResponseBody, the returned results are considered as "Object", most likely, JSON | We use this annotation for AJAX. |
| @Mapper | TYPE | @Mapper normally works on a interface for MyBatis, the methods declared in the interface will load their mybatis CRUD annotations to load the SQL. | Work for MyBatis on interface. |
| @Transactional | METHOD, TYPE | @Transactional works on Class or method, it will ensure the safety of transaction of SQL database. | Transactional works on multiple sql commands and will roll back when error happens in current transaction. |
| @Valid | METHOD, FIELD, CONSTRUCTOR, PARAMETER, TYPE_USE  | Majorly, @Valid is used on spring auto binding parameters. It will follow the class given in @Constraint, if return false in the isValid method, it will throw a BindException. | We can create our own validation annotations by following the rules and use globalExceptionHandler to deal with the exceptions.|
| @CookieValue | PARAMETER | Cookie value is used to bind value in Cookie, we can specify the cookie name and bind the value to parameters. | |



