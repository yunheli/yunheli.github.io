---
layout: post
title: Insert sort algorithm
date: 2018-01-01 00:00
tags: [java, algorithm]
---

### 插入排序算法

```java
public class InsertSort {

    public static void main(String[] args) {
        int[] items = {1, 3, 5, 6, 7, 9, 1, 2, 3};
        doInsertSort(items);

        Util.print(items);
    }

    private static void doInsertSort(int[] items) {

        for (int i = 1; i < items.length; i++) {

            int current = items[i];
            if(items[i - 1] < items[i]) {
                continue;
            }

            int j = 0;
            int flag = 0;
            for (j = i - 1; j >= 0; j--) {

                if(items[j] > current) {
                    items[j + 1] = items[j];
                    flag = j;
                } else {
                    break;
                }
            }

            items[flag] = current;
        }
    }
}
```