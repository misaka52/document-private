## 大纲

题库

- leetcode官网
  - 按照标签刷题，先集中刷简单，再刷中等。刷前200道即可
  - 一般一个题想 超过20分钟没思路，就可以直接看题解了。收藏题目
  - 建议题多刷几遍，加深印象

- 《剑指offer》



## 标签分类

### 数组

- 在一个顺序数组中，找两个和为指定值的两个数，可以采用双指针法，时间复杂度O(n)



## 排序

### 快排

```
// A:array; p:array left index; r:array right index
QUICKSORT(A, p, r)
	if p < r
		q = PARTITION(A, p, r)
		QUICKSORT(A, p, q - 1)
		QUICKSORT(A, q + 1, r)
		
PARTITION(A, p, r)
x = A[r]
i = p - 1
for j = p to r - 1
	if A[j] <= x
		i = i + 1
		exchange A[i] with A[j]
exchange A[i + 1] with A[r]
return i + 1
```

快排取数时可以随机取数，增加随机性



## 总结点

- int相加越界问题
  - 如：(l+r)/2  等价于 (r - l) / 2 + l

