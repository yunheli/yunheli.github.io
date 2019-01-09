---
layout: post
title: 交替打印 - Semaphore
date: 2018-02-03 00:00
tags: [java, algorithm]
---

### 交替打印 - Semaphore

```java
public class SempheDemo {

    private static volatile Integer i = 0;
    private final static Integer TOTAL = 100;

    public static void main(String[] args) throws InterruptedException {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 2, 2, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));

        Semaphore a = new Semaphore(1);
        Semaphore b = new Semaphore(1);
        a.acquire();

        executor.execute(() -> {
            while (i <= TOTAL) {

                if(i % 2 == 0) {
                    try {
                        System.out.println("偶数: "+ i++);
                        b.acquire();
                        a.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        });

        executor.execute(() -> {
            while (i <= TOTAL) {

                if(i % 2 == 1) {
                    try {
                        System.out.println("奇数: "+ i++);
                        a.acquire();
                        b.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        });
    }
}
```
