---
title: 取样Sampling
date: 2013-11-10 21:17:00
categories:
 - IT Technology
tags:
 - Algorithm
 - Sampling
---

### 问题描述

从0~n-1的范围内的n个整数中，随机选取m个整数，不允许重复选择，每个数字被选中的概率相同

### n可知

#### Solution 1
小数据分析
假设n=5，m=1，那么
$$ P(0)=\frac{1}{5} $$
$$ P(1)=\frac{1}{4} (如果0未被选中, 否则P(1)=0) $$ 
$$ 总体概率为 \frac{4}{5} \* \frac{1}{4} = \frac{1}{5} $$
<!-- more -->

$$ P(2) = \frac{1}{3} (如果0，1未选中) $$
$$ 总体概率：\frac{4}{5} \* \frac{3}{4} \* \frac{1}{3} = \frac{1}{5} $$

假设n=5，m=2，那么
$$ P(0) = \frac{2}{5} $$
$$ P(1) = (0被选中)\frac{1}{4}, (0未选中)\frac{2}{4} $$
$$ 总体概率为 \frac{2}{5} \* \frac{1}{4} + \frac{3}{5} \* \frac{2}{4} = \frac{2}{5} $$

以此类推，可以得到结论，如果需要从r个剩余的数字中选择s个，那么选择该数字的概率为：s/r。如果select为0时，则不再继续选择。如果s=r，则概率为100%，此后的概率也均为100%。

```pseudo-code
void select(int n, int m)
{
	for (int i=0; i<n; i++)
	{
		if (rand() * (n-i) < m)
		{
			// choose i
			m--;
			if (m == 0)
			{
				// choose enough number
				break;
			}
		}
	}
}
```

rand() -> [0,1)

#### Solution 2
在初始为空的集合中插入随机整数，直到个数足够。

```pseudo-code
void select(int n, int m)
{
	set<int> s;
	while(s.size() < m)
	{
		s.insert(rand(n));
	}
}
```

时间复杂度：O(mlogm)
空间开销比较大？

#### Solution 3
思想：打乱数组的前m个整数，遍历前m个数，分别把i和random(i, n-1)交换位置

```pseudo-code
void select(int n, int m)
{
	for(int i=0; i<n; i++)
	{
		x[i] = i;
	}
	for(int i=0; i<m; i++)
	{
		swap(x[i], x[random(i,n-1)]);
	}
}
```

时间复杂度：如果不需要有序：O(m+n),如果需要有序：O(mlogm+n)。
该算法可以证明经过计算，每一个数位于前m的概率是相等的。

### n未知或n很大
如果n未知或者n很大，以至于不足以把所有数字装入内存，我们需要使用蓄水池抽样Reservoir Sampling

1.初始集合先选择0~m-1
2.从m开始，假设index=i，以概率m/i+1被选中，如果选中，则随机替换掉m集合中的任意一个元素。

证明
(1)当i=m
$$ P(i) = \frac{1}{m+1} + \frac{m}{m+1} \* \frac{m-1}{m} = \frac{m}{m+1} (i=[0,m-1]) $$
$$ P(m) = \frac{m}{m+1} $$

(2)当i=m+1
$$ P(i) = \frac{m}{m+1} \* (\frac{2}{m+2} + \frac{m}{m+2} \* \frac{m-1}{m}) = \frac{m}{m+2} (i=[0,m]) $$
$$ P(m+1) = \frac{m}{m+2} $$

...

(3)当i=j(j >= m)
$$ P(i) = \frac{m}{j} \* (\frac{j+1-m}{j+1} + \frac{m}{j+1} \* \frac{m-1}{m}) = \frac{m}{j+1} (i=[0,j]) $$

因此，每一个元素被选中的概率相同，都为m/n。