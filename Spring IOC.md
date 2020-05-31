# IOC容器工作原理

## IOC容器作用。

1.初始化一个bean工程

```java
this.beanFactory = new DefaultListableBeanFactory();

//缓存beanDefinition key:id(beanName)
Map<String,BeanDefinition> beanDefinitionMap;

//bean的定义，装载bean的属性
BeanDefinition

//Bean的注册器
BeanDefinitionRegistry 
    
//注册Bean
registerBeanDefinition(String beanName, BeanDefinition beanDefinition)

//Bean工厂的后置处理器，解析注解
BeanFactoryPostProcessor
```

2.注册bean到beanDefinitionMap

3.实例化bean

```java
//Bean的后置处理器
BeanPostProcessor
    
//实例化非懒加载的单例bean
finishBeanFactoryInitialization(beanFactory);
```

创建bean之前判断，先从singletonObjects(单例对象池)中获取

```java
//缓存单例对象实例
Map<String, Object> singletonObjects 
getSingleton(beanName)

//单例工厂
singletonFactory
```



# AOP原理

https://start.spring.io