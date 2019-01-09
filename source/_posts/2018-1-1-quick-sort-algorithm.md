---
layout: post
title: Quick sort algorithm
date: 2018-01-01 00:00
tags: [java, algorithm]
---

### 快速排序算法

```java
public class QuickSort {

    public static void main(String[] args) {

        int[] items = {1, 3, 5, 6, 7, 9, 1, 2, 3};

        doQuickSort(items, 0, items.length - 1);
        Util.print(items);
    }

    private static void doQuickSort(int[] items, int start, int end) {

        if (start < end) {
            int middle = splitMiddle(items, start, end);

            if (middle - 1 > start) {
                doQuickSort(items, start, middle - 1);
            }

            if (end > middle + 1) {
                doQuickSort(items, middle + 1, end);
            }
        }
    }

    private static int splitMiddle(int[] items, int start, int end) {
        int data = items[start];

        while (start < end) {

            while (start < end && items[end] > data) {
                end --
                ;
            }

            items[start] = items[end];

            while (start < end && items[start] <= data) {
                start ++
                ;
            }
            items[end] = items[start];

        }

        items[start] = data;
        return start;
    }


}
```