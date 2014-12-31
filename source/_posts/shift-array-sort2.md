title: 循环移位算法题2
date: 2014-12-27
tags:
---
原题在此：
https://oj.leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/

> Follow up for “Find Minimum in Rotated Sorted Array”:  
What if duplicates are allowed?  
Would this affect the run-time complexity? How and why?  
Suppose a sorted array is rotated at some pivot unknown to you beforehand.  
(i.e., **0 1 2 4 5 6 7** might become **4 5 6 7 0 1 2**).  
Find the minimum element.  
The array may **contain duplicates**.

就是说一个从小到大排序好的数组循环移位不知多少次，求最小值。数组可以有重复值！
这就比无重复的难一些了。
可以重复会带来不少问题，之前的不重复循环移位的判定条件都要重新思考是否有效。
比如全部是1的数列，和除了某位置有个2，其余全部是1的数列，都是合法的。
上个题目中的移动和终止条件似乎都无效了

相比上题而言，这里有几个针对重复值的关键：
- 如果左游标遇到重复值，则移动到该重复值序列的最右边（moveRight）
- 如果右游标遇到重复值，则移动到该重复值序列的最左边（moveLeft）

还有一个边界情况：如果碰到全1的序列呢？moveLeft会一直进行下去死循环了。所以对于这种情况我们如果循环一圈发现还没有停止moveLeft的意思，就要break掉并且在返回值中进行标记了。

这个算法的时间复杂度相对于上个算法有什么变化呢？平均时间复杂度还是O(log n)，但是出现了最坏的情况，全部重复或者很多重复的情况。最坏肯定是O(n)啦。上一个算法似乎不会出现这种情况吧。

```java
import java.util.ArrayList; 
import java.util.Collections;
import java.util.List;
import java.util.Random;
import java.util.Set;
import java.util.TreeSet;
public class ShiftFinder2 {
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
            cur = moveLeft(array, cur);
            if (cur == -1) {
                cur = 0;
                break;
            }
            l = moveRight(array, l);
            r = moveLeft(array, r);
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
                l = moveRight(array, cur);
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];
    }
    public static int moveLeft(int[] array, int pos) {
        int counter = 0;
        int len = array.length;
        pos = index(pos, len);
        while (true) {
            if (array[pos] != array[index(pos - 1, len)]) {
                return pos;
            }
            if (counter >= len) {
                // special case
                return -1;
            }
            pos = index(pos - 1, len);
            counter++;
        }
    }
    public static int moveRight(int[] array, int pos) {
        int len = array.length;
        pos = index(pos, len);
        while (true) {
            if (array[pos] != array[index(pos + 1, len)]) {
                return pos;
            }
            pos = index(pos + 1, len);
        }
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
        int[] h = { 1, 1, 1, 1, 1, 1 };
        int[] m = { 2, 1, 1, 1, 1, 1, 1 };
        int[] n = { 2, 2, 1, 1, 1, 1, 1, 1, 2 };
        int[] o = { 1, 1, 1, 1, 1, 1, 1, 2 };
        int[] p = { 1, 1, 1, 1, 1, 1, 2, 2 };
        int[] q = { 5, 5, 1, 3, 3, 3, 3, 3, 3, 4 };
        int[] r = { 4, 4, 5, 6, 6, 6, 6, 7, 9, 1, 1, 2 };
        int[] s = { 18, 18, 18, 19, 19, 19, 20, 20, 20, 20, 20, 20, 21, 21, 21, 22, 22, 23, 23, 23, 23, 23, 23, 23, 23,
                24, 24, 25, 25, 25, 25, 26, 27, 27, 27, 27, 28, 28, 29, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3, 3, 4, 4, 4, 5, 6,
                6, 7, 8, 8, 9, 9, 9, 9, 10, 10, 11, 11, 11, 11, 11, 11, 12, 12, 12, 12, 12, 13, 13, 13, 13, 14, 14, 14,
                14, 15, 15, 16, 17, 17, 17, 17, 17, 17, 18, 18 };
        int[] t = { 7, 7, 8, 8, 8, 9, 9, 9, 9, 9, 10, 10, 10, 11, 11, 12, 12, 12, 12, 12, 13, 14, 14, 15, 15, 15, 15,
                16, 16, 17, 17, 17, 17, 18, 18, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 21, 22, 22, 22, 22, 22, 22,
                23, 23, 23, 23, 24, 24, 25, 26, 28, 28, 28, 28, 29, 29, 29, 29, 0, 0, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 3,
                3, 3, 3, 4, 4, 5, 5, 5, 6, 6, 6, 6, 6, 6, 6, 7, 7, 7 };
        int[] u = { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 1, 1, 1 };
        int[] v = { 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 };
        System.out.println(ShiftFinder2.findMin(a));
        System.out.println(ShiftFinder2.findMin(b));
        System.out.println(ShiftFinder2.findMin(c));
        System.out.println(ShiftFinder2.findMin(d));
        System.out.println(ShiftFinder2.findMin(e));
        System.out.println(ShiftFinder2.findMin(f));
        System.out.println(ShiftFinder2.findMin(g));
        System.out.println(ShiftFinder2.findMin(h));
        System.out.println(ShiftFinder2.findMin(m));
        System.out.println(ShiftFinder2.findMin(n));
        System.out.println(ShiftFinder2.findMin(o));
        System.out.println(ShiftFinder2.findMin(p));
        System.out.println(ShiftFinder2.findMin(q));
        System.out.println(ShiftFinder2.findMin(r));
        System.out.println(ShiftFinder2.findMin(s));
        System.out.println(ShiftFinder2.findMin(t));
        System.out.println(ShiftFinder2.findMin(u));
        System.out.println(ShiftFinder2.findMin(v));
        // gen non-repeatable random shift array
        System.out.println("----- test non-repeatable shift array -----");
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
            aa[ShiftFinder2.index(i + shift, aa.length)] = iay[i];
        }
        for (int i = 0; i < aa.length; i++) {
            System.out.print(aa[i] + " ");
        }
        System.out.println();
        System.out.println("non-repeatable random minimum find: " + ShiftFinder2.findMin(aa));
        System.out.println();
        // gen repeatable random shift array
        System.out.println("----- test repeatable shift array -----");
        attemptSize = 100;
        randomRange = 30;
        shift = rdm.nextInt(attemptSize);
        System.out.println("size: " + attemptSize + "; shift: " + shift);
        List<Integer> jay = new ArrayList<Integer>();
        for (int i = 0; i < attemptSize; i++) {
            jay.add(rdm.nextInt(randomRange));
        }
        Collections.sort(jay);
        int[] bb = new int[attemptSize];
        for (int i = 0; i < attemptSize; i++) {
            bb[ShiftFinder2.index(i + shift, attemptSize)] = jay.get(i);
        }
        for (int i = 0; i < attemptSize; i++) {
            System.out.print(bb[i] + " ");
        }
        System.out.println();
        System.out.println("repeatable random minimum find: " + ShiftFinder2.findMin(bb));
        System.out.println();
    }
}
```
在这里，测试用例也进行了增加，尽量覆盖各种奇葩情况。后面除了不重复的随机移位数列之外，还有可重复的随机移位序列。
最终提交的代码：
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
            cur = moveLeft(array, cur);
            if (cur == -1) {
                cur = 0;
                break;
            }
            l = moveRight(array, l);
            r = moveLeft(array, r);
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
                l = moveRight(array, cur);
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];        
    }
   
    public static int moveLeft(int[] array, int pos) {
        int counter = 0;
        int len = array.length;
        while (true) {
            if (array[pos] != array[(pos+len-1)%len]) {
                return pos;
            }
            if (counter >= len) {
                // special case
                return -1;
            }
            pos = (pos+len-1)%len;
            counter++;
        }
    }

    public static int moveRight(int[] array, int pos) {
        int len = array.length;
        while (true) {
            if (array[pos] != array[(pos+1)%len]) {
                return pos;
            }
            pos = (pos+1)%len;
        }
    }
}
```
![](http://7teb9z.com1.z0.glb.clouddn.com/java_2.jpg)

**C++ 算法**：
跟java算法完全一样
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
            cur = moveLeft(array, cur);
            if (cur == -1) {
                cur = 0;
                break;
            }
            l = moveRight(array, l);
            r = moveLeft(array, r);
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
                l = moveRight(array, cur);
                cur = (l + r) / 2;
                continue;
            }
            r = cur;
            cur = (l + r) / 2;
        }
        return array[cur];
    }
   
    int moveLeft(vector<int> &array, int pos) {
        int counter = 0;
        int len = array.size();
        while (true) {
            if (array[pos] != array[(pos+len-1)%len]) {
                return pos;
            }
            if (counter >= len) {
                // special case
                return -1;
            }
            pos = (pos+len-1)%len;
            counter++;
        }
    }
    int moveRight(vector<int> &array, int pos) {
        int len = array.size();
        while (true) {
            if (array[pos] != array[(pos+1)%len]) {
                return pos;
            }
            pos = (pos+1)%len;
        }
    }
};
```
![](http://7teb9z.com1.z0.glb.clouddn.com/cpp_2.jpg)
时间也不准。