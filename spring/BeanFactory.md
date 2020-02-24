# BeanFactory

* ### 是什么？  
    Bean工厂/Bean容器  
    
* ### 能干什么？
    1. 实例化Bean
    2. 解决Bean之间依赖（DI）
    
* ### 怎么干？
    1. 实例化Bean  
        将纳入到spring管理的bean（通过xml文件、通过扫描package+注解(@Component/@ManagedBean/@Named)）解析成BeanDefinition注册到
        DefaultListableBeanFactory#beanDefinitionMap中保存，通过BeanFactory#getBean(java.lang.String)
        实例化Bean（单利第一次实例化完成后会保存在DefaultSingletonBeanRegistry#singletonObjects中，原型每次会新创建）
    2. DI  
        在AbstractAutowireCapableBeanFactory#populateBean方法中对依赖的属性进行配置，完成DI

* ### 还能干什么？  
    在Bean的整个生命周期内，对Bean进行各种管理，
    spring将各种阶段通过各种接口暴露给用户，用户只要实现相应的接口
    就可以对bean各个阶段处理  
    
* ### spring暴露出来哪些接口？
    * InstantiationAwareBeanPostProcessor Bean实例化前后动作
    * BeanPostProcessor Bean 初始化前后动作
    * MergedBeanDefinitionPostProcessor
    * Aware(BeanNameAware、BeanClassLoaderAware、BeanFactoryAware)
    * InitializingBean 初始化
    * DisposableBean 销毁
    
* ### BeanFactory中的接口  
![BeanFactory中的接口](./img/BeanFactory-methods.png)


* ### BeanFactory继承体系
![BeanFactory继承体系](./img/BeanFactory.png)
