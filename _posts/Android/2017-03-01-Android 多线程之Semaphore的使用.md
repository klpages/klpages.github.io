---
layout:     post
title:      Android多线程之Semaphore的使用
subtitle:   
date:       2017-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Java
---

Semaphore是一个计数信号量。信号量中维护着一个信号量许可集，它可以控制同时访问特定资源的线程数量。

你可以把他想象成一个有固定车位的停车场，线程可以通过调用acquire()来获取信号量的许可，申请占用停车位，当信号量被许可时，线程可以向下执行，否则线程等待。同时，当车离开时，线程通过release()来释放它所占用的停车位。   
**(ps：acquire()一定不要用于主线程，避免被阻塞)**   
以下代码来自互联网，介绍了Semaphore的简单用法：

```java
public class SemaphoreTest {  
    private static Semaphore semaphore;  
    private static Random rnd = new Random();  
      
    public static void main(String[] args) {  
        semaphore = new Semaphore(3); //初始化许可证数量  
        System.out.println(semaphore.availablePermits()); //当前可用的许可证数量  
        //System.out.println(semaphore.drainPermits()); //获得并返回剩余所有可用的许可证  
        System.out.println(semaphore.getQueueLength()); //获取当前Semaphore对象上正在等待许可证的线程数量  
        System.out.println(semaphore.hasQueuedThreads()); //判断当前Semaphore对象上是否存在正在等待许可证的线程  
        System.out.println(semaphore.isFair()); //是否公平模式FIFO  
          
        for(int i=0; i<10; i++){  
            Thread thread = new Thread(new Runnable() {  
                @Override  
                public void run() {  
                    try{  
                        //boolean b = semaphore.acquire(); //阻塞式获取一个许可，线程可中断  
                        //semaphore.acquireUninterruptibly(); //阻塞式获取一个许可，线程不可中断  
                        boolean b = semaphore.tryAcquire(); //非阻塞式获取一个许可  
                          
                        if(b){  
                            int value = rnd.nextInt(2000);  
                            if(value < 100){  
                                value = 100;  
                            }  
                            System.out.println(Thread.currentThread().getName() + ": " + value);  
                              
                            TimeUnit.MILLISECONDS.sleep(value);  
                              
                            semaphore.release(); //释放一个许可  
                            System.out.println(Thread.currentThread().getName() + " ok");  
                        }  
                          
                    }catch(Exception ex){  
                        ex.printStackTrace();  
                    }  
                }  
            });  
              
            thread.start();  
        }  
          
        System.out.println("ok");  
    }     
}  
```
