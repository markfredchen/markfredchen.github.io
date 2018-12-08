---
title: mybatis-plus-note
tags:
---


## 启动流程

### Mapper
1. `@MapperScan`
```java
@Import(MapperScannerRegistrar.class)
public @interface MapperScan {
  ...
}
```
2. `MapperScannerRegistrar`
  1. org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
  2. registerBeanDefinitions
    1. AnnotationMetadata importingClassMetadata
      - StandardAnnotationMetadata
        - annotations
          - `@SpringBootApplication`
          - `@MapperScan`
        - introspectedClass -> SpringBootApplication class
    2. BeanDefinitionRegistry registry
      - org.springframework.beans.factory.support.DefaultListableBeanFactory
        - org.springframework.context.annotation.internalConfigurationAnnotationProcessor
        - org.springframework.context.annotation.internalAutowiredAnnotationProcessor
        - org.springframework.context.annotation.internalRequiredAnnotationProcessor
        - org.springframework.context.annotation.internalCommonAnnotationProcessor
        - org.springframework.context.event.internalEventListenerProcessor
        - org.springframework.context.event.internalEventListenerFactory
        - myBatisPlusDemoApplication
        - org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory
        - productService
        - runner
        - org.springframework.boot.autoconfigure.AutoConfigurationPackages
3. a
