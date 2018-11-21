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


