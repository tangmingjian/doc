# BeanFactory

* ### 是什么？  
    Bean工厂/Bean容器  
    
* ### 能干什么？
    1. 实例化Bean
    2. 解决Bean之间依赖(DI)
    
* ### 怎么干？
    1. 实例化Bean  
        将纳入到spring管理的bean(通过xml文件、通过扫描package+注解(@Component/@ManagedBean/@Named))解析成BeanDefinition注册到
        DefaultListableBeanFactory#beanDefinitionMap中保存，通过BeanFactory#getBean(java.lang.String)
        实例化Bean(单利第一次实例化完成后会保存在DefaultSingletonBeanRegistry#singletonObjects中，原型每次会新创建)
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
  
   - 14个方法  
       + 3个getBean(String name) 按名字查找，都会转到AbstractBeanFactory#doGetBean中处理，带类型的会匹配类型，带参数的会在创建时使用参数
       + 2个getBean(Class type) 按类型查找，都会转到DefaultListableBeanFactory#resolveBean处理，如果找到一个会转到按名字查找，
             如果找到多个，先找@Primary的，再找javax.annotation.Priority优先级最高的，最后还是转到按名字查找
       + 2个getBeanProvider(Class type) 获取一个bean的提供者，获取bean的地方仍然是会转到DefaultListableBeanFactory#resolveBean
       + 1个containsBean(String name) 判断name是否存在(在DefaultSingletonBeanRegistry#singletonFactories单利缓存中或者在DefaultListableBeanFactory#beanDefinitionMap缓存中)
       + 1个isSingleton(String name) 判断是否是单利(1.单利缓存中是否存 2.bd缓存中是否是)
       + 1个isPrototype(String name) 判断是否原型(bd缓存中是否是)
       + 2个isTypeMatch(String name, Class<?> typeToMatch) 判断类型和名字是否匹配
       + 1个getType(String name) 获取name对应bean的类型
       + 1个getAliases(String name) 获取别名

* ### BeanFactory继承体系
![BeanFactory继承体系](./img/BeanFactory.png)

* ### BeanFactory继承体系分析
    - BeanFactory儿子接口
        + ListableBeanFactory 可以枚举所有Bean实例
        + AutowireCapableBeanFactory 可装配的能力
        + HierarchicalBeanFactory 可获取父容器

