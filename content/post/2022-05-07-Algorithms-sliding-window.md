---
layout:     post

title:      "滑动窗口相关算法"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-05-07
description: "总结滑动窗口相关LeetCode题目。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-05-07 
tags:
     - Java
     - 算法
     - 滑动窗口

categories: [ Tech ]
URL: "/2022/05/07/Algorithms-sliding-window/"
---
## 滑动窗口
滑动窗口模板：
```java
int left = 0, right = 0;

while (right < s.size()) {
    // 增大窗口
    window.add(s[right]);
    right++;
    
    while (window needs shrink) {
        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}
```
## [209.长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)
这道题HotStar一面考到了，用滑动窗口解决。这里写的是前缀和数组和滑动窗口来解决，
其实也可以降低空间复杂度，直接用滑动窗口解决，一个变量来记子数组的和。
```java
class Solution {
    // 前缀和数组 + 滑动窗口
    public int minSubArrayLen(int target, int[] nums) {
        int[] preSum = new int[nums.length];
        int sum = 0;
        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
            preSum[i] = sum;
        }
        int left = 0, right = 0;
        int length = Integer.MAX_VALUE;
        while (left <= right && right < nums.length) {
            while (right < nums.length && preSum[right] - preSum[left] + nums[left] < target) {
                right++;
            }
            System.out.println(right);
            length = Math.min(length, right - left + 1);
            System.out.println(length);
            left++;
        }
        return preSum[nums.length - 1] < target ? 0 : length;
    }
}
```
## [3.无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/description/)
直接上代码：
```java
class Solution {
    // 滑动窗口
    public int lengthOfLongestSubstring(String s) {
        if (s.length() == 0) return 0;
        int left = 0, right = 0;
        int length = 1;
        Set<Character> set = new HashSet<>();
        while (left <= right && right < s.length()) {
            while (right < s.length() && !set.contains(s.charAt(right))) {
                set.add(s.charAt(right));
                right++;
            }
            // System.out.println("left " + left + " right " + right);
            length = Math.max(length, right - left);
            set.remove(s.charAt(left));
            left++;  
        }
        return length;
    }
}
```
这里和模板有所区别，不过思路都是一样的。这里是不停的在移动右指针，一旦不满足条件，就调节左指针。
## [76.最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)
这道题需要好好想一下，什么时候收缩窗口，我们知道肯定是窗口中已经包含了整个字符串`t`中的所有字符，但是这个逻辑如何去实现呢？下面假设`String s = "ADOBECODEBANC"`，`String t = "ABC"`。
* 首先根据字符串`t`构造一个哈希表`subStringMap`。
* 滑动窗口内的哈希表记为`curMap`，但是这个哈希表不存储滑动窗口内所有的字符和相应出现的次数，我们只关心那些在`subStringMap`中包含的字符。
* 设置一个变量`valid`，记录窗口内的有效字符的个数。如果当前窗口中`A`出现的次数达到了`subStringMap`中`A`出现的次数，`valid++`，也就是滑动窗口内覆盖了字符`A`，这也是为了后面好判断窗口收缩条件，也就是`valid == subStringMap.size()`，满足这个条件说明滑动窗口内刚好覆盖了子串。这一步思路最为关键。
* 窗口收缩后，更新相应变量的值，`curMap`要变，`valid`也需要更新。
* 最后为了返回最小的覆盖子串，用`len`和`start`两个变量来记录结果。
```java
class Solution {
    // 滑动窗口
    public String minWindow(String s, String t) {
        if (t.length() > s.length()) return "";
        HashMap<Character, Integer> subStringMap = new HashMap<>();
        for (int i = 0; i < t.length(); i++) {
            subStringMap.put(t.charAt(i), subStringMap.getOrDefault(t.charAt(i), 0) + 1);
        }
        HashMap<Character, Integer> curMap = new HashMap<>();
        int left = 0, right = 0;
        int valid = 0;
        int start = 0, len = Integer.MAX_VALUE;
        while (right < s.length()) {
            char c = s.charAt(right);
            right++;
            if (subStringMap.containsKey(c)) {
                curMap.put(c, curMap.getOrDefault(c, 0) + 1);
                if (curMap.get(c).equals(subStringMap.get(c))) {
                    valid++;
                }
            }

            while (valid == subStringMap.size()) {
                // 更新最小字符串
                if (right - left < len) {
                    start = left;
                    len = right - left;
                }
                char d = s.charAt(left);
                left++;
                if (subStringMap.containsKey(d)) {
                    if (subStringMap.get(d).equals(curMap.get(d))) {
                        valid--;
                    }
                    curMap.put(d, curMap.getOrDefault(d, 0) - 1);
                }
            }
        }
        return len == Integer.MAX_VALUE ? "" : s.substring(start, start + len);
    }
}
```
## 参考资料
* [我写了首诗，把滑动窗口算法算法变成了默写题](https://labuladong.github.io/algo/2/18/25/)