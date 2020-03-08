# Java锁

## synchronized
* 锁状态(从上至下锁升级)   
    * 无锁                          用户态    
    * 偏向锁                        用户态  
    * 轻量级锁(自旋锁、自适应锁)       用户态  
    * 重量级锁(操作系统mutex)         内核态  
    
     第一次申请锁       ->    偏向锁，  当前线程指针写到对象头里   
     再有竞争者         ->    自旋锁，  cas+自旋  将Lock record往markwork里贴，贴成功，占锁成功  
     竞争状态特别激烈    ->    重量级锁  自旋超过10次／自旋线程数超过cpu核数一半 jdk1.6+改成自适应     向操作系统申请锁  
     
* 为什么要从轻量级锁升级到重量级锁？  
    轻量级锁不停的自旋，耗费cpu资源；重量级锁竞争进入锁的等待队列，无cpu消耗
    
    
```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class MyFile {
        private int index;
        private File file;
    
        public static void main(String[] args) {
            Object o = new MyFile();
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
            System.out.println();
            synchronized (o) {
                System.out.println(ClassLayout.parseInstance(o).toPrintable());
            }
        }
    }
```
```text
com.gpay.bi.report.models.MyFile object internals:
 OFFSET  SIZE           TYPE DESCRIPTION                               VALUE
      0     4                (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4                (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4                (object header)                           05 c0 00 f8 (00000101 11000000 00000000 11111000) (-134168571)
     12     4            int MyFile.index                              0
     16     4   java.io.File MyFile.file                               null
     20     4                (loss due to the next object alignment)
Instance size: 24 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total


com.gpay.bi.report.models.MyFile object internals:
 OFFSET  SIZE           TYPE DESCRIPTION                               VALUE
      0     4                (object header)                           d0 d8 67 09 (11010000 11011000 01100111 00001001) (157800656)
      4     4                (object header)                           00 70 00 00 (00000000 01110000 00000000 00000000) (28672)
      8     4                (object header)                           05 c0 00 f8 (00000101 11000000 00000000 11111000) (-134168571)
     12     4            int MyFile.index                              0
     16     4   java.io.File MyFile.file                               null
     20     4                (loss due to the next object alignment)
Instance size: 24 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

```
    
