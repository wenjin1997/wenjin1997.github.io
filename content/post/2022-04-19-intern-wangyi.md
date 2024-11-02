---
layout:     post

title:      "网易实习笔试题"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-04-19
description: "总结网易实习笔试题。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-04-19 
tags:
     - Java
     - 面试

categories: [ Tech ]
URL: "/2022/04/19/intern-wangyi"
---
## 打印字母“O”
给一个数n，打印4n行字符串，是一个大“O”的形状。

例1: `n = 1`
```
.**.
*..*
*..*
.**.
```

例2: `n = 2`
```
..****..
.******.
**....**
**....**
**....**
**....**
.******.
..****..
```

例3: `n = 3`
```
...******...
..********..
.**********.
***......***
***......***
***......***
***......***
***......***
***......***
.**********.
..********..
...******...
```
### 思路
将打印每一行抽象成一个函数，例如第一行，先打印字符`ch1`，重复`times1`遍，再打印`ch2`有`times2`遍，最后打印`ch1` 有`times1`遍。
### 代码
```java
import java.util.Scanner;

/**
 * @author 文进
 * @version 1.0
 */
public class N1 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 4 * n; i++) {
            if (i < n) { // [0, ..., n - 1]
                sb.append(printStr('.', n - i, '*', 4 * n - 2 * (n - i)));
            } else if (i >= 3 * n) { // [3n, ..., 4n - 1]
                sb.append(printStr('.', i - 3 * n + 1, '*', 4 * n - 2 * (i - 3 * n + 1)));
            } else { // [n, ..., 3n - 1]
                sb.append(printStr('*', n, '.', 2 * n));
            }
            sb.append("\n");
        }

        // // 或者分开写，分别打印三大段
        // for (int i = n; i > 0; i--) {
        //     sb.append(printStr('.', i, '*', 4 * n - 2 * i));
        //     sb.append("\n");
        // }

        // for (int i = 0; i < 2 * n; i++) {
        //     sb.append(printStr('*', n, '.', 2 * n));
        //     sb.append("\n");
        // }

        // for (int i = 1; i <= n; i++) {
        //     sb.append(printStr('.', i, '*', 4 * n - 2 * i));
        //     sb.append("\n");
        // }

        System.out.println(sb);
    }

    // ch1 * times1 + ch2 * times2 + ch1 * times1
    public static StringBuilder printStr(char ch1, int times1, char ch2, int times2) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < times1; i++) {
            sb.append(ch1);
        }
        for (int i = 0; i < times2; i++) {
            sb.append(ch2);
        }
        for (int i = 0; i < times1; i++) {
            sb.append(ch1);
        }
        return sb;
    }
}
```
## n数之和
要构造一个长度为n的数组，输入三个参数n，k，x。n表示数组的长度，数组中所有元素不相等且不超过k，n个数的和为k。最后输出一个任意一个结果就可以，如果找不到这样的数组，就返回-1。

例1
输入
```
4 6 15
```
输出：
```
1 5 6 3
```

例2
输入
```
2 3 4
```
输出
```
-1
```
### 思路
这个题是n数之和的问题，联想到在一个数组中求和为target的n个数。如果n=2，那么就是两数之和，可以先对数组排序，用双指针求两数之和。当求n个数之和为target时，可以先固定一个数nums[i]，再求n-1个数的和为target-nums[i]。

这道题只需返回一个可行结果，那么当找到一个解时就立刻返回，可以避免不必要的递归调用。

同时题目中告诉我们返回的数不超过k并且各不相同，那么提前构造的数组可以是`nums = [1,2,3,...,k]`，且`nums`数组不需要再排序了。

类似的LeetCode题目：
* [1.两数之和](https://leetcode-cn.com/problems/two-sum/)
* [15.三数之和](https://leetcode-cn.com/problems/3sum/)
* [18.四数之和](https://leetcode-cn.com/problems/4sum/)
* [454.四数相加II](https://leetcode-cn.com/problems/4sum-ii/)

### 代码
```java
import java.util.LinkedList;
import java.util.List;
import java.util.Scanner;

/**
 * @author 文进
 * @version 1.0
 */
public class N2 {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int  n = scanner.nextInt();
        int k = scanner.nextInt();
        int x = scanner.nextInt();
        int[] nums = new int[k];
        for (int i = 0; i < k; i++) {
            nums[i] = i + 1;
        }
        List<Integer> res = nSum(nums, n, 0, x);
        if (res.size() == n) {
            StringBuilder sb = new StringBuilder();
            for (Integer ele : res) {
                sb.append(ele + " ");
            }
            System.out.println(sb);
        } else {
            System.out.println(-1);
        }
    }

    /*
    n 数之和，nums[] 提前排好序了，并且各不相同
    双指针方法
     */
    public static List<Integer> nSum(int[] nums, int n, int start, int target) {
        List<Integer> res = new LinkedList<>();
        if (n < 2 || n > nums.length) return res;
        if (n == 2) {
            int left = start, right = nums.length - 1;
            while (left < right) {
                int sum = nums[left] + nums[right];
                if (sum < target) {
                    left++;
                } else if (sum > target) {
                    right--;
                } else {
                    res.add(nums[left]);
                    res.add(nums[right]);
                    left++;
                    right--;
                }
            }
        } else {
            for (int i = start; i < nums.length - 1; i++) {
                List<Integer> list = nSum(nums, n - 1, i + 1, target - nums[i]);
                if (list.size() == n - 1) {
                    for (Integer ele : list) {
                        res.add(ele);
                    }
                    res.add(nums[i]);
                    break;
                }
            }
        }
        return res;
    }
}
```