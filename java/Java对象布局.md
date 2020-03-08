
#对象布局  

1. new Object()内存布局怎样？占用多少字节？

    * 可视化  
        工具：openjdk
        ```xml
            <dependency>
                <groupId>org.openjdk.jol</groupId>
                <artifactId>jol-core</artifactId>
                <version>0.9</version>
            </dependency>
        ```
        代码：
        ```java
           Object o = new Object();
           System.out.println(ClassLayout.parseInstance(o).toPrintable());
        ```
        结果：
        ```text
            java.lang.Object object internals:
             OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
                  0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
                  4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
                  8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
                 12     4        (loss due to the next object alignment)
            Instance size: 16 bytes
            Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

            解释：  
                前两行->markword 共8字节
                第三行->class pointer 4字节
                最后一行->补齐
        ```
        
2. 对象(普通对象、数组)内存布局  
    * markword                8字节 锁标志位、分代年龄...
    * class pointer 类型指针   8字节(不压缩)／4字节(默认压缩)
    * instance data 实例数据   
    * padding 补齐到能被8整除
        
        
3. new Object() jvm都干了啥？
    java代码
    ```java
        public static void main(String[] args) {
               Object o = new Object();
           }
    ```
    javap 查看机器指令
    ```text
          public static void main(java.lang.String[]);
              Code:
                 //申请内存空间
                 0: new           #2                  // class java/lang/Object
                 //复制栈顶地址，并再将其压入栈顶
                 3: dup
                 //调用构造方法初始化对象
                 4: invokespecial #1                  // Method java/lang/Object."<init>":()V
                 //从操作数栈顶取出 DupTest 对象的引用并存到局部变量表
                 7: astore_1
                 //
                 8: return
    ```
    new对象不是原子操作，指令发生重排序(对象引用先赋值，后初始化)，导致后续线程拿到"不完整"的对象  
    
    java 双重检查的例子  
    ```java
           private volatile Object resource;
       
           public void loadResource() {
               if (resource == null) {
                   synchronized (this) {
                       if (resource == null) {
                           resource = new Object();
                       }
                   }
               }
           }
    ```
    双重检查即减少加锁开销，又避免了多次实例化单利对象，但是如果单利非volatile修饰还是会出现问题  
    指令重排序后，线程2看到了线程1未初始化的对象，直接使用，发生错误

4. java默认参数  
    * 使用PrintCommandLineFlags打印默认参数
   ```commandline
          java -XX:+PrintCommandLineFlags -version
    ```
    * 结果
    ```text
          -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 
          -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers 
          -XX:+UseCompressedOops -XX:+UseParallelGC 
    
          java version "1.8.0_144"
          Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
          Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)

          含义：  
          InitialHeapSize:初始堆大小  
          MaxHeapSize:最大堆大小
          PrintCommandLineFlags:打印默认参数  
          UseCompressedClassPointers:压缩类型指针  8字节->4字节
          UseCompressedOops: oops:ordinary object pointer普通对象指针 8字节->4字节
          UseParallelGC:并行GC  
    ```
    
   
 