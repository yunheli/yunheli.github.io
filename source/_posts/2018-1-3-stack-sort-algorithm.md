---
layout: post
title: Stack sort algorithm
date: 2018-01-03 00:00
tags: [java, algorithm]
---

### 堆排序算法

```java
public class StackSort {

    public static void main(String[] args) {
        int[] items = {1, 3, 5, 6, 7, 9, 1, 2, 3};

        doStackSort(items);

        Util.print(items);
    }

    private static void doStackSort(int[] items) {

        for (int i = items.length / 2; i >= 0; i--) {
            adJustStack(items, i);
        }
    }

    private static void adJustStack(int[] items, int i) {

        int lowest = i;
        int left = i * 2 + 1;
        int right = i * 2 + 2;

        if(left < items.length && items[left] < items[lowest]) {
            lowest =  left;
        }

        if(right < items.length && items[right] < items[lowest]) {
            lowest = right;
        }

        if(lowest == i) {
            return;
        }

        swap(items, lowest, i);

        adJustStack(items, lowest);
    }

    private static void swap(int[] items, int lowest, int i) {

        int total = items[lowest] + items[i];
        items[lowest] = total - items[lowest];
        items[i] = total - items[i];
    }
}

```