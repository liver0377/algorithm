### 二分

**全局**

- `upper_bound()`
  - `upper_bound(begin, end, x)`: 返回第一个大于`x`的元素
  - `upper_bound(begin, end, x, cmp)`: 返回第一个大于`x`的元素, 使用自定义比较器
- `lower_bound()`
  - `lower_bound(begin, end, x)`: 返回第一个大于等于`x`的元素
  - `lower_bound(begin, end, x, cmp)`, 返回第一个大于等于`x`的元素, 使用自定义比较器

- 例

  在有序`int`数组中查找大于等于给定整数的最小下标

  ```cc
  int i = lower_bound(a, a + n, x) - a;
  ```

  在`vector<int>`中查找小于等于`x`的最大整数

  ```cc
  int i = *--upper_bound(a.begin(), a.end(), x);
  ```

  

**容器**

- `set`
  - `upper_bound(key)`: 返回大于给定`key`的第一个元素的迭代器
  - `lower_bound(key)`: 返回大于等于给定`key`的第一个元素的迭代器
- `map`
  - `upper_bound(key)`: 返回大于给定`key`的第一个元素的迭代器
  - `lower_bound(key)`: 返回大于等于给定`key`的第一个元素的迭代器



容器自身的算法通常会比全局更快





### 去重

**全局**

- `unique`

  - `unique(begin, end)`: 对给定区间内的元素去重, 重复元素会被排到后面, 返回去重之后, 末尾元素的下一个位置

- 例

  计算去重之后的元素数目

  ```cc
  int m = unique(a.begin(), a.end()) - a.begin();
  ```





### 查找

**全局**

- `find(begin, end, x)`: 返回`x`在给定序列中的下标/迭代器

**容器**

- `set`
  - `find(key)`: 查找等于`key`的元素, 返回其迭代器, 时间复杂度$O(log(n))$
- `map`
  - `find(key)`: 查找关键字为`key`的元素, 返回其迭代器, 时间复杂度$O(log(n))$





### iota

定义在`<numeric>`中

- `iota(begin, end, init)`: 在[begin, end)范围内产生一个初始值为`init`的递增序列



- 例

  获取一个数组在排序之后, 其元素在排序前数组中的下标

  ```cc
  # v: 4 2 1 3 -->
  # t: 2 1 3 0
  iota(t.begin(), t.end(), [&](int a, int b){
      return v[a] < v[b];
  }); 
  ```

  

