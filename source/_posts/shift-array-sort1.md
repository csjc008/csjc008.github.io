title: 循环移位算法题1
date: 2014-12-27
tags:
---

问题从这个网站上看到：
https://oj.leetcode.com/problems/find-minimum-in-rotated-sorted-array/

> **Question:**  
Suppose a sorted array is rotated at some pivot unknown to you beforehand.  
(i.e., **0 1 2 4 5 6 7** might become **4 5 6 7 0 1 2**).  
Find the minimum element.  
You may assume **no duplicate** exists in the array.  

就是说一个从小到大排序好的数组循环移位不知多少次，求最小值。数组无重复值！
无重复的话就比较简单，用二分查找即可。算法时间复杂度为O(log n)。
基本想法就是定义三个游标：左游标，右游标，当前游标。当前游标始终是左右游标中点位置，与左右游标的数值比较。

<!--more-->

有几个要点：
- 基本终止条件为：左边的数比当前的数大，那么当前数即是最小值。
- 额外终止条件：当左右游标重合，或者左右游标相邻。
- 需要考虑边界条件。
- 移动条件1：如果当前游标的数值比左游标数值小，则游标左移。
- 移动条件2：如果当前游标的数值比右游标数值大，则游标右移。
- 默认移动：游标左移。

我写的算法代码和测试代码如下：
```java
import java.util.Random; 
import java.util.Set;
import java.util.TreeSet;
public class ShiftFinder {

    public static int findMin(int[] array) {
        if (array.length == 0) {
            return 0;
        }
        if (array.length == 1) {
            return array[0];
        }
        int len = array.length;
        int l = 0;
        int r = len - 1;
        int cur = (l + r) / 2;
        while (true) {
            if (array[cur] < array[index(cur - 1, len)]) {
                break;
            }
            if (l == r) {
                cur = l;
                break;
            }
            if (r == (l + 1)) {
                if (array[l] < array[r]) {
                    cur = l;
                } else {
                    cur = r;
                }
                break;
            }
            if (array[cur] < array[l]) {
                r = cur;
                cur = (l + r) / 2;
                continue;
            }
            if (array[cur] > array[r]) {
                l = cur;
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];
    }

    public static int index(int cur, int length) {
        return (cur % length + length) % length;
    }

    public static void main(String[] args) {
        int[] a = { 7, 8, 11, 12, 13, 14, 19, 22, 1, 2, 4, 5 };
        int[] b = { 1, 2, 3, 4, 5, 6, 7 };
        int[] c = { 11, 1, 2, 4, 5, 7, 8 };
        int[] d = { 1 };
        int[] e = { 1, 2 };
        int[] f = { 2, 1 };
        int[] g = { 3, 1, 2 };
        // System.out.println(ShiftFinder.index(0, 10));
        // System.out.println(ShiftFinder.index(2, 10));
        // System.out.println(ShiftFinder.index(-2, 10));
        // System.out.println(ShiftFinder.index(13, 10));
        // System.out.println(ShiftFinder.index(-13, 10));
        // System.out.println(ShiftFinder.index(1, 1));
        System.out.println(ShiftFinder.findMin(a));
        System.out.println(ShiftFinder.findMin(b));
        System.out.println(ShiftFinder.findMin(c));
        System.out.println(ShiftFinder.findMin(d));
        System.out.println(ShiftFinder.findMin(e));
        System.out.println(ShiftFinder.findMin(f));
        System.out.println(ShiftFinder.findMin(g));

        // gen random shift array
        int attemptSize = 100;
        int randomRange = 999;
        Random rdm = new Random();
        Set<Integer> ts = new TreeSet<Integer>();
        for (int i = 0; i < attemptSize; i++) {
            ts.add(rdm.nextInt(randomRange));
        }
        int shift = rdm.nextInt(ts.size());
        System.out.println("size: " + ts.size() + "; shift: " + shift);
        Integer[] iay = new Integer[ts.size()];
        ts.toArray(iay);
        int[] aa = new int[ts.size()];
        for (int i = 0; i < ts.size(); i++) {
            aa[ShiftFinder.index(i + shift, aa.length)] = iay[i];
        }
        for (int i = 0; i < aa.length; i++) {
            System.out.print(aa[i] + " ");
        }
        System.out.println();
        System.out.println("random minimum find: " + ShiftFinder.findMin(aa));
    }
}
```

在固定的测试用例之后，附带了一段随机生成的数值测试，生成随机数，插入一个TreeSet中，去重并由小到大排序好，再遍历一次以随机移位存储。

最终我提交到平台上的代码是：
```java
public class Solution {
    public int findMin(int[] array) {
        if (array.length == 0) {
            return 0;
        }
        if (array.length == 1) {
            return array[0];
        }
        int len = array.length;
        int l = 0;
        int r = len - 1;
        int cur = (l + r) / 2;
        while (true) {
            if (array[cur] < array[(cur+len-1)%len]) {
                break;
            }
            if (l == r) {
                cur = l;
                break;
            }
            if (r == (l + 1)) {
                if (array[l] < array[r]) {
                    cur = l;
                } else {
                    cur = r;
                }
                break;
            }
            if (array[cur] < array[l]) {
                r = cur;
                cur = (l + r) / 2;
                continue;
            }
            if (array[cur] > array[r]) {
                l = cur;
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];
    }
}
```
所有用例测试通过。
![](http://7teb9z.com1.z0.glb.clouddn.com/java_1.jpg)
这个时间并不准，有时候好，有时候坏。这是最好的情况。在java算法中排名靠前。

**C++算法**：
跟java算法完全相同

```cpp
class Solution {
public:
    int findMin(vector<int> &array) {
        int len = array.size();
        if (len == 1) {
            return array[0];
        }
       
        int l = 0;
        int r = len - 1;
        int cur = (l + r) / 2;
        while (true) {
            if (array[cur] < array[(cur+len-1)%len]) {
                break;
            }
            if (l == r) {
                cur = l;
                break;
            }
            if (r == (l + 1)) {
                if (array[l] < array[r]) {
                    cur = l;
                } else {
                    cur = r;
                }
                break;
            }
            if (array[cur] < array[l]) {
                r = cur;
                cur = (l + r) / 2;
                continue;
            }
            if (array[cur] > array[r]) {
                l = cur;
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];
    }
};
```
![](http://7teb9z.com1.z0.glb.clouddn.com/cpp_1.jpg)
居然排在了最前面。。。