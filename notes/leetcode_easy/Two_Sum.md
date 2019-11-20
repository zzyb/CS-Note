## 描述：

**给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。**

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/two-sum

## 尝试：

```java
package com.yber.leetcode.easy;

public class TwoNumber {
    public static void main(String args[]){
        int[] source = {5,3,6,4,9,10,7,14};
        int tager = 24;
        int[] res0 = get(source,tager);
        for(int value:res0){
            System.out.println("数组下标:"+value);
        }
    }

    public static int[] get(int[] source,int tager){
        int[] res = new int[2];
        boolean flag = false;
        for(int i = 0;i<source.length-1;i++){
            for(int j = i+1;j<source.length;j++){
                if (source[i] + source[j] == tager) {
                    res[0] = i;
                    res[1] = j;
                    flag = true;
                    break;
                }
            }
            if(flag){
                break;
            }
        }
        return res;
    }
}

```

## 官方写法：

### 一：暴力方法

暴力法很简单，遍历每个元素 xxx，并查找是否存在一个值与 target−xtarget - xtarget−x 相等的目标元素。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for(int i = 0;i<nums.length-1;i++){
            for(int j = i+1;j<nums.length;j++){
                if(nums[i]+nums[j]==target){
                    return new int[] {i,j};
                }
            }
        }
        throw new IllegalArgumentException("no");
    }
}
```

### 二：两遍哈希表

为了对运行时间复杂度进行优化，我们需要一种更有效的方法来检查数组中是否存在目标元素。如果存在，我们需要找出它的索引。保持数组中的每个元素与其索引相互对应的最好方法是什么？哈希表。

通过以空间换取速度的方式，我们可以将查找时间从 O(n)O(n)O(n) 降低到 O(1)O(1)O(1)。哈希表正是为此目的而构建的，它支持以 近似 恒定的时间进行快速查找。我用“近似”来描述，是因为一旦出现冲突，查找用时可能会退化到 O(n)O(n)O(n)。但只要你仔细地挑选哈希函数，在哈希表中进行查找的用时应当被摊销为 O(1)O(1)O(1)。

一个简单的实现使用了两次迭代。在第一次迭代中，我们将每个元素的值和它的索引添加到表中。然后，在第二次迭代中，我们将检查每个元素所对应的目标元素（target−nums[i]target - nums[i]target−nums[i]）是否存在于表中。注意，该目标元素不能是 nums[i]nums[i]nums[i] 本身！

```JAVA
    public static int[] twoSum1(int[] source,int targer){
        Map<Integer,Integer> map = new HashMap<Integer,Integer>();
        for(int i = 0;i<source.length;i++){
            map.put(source[i],i);
        }
        for(int j = 0;j<source.length;j++){
            int res = targer-source[j];
            if(map.containsKey(res) && map.get(res)!=j)
                return new int[] {j,map.get(res)};
        }
        throw new IllegalArgumentException("no two number");
    }
```

### 三：一遍哈希表

事实证明，我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

```JAVA
    public static int[] twoSum2(int[] source,int targer){
        Map<Integer,Integer> map = new HashMap<Integer, Integer>();
        for(int i = 0;i<source.length;i++){
            int res = targer - source[i];
            if(map.containsKey(res) && map.get(res)!=i)
                return new int[] {i,map.get(res)};
            map.put(source[i],i);
        }
        throw new IllegalArgumentException("no!");
    }
```

