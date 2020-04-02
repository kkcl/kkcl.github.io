---
title: leetcode153 154 Find Minimum in Rotated Sorted Array I/II
date: 2019-11-22 21:07:54
categories: 题解
tags:
- leetcode
- 数组
- 最小值
- 分治
- 二分查找
copyright: true
---

# leetcode153 154. Find Minimum in Rotated Sorted Array I/II

题目来源[leetcode153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/) 和[leetcode154](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) 要求在排序的旋转数组中寻找最小值。最小值可以很简单的遍历一次数组得到，时间复杂度为$O(n)$但是没有用到题目给的性质，所以会超时。

<!--more-->

## 解题思路

### leetcode153 无重复元素

先分析leetcode153中没有重复值的情况，采用二分查找的思路，设输入的数组长度为n表示为$A[n]$,经过观察可以发现pivot将A划分成了两个递增子数组L，R。使用指针p指向A的第一个元素A[0],指针q指向A的最后一个元素$A[n-1]$,使用指针m指向A的中间元素$A[\frac{n}{2}]$。如果中间元素位于第一个递增数组L，则有关系：A[p]<A[m],A[m]>A[q],此时最小元素在m与q之间，将p移动到m，原来规模为n的问题变成规模为$q-m=\frac{n}{2}$的子问题。如果中间元素位于第二个递增数组R，则有关系：A[p]>A[m],A[m]<A[q],此时最小元素在p与m之间，将q移动到m，原来规模为n的问题变成规模为$m-p=\frac{n}{2}$的子问题。直到最后q=p+1时，问题规模下降到2的情况，得到答案最小的元素为q指针指向的元素。可以看到每一次迭代，都会将n规模的问题下降到$\frac{n}{2}$规模。

### leetcode154 有重复元素

在有重复值的情况下，会出现$A[m]==A[p]$的情况，这个时候是无法判断到底左右两边谁为递增序列。如
$$
[2, 2, 2, 2, 1,1,1, 2]
$$
这时候的$A[m]==A[3]$与$A[p]==A[0]$是相等的都是2，所以只能把两边的最小值，递归的调用求解出来。

## 复杂度

### leetcode153 无重复元素

可以画出递归图:

![](https://res.cloudinary.com/bravey/image/upload/v1574430318/blog/coding/leetcode_153_reduction_graph.jpg )

用$T(n)​$表示对于一个长度为n的旋转数组进行查找最小值需要的次数，对于每一次迭代都有$T[n]=T[{\lfloor\frac{n}{2}\rfloor]+C}​$,C是一个常数。所以有
$$
\begin{split}

T(n)&=T[{\lfloor\frac{n}{2}\rfloor]+C}\\

&=T[{\lfloor\frac{n}{2^2}\rfloor]+2C}\\

&=T[{\lfloor\frac{n}{2^3}\rfloor]+3C}\\

\dots\\

&=T[{\lfloor\frac{n}{2^k}\rfloor]+kC}\\

\end{split}
$$

设在第k次迭代的时候长度下降到2，  此时有$\frac{n}{2^k}=2$解出$k=\log_2n-1$，所以有$T[n]=(\log_2n-1)C=O(\log_2n)$，所以算法的时间复杂度为$O(logn)$

### leetcode154 有重复元素

最坏的情况，当出现$A[m]==A[p]$的情况的时候有递推表达式：
$$
\begin{split}
T(n)&=2T[{\lfloor\frac{n}{2}\rfloor]+C}\\
&=2^2T[{\lfloor\frac{n}{2^2}\rfloor]+C+2C}\\
&=2^3T[{\lfloor\frac{n}{2^3}\rfloor]+C+2C+2C}\\
\dots\\
&=2^kT[{\lfloor\frac{n}{2^k}\rfloor]+(2k-1)C}\\
\end{split} 
$$
设当第k次的时候下降到n=1，因此有$\frac{n}{2^k}=1$推出$k=\log_2n$所以有$T(n)=n+(2\log_2n-1)C=O(n)$,所以时间复杂度为O(n)。如果没有出现$A[m]==A[p]$的情况则和无重复值的情况一样，时间复杂度为$O(n)$

## 代码

使用迭代和循环两种方式实现。 循环的时候比较的选择？mid是该和左边还是右边比呢？我自己写的是和左边的比，但是我看leetcode的题解都是和右边的比。

### leetcode153 无重复元素

```c++
/*leetcode#153
https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/*/
#include<iostream>
#include<vector>
#include<map>
#include<algorithm>
#include<utility>
#include <string>
#include <stdio.h>

using namespace std;

class Solution {
public:
    int findMin(vector<int>& nums) {
        int len = nums.size();
        if(len==1 || nums[0]<nums[len-1]) return nums[0];//长度为1的时候或者不旋转的时候直接返回
        // return dc_find(nums, 0, len-1);
        return loop_find(nums, 0, len-1);
    }
private:
	int dc_find(vector<int>& nums, int lo, int hi){
		if((hi -lo)==1) return nums[hi];
		int ans = 0;
		int mid = lo + (hi -lo)/2;
		if(nums[mid]>nums[lo]) ans = dc_find(nums, mid, hi);
		else ans = dc_find(nums, lo, mid);
		return ans;
	}
	int loop_find(vector<int>& nums, int lo, int hi){
		int ans = 0;
		int mid = 0;
		while(lo<hi){
			if((hi-lo)==1) {
				ans = nums[hi];
				break;
			}
			mid = lo + (hi -lo)/2;
			if(nums[mid]>nums[lo]) lo = mid ;
			else hi = mid ;
		}
		return ans;
	}
};

int main(int argc, char const *argv[])
{
	/* code */
	ios::sync_with_stdio(false);
	std::vector<int> vec = {3, 4, 5, 1, 2};
	Solution Sol;
	cout<<Sol.findMin(vec)<<endl;
	system("pause");
	return 0;
}
```

### leetcode154 有重复元素

```c++
/*
leetcode#154
https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/
*/
#include<iostream>
#include<vector>
#include<map>
#include<algorithm>
#include<utility>
#include <string>
#include <stdio.h>

using namespace std;

class Solution {
public:
    int findMin(vector<int>& nums) {
        int len = nums.size();
        if(len==1 || nums[0]<nums[len-1]) return nums[0];
        return dc_find(nums, 0, len-1);
    }
private:
	int dc_find(vector<int>& nums, int lo, int hi){
		if((hi -lo)==1) return min(nums[lo], nums[hi]);// 与无重复值的情况不同
		int ans = 0;
		int mid = lo + (hi -lo)/2;
		if(nums[mid]>nums[lo]) ans = dc_find(nums, mid, hi);
		else if (nums[mid]==nums[lo]){
			ans = min(dc_find(nums, lo, mid), dc_find(nums, mid, hi));
		}
		else ans = dc_find(nums, lo, mid);
		return ans;
	}
    
};

int main(int argc, char const *argv[])
{
	/* code */
	ios::sync_with_stdio(false);
	std::vector<int> vec = {2, 2, 2, 1, 1 ,2 };
	Solution Sol;
	cout<<Sol.findMin(vec)<<endl;
	system("pause");
	return 0;
}
```

需要注意的是：`if((hi -lo)==1) return min(nums[lo], nums[hi]);` 递归的退出条件， 与无重复值的情况不同是可能出现只有两个值的时候最小值在左边的情况，所以需要比较最小值，而不是直接返回第二个。比如$[2,3,2]$ 的情况，划分为两个子问题$[2, 3] 与[3,2]$第一个子问题的时候最小值在第一个元素。