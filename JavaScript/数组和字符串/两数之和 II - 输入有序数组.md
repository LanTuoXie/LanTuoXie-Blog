# 两数之和 II - 输入有序数组

[题目链接](https://leetcode-cn.com/leetbook/read/array-and-string/cnkjg/)

给定一个已按照**升序排列** 的有序数组，找到两个数使得它们相加之和等于目标数。

函数应该返回这两个下标值 `index1` 和 `index2`，其中 `index1` 必须小于 `index2`。

- 返回的下标值`(index 1 和 index2)` 不是从零开始。
- 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

```bash
输入: numbers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
```

## 题目分析

条件分析：

- 升序排列的有序数组
- index1 必须小于 index2
- 两数相加等于目标数，可以先确定一个数，然后查找另一个数。问题就变成了搜索查找某个值了。
- 在有序数列中查找某个值，首先想到的是 `二分搜索`。

## 解题思路：备忘录(`javascript`)

```js
/**
 * @param {number[]} numbers
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(numbers, target) {
    var map = {};
    numbers.forEach((num, index) => {
        map[num] = index;
    });

    for(var i = 0; i < numbers.length; i++) {
        const v = target - numbers[i];
        if (map[v] !== undefined) {
            return map[v] + 1 > i + 1 ? [i + 1, map[v] + 1] : [map[v] + 1, i + 1]
        }
    }

    return [-1， -1];
};
```

- 上面使用了备忘录的功能，但是没有使用到题目提供的 2 个条件：`升序排列的有序数组` `index1 必须小于 index2`
- 缺点：内容`O(n)` 时间`O(2n)`

## 二分搜索

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        for (int i = 0; i < numbers.length; ++i) {
            int low = i + 1, high = numbers.length - 1;
            while (low <= high) {
                int mid = (high - low) / 2 + low;
                if (numbers[mid] == target - numbers[i]) {
                    return new int[]{i + 1, mid + 1};
                } else if (numbers[mid] > target - numbers[i]) {
                    high = mid - 1;
                } else {
                    low = mid + 1;
                }
            }
        }
        return new int[]{-1, -1};
    }
}
```

- 二分搜索的条件：有序的数组，符合二分搜索，而且题目要求`index1 小于 index2`，很明显提示要使用二分搜索；
- 时间复杂度`O(nlogn)`

## 双指针搜索

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int low = 0, high = numbers.length - 1;
        while (low < high) {
            int sum = numbers[low] + numbers[high];
            if (sum == target) {
                return new int[]{low + 1, high + 1};
            } else if (sum < target) {
                ++low;
            } else {
                --high;
            }
        }
        return new int[]{-1, -1};
    }
}
```

- 这里还是抓住了数组的有序性，而双指针和二分搜索本来就有点相似。
