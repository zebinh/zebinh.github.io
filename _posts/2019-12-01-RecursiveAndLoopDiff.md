---
layout: post
title: 循环和递归对比
date: 2019-12-01
tags: 算法
---

# 循环与递归对比

大学学习递归的时候有一句话印象深刻：所有的递归都可以改写为循环。这句话我是同意的，因为递归其实本质上就是栈的操作。虽然如此，但递归有递归的优点，递归算法写出来的程序都比较简短而且优雅，缺点是其站在了计算机的角度，人很难深究其中，容易陷入递归的汪洋大海中。既然所有的递归都可以改写为循环，反之，那循环是否可以改写为递归呢？

今天再次想到这个问题，是因为做了《剑指Offer》的一道递归题：合并两个排序的链表，[牛客网剑指offer第16道题](https://www.nowcoder.com/practice/d8b6b4358f774294a89de2a6ac4d9337?tpId=13&tqId=11169&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

## 递归和循环相同点

递归和循环都有退出条件，每次递归或循环执行的程序体都是一样的。while (true)循环和递归最相似，两者思路都很清晰，所以我比较喜欢while (true)循环

```java
// 递归写法
public void recursive(int n) {
    // 递归结束条件
    if (n == 0) {
        return;
    }

    // 其他子问题，递归解决
    recursive(n - 1);
}


public void theLoop() {

    // 循环写法
    while (true) {
        // 循环结束条件
        if (n == 0) {
            break;
        }

        // 其他问题继续循环
        n = n - 1;
    }
}
```

## 递归种类

我大致将递归分为两种：可以有返回值和无返回值，这两个貌似差别挺大，但目前我想不明白。比如这道题，打印从1加到n的和。无返回值的话写起来比较麻烦。

实现如下：
```java
/**
 * 自我思考，1加到n无返回值和有返回值两种
 */

 public class Sum {

    public static void main(String[] args){
        System.out.println(recursiveReturnInt(5));
        recursiveReturnVoid(5, 0);
    }

    // 有返回值的递归写法
    public static int recursiveReturnInt(int n){
        if (n == 0) {
            return 0;
        }
        return n + recursiveReturnInt(n - 1);
    }

    // 无返回值的递归写法
    public static void recursiveReturnVoid(int n, int sum){

        if (n == 0) {
            System.out.println(sum);
            return;
        }
        sum += n;
        recursiveReturnVoid(n - 1, sum);
        
    }
 }
 ```

## 循环是否都可以改为递归

应该是不可以的，不过解决子问题的循环都可以改为递归。如冒泡排序，反转链表等。冒泡排序n个数时每次都把最大的数排到最后一位，接着排序剩下的n-1位。反转链表每次都反转当前节点和下一个节点，接着指针前进反转下一个节点和下下个节点。

以下是冒泡排序的非递归和递归写法。
```java
/**
 * 自我思考，冒泡排序的递归和非递归写法
 */

 public class BubbleSortTwoSolution {

    public static void main(String[] args) {

        int[] array = new int[]{4, 6, 1, 3, 2, 3, 9, 8, 0};
        bubbleSortLoopSolution(array);
        System.out.print("循环写法：");
        for(int i : array) {
            System.out.print(i + " ");
        }
        System.out.println();

        array = new int[]{4, 6, 1, 3, 2, 3, 9, 8, 0};
        bubbleSortRecursiveSolution(array, 8);
        System.out.print("递归写法：");
        for(int i : array) {
            System.out.print(i + " ");
        }
        System.out.println();

        array = new int[]{4, 6, 1, 3, 2, 3, 9, 8, 0};
        bubbleSortWhileLoopSolution(array);
        System.out.print("while (true)写法：");
        for(int i : array) {
            System.out.print(i + " ");
        }
        System.out.println();
    }

    // 冒泡排序循环写法
    public static void bubbleSortLoopSolution(int[] array) {

        for (int i = 0; i < array.length; i++) {
            for (int j = 0; j < array.length - i - 1; j++) {
                if (array[j] > array[j+1]) {
                    int tmp = array[j];
                    array[j] = array[j+1];
                    array[j+1] = tmp;
                }
            }
        }
    }

    // 冒泡排序递归写法
    public static void bubbleSortRecursiveSolution(int[] array, int index) {

        if (index < 0) {
            return;
        }

        for (int j = 0; j < index; j++) {
            if (array[j] > array[j+1]) {
                int tmp = array[j];
                array[j] = array[j+1];
                array[j+1] = tmp;
            }
        }

        bubbleSortRecursiveSolution(array, index-1);
    }

    // while (true)循环写法，跟递归很像
    public static void bubbleSortWhileLoopSolution(int array[]) {

        int index = array.length - 1;
        while (true) {

            if (index < 0) {
                break;
            }

            for (int j = 0; j < index; j++) {
                if (array[j] > array[j+1]) {
                    int tmp = array[j];
                    array[j] = array[j+1];
                    array[j+1] = tmp;
                }
            }

            index--;
        }
    }
 }
```
从上面可以看出来，递归和while(true)循环很像，并且思路很清晰。用递归来解决存在子问题的算法，代码很优雅。



