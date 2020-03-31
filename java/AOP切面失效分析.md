###AOP失效分析

* spring aop代理类产生方式
    1. jdk proxy 产生实现类
    2. cglib     产生子类
    
* aop处理的类某些方法会失效(@Transactional失效、自定义切面失效...)
    
    eg.
    ```java
      class Test{
          void a(){
            b();
          }
          @AspectAnnotation
          void b(){
            
          }
      }  
    
    ```
    
    ```java
      public static void main(String[] args){
          test.a();//AspectAnnotation对应的切面逻辑失效
          test.b();//AspectAnnotation对应的切面逻辑有效
      }
    ```
* 失效分析  
    输出代理类的字节码
    ```java
         System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY,"/Users/tangmingjian/Desktop");

    ```
    a方法里调用b，为什么会失效？  
    因为通过字节码可以看出a调用的是this.b(),是目标类的b方法，并非代理增强过的b方法；
    而直接调用b方法，就是调用的增强处理的b所以有效
    
* 解决方案  
    1. 设置exposeProxy=true，暴露代理类，通过代理类来调用 ((Test)AopContext.currentProxy()).a();
    2. 避免同类调用且被调用方法是切面处理方法，将方法拆分到两个类去
    
    
* 总结
  切面逻辑在非直接外部调用处，一定会失效