---
title: "将有重叠或相邻的时间区间进行合并"
date: 2025-02-06T14:16:03+08:00
lastmod: 2025-02-06T14:16:03+08:00
draft: false
keywords: ["区间重叠","区间相邻","合并"]
description: "基于力扣中的算法题对区间合并算法进行改进，不仅能合并重叠区间，也能合并相邻的区间"
tags: []
categories: ["算法"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: false
codeTabSeperator: "::"
moreMeta: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

基于力扣中的[**合并区间**](https://leetcode.cn/problems/merge-intervals/)题解对其进行适当的改进，以实现不仅能对重叠的时间区间进行合并，同时也可对相邻的时间区间进行合并。

<!--more-->

## 背景

原始的[合并区间](https://leetcode.cn/problems/merge-intervals/)中的问题描述如下：

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

**示例 1：**

输入：intervals = [[1,3],[2,6],[8,10],[15,18]]<br>
输出：[[1,6],[8,10],[15,18]]<br>
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6]

**示例 2：**

输入：intervals = [[1,4],[4,5]]<br>
输出：[[1,5]]<br>
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间

基于原题目的描述以及对应的答案，当输入为 [[1,3],[4,5]]时，输出依旧为 [[1,3],[4,5]]，而在个人项目中的数据都是整数，对于此种场景，由于3和4时连续的，故要求输出为  [[1,5]]，即**当两个区间不重叠但相邻时也需要将其进行合并**。

## 分析

大体思路是先看明白原题的解决思路，然后再进行适当改进。

官方提供的答案类似如下

```java {data-line="2,10,14"}
public List<List<Integer>> mergeIntervals(List<List<Integer>> configs) {
    configs.sort(Comparator.comparingInt(a -> a.get(0)));
    List<List<Integer>> merged = new ArrayList<>();

    for (List<Integer> config : configs) {
        int start = config.get(0);
        int end = config.get(1);

        // 如果 merged 为空或当前区间与最后一个合并区间不重叠
        if (merged.isEmpty() || merged.get(merged.size() - 1).get(1) < start) {
            merged.add(Arrays.asList(start, end));
        } else {
            // 有重叠或相邻，更新最后一个合并区间的结束点
            merged.get(merged.size() - 1).set(1, Math.max(merged.get(merged.size() - 1).get(1), end));
        }
    }

    return merged;
}
```

上述代码中最关键的是高亮显示的3行代码(2,10,14)，它们的作用分别如下：

* 第2行用于对输入数据基于区间的开始位置进行排序，只有排序后有序数组才能进行下一步的处理
* 第10行用于识别到新的区间时，将其添加到最终结果列表中，识别的规则为新区间的开始位置与最后1个区间的结束位置
* 第14行用于对重叠或相邻区间进行合并，由于数据已经排好序了，所以合并区间时只需要更新其结束位置即可

以输入数据`[[2,6],[8,15],[1,3],[4,5],[16,19]]`为例，基于前述代码的运行结果为`[1, 6] [8, 15] [16, 19] `，接下来具体分析上述代码的运行过程：

1. 执行第2行代码进行排序，结果如下

   ![对数据进行排序](/blog_img/algorithm/merge-overlapping-time-intervals/intervals-before-and-after-sort.png "对数据进行排序")

2. 此时结果数组`merged`为空

   ![初始结果数据](/blog_img/algorithm/merge-overlapping-time-intervals/init-result-data.png "初始结果数据")

3. 由于`merged`为空，将执行第10行的判断逻辑`merged.isEmpty()`，直接将1个遍历对象`[1,3]`加入`merged`中去

   ![第1次遍历](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-1-result.png "第1次遍历")

4. 接下来遍历的数据为`[2,6]`，对第10行的`merged.isEmpty()`不满足，同时由于当前遍历数据的结束区间为6，而`merged`中最后1个区间的结束数字为3，故第10行的`merged.get(merged.size() - 1).get(1) < start`也同样不满足，此时即表示检测到区间重叠

   ![检测到重叠区间](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-2-check.png "检测到重叠区间")

5. 由于有重叠区间，故执行第14行的代码，由于排序时是按照开始位置进行排序的，故**每次当前遍历区间的开始位置一定不会大于前面所有区间的开始位置**，而结束位置没有进行排序，故只需重新设置结束区间即可。

   由于结束位置的大小存在不确定性，实际设置时需要将当前区间的结束位置和重叠区间的结束位置取最大值，然后重新设置。在本例中由于当前区间的结束位置为6，比`merged`中最后一个区间的结束位置3大，故重叠区间的结束位置为6
   ![第2次遍历](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-2-result.png "第2次遍历")

6. 接下来遍历的数据为`[4,5]`，当执行第10行代码时，两个判断条件都不满足，同样表示检测到重叠区间，执行第14行的代码

   ![检测到重叠区间](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-3-check.png "检测到重叠区间")

7. 同步骤5中的判断逻辑类似，由于当前区间的结束位置为5，比`merged`中最后一个区间的结束位置6小，故重叠区间的结束位置依旧为6保持不变，此时意味着当前遍历的区间为一个重叠子区间

   ![第3次遍历](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-3-result.png "第3次遍历")

8. 接下来遍历的数据为`[8,15]`，由于当前区间的开始位置为8，比`merged`中最后一个区间的结束位置6大，此时意味着遇到没有重叠的区间，故第10行中的判断逻辑`merged.get(merged.size() - 1).get(1) < start`会执行

   ![识别到新区间](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-4-check.png "识别到新区间")

9. 将该区间加入到`merged`中的结果如下

   ![第4次遍历](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-4-result.png "第4次遍历")

10. 接下来遍历的数据为`[16,19]`，其判断逻辑和步骤8中的类似，同样会执行第10行中的判断逻辑，最终结果如下，理论分析结果与实际运行结果一致，至此整个过程分析完毕。

    ![第5次遍历](/blog_img/algorithm/merge-overlapping-time-intervals/iteration-5-result.png "第5次遍历")

## 改进

理解了原有的算法逻辑之后，只需要修改识别新区间的判断条件即可，将判断条件修改为相邻，在本文中即是将第10行中的判断条件从`< start`修改为`< start -1 `

```java
// 原有的只是合并重叠区间
if (merged.isEmpty() || merged.get(merged.size() - 1).get(1) < start) {
    merged.add(Arrays.asList(start, end));
} 

//---------------------------------------------------------------------

// 修改为相邻区间也能合并
if (merged.isEmpty() || merged.get(merged.size() - 1).get(1) < start - 1) {
    merged.add(Arrays.asList(start, end));
} 
```

完整代码如下

```java
public List<List<Integer>> mergeIntervals(List<List<Integer>> configs) {
    configs.sort(Comparator.comparingInt(a -> a.get(0)));
    List<List<Integer>> merged = new ArrayList<>();

    for (List<Integer> config : configs) {
        int start = config.get(0);
        int end = config.get(1);

        // 如果 merged 为空或当前区间与最后一个合并区间不相邻
        if (merged.isEmpty() || merged.get(merged.size() - 1).get(1) < start - 1) {
            merged.add(Arrays.asList(start, end));
        } else {
            // 有重叠或相邻，更新最后一个合并区间的结束点
            merged.get(merged.size() - 1).set(1, Math.max(merged.get(merged.size() - 1).get(1), end));
        }
    }

    return merged;
}
```

