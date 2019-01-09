---
layout: post
title: 交替打印 - ReentrantLock
date: 2018-02-01 00:00
tags: [java, algorithm]
---

### 交替打印 - ReentrantLock

```java
public class SynchronizedDemo {


    private static volatile Integer i = 0;

    public static void main(String[] args) {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 2, 2, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1000));

        Lock lock = new ReentrantLock();
        Condition conditionA = lock.newCondition();
        Condition conditionB = lock.newCondition();

        executor.execute(() -> {

            while (i <= 100) {
                lock.lock();
                try {
                    if (i % 2 == 0) {
                        System.out.println("偶数: " + i++);
                    } else {
                        if (i <= 100) {
                            conditionB.signal();
                            conditionA.await();
                        }
                    }
                } catch (Throwable throwable) {

                } finally {
                    lock.unlock();
                }
            }
        });

        executor.execute(() -> {

            while (i <= 100) {
                lock.lock();
                try {
                    if (i % 2 == 1) {
                        System.out.println("奇数: " + i++);
                    } else {
                        if (i <= 100) {
                            conditionA.signal();
                            conditionB.await();
                        }
                    }
                } catch (Throwable throwable) {

                } finally {
                    lock.unlock();
                }
            }
        });
    }
}
```