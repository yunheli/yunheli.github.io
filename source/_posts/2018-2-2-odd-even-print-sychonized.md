---
layout: post
title: 交替打印 - synchronized
date: 2018-02-02 00:00
tags: [java, algorithm]
---

### 交替打印 - synchronized

```java
public class LockDemo {

    private final static int TOTAL = 100;
    private static volatile Integer i = 0;

    public static void main(String[] args) {

        Object picker = new Object();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 2, 2, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));

        executor.execute(() -> {
            while( i <= TOTAL) {

                synchronized(picker) {

                    if(i % 2 == 0) {
                        System.out.println("偶数：" + i++);
                    } else {
                        picker.notifyAll();
                        if(i <= TOTAL) {
                            try {
                                picker.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
        });

        executor.execute(() -> {
            while( i <= TOTAL) {

                synchronized(picker) {

                    if(i % 2 == 1) {
                        System.out.println("奇数：" + i++);
                    } else {
                        picker.notifyAll();
                        if(i <= TOTAL) {
                            try {
                                picker.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
        });

    }
}

```