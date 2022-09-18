### 树的深搜

树的深搜，就是对点`x`的多个分支，任意选择一条边走下去，执行递归，直至回溯到点`x`, 再考虑走点`x`的其它分支

![image-20220807141133236](http://www.cdn.liver0377.xyz/typora/202208071426871.png)

- 模板: 适用于所有图

  ```cc
  void dfs(int x) {
      visited[x] = 1;
      for (int i = h[x]; i != -1; i = ne[i]) {
          int j = e[i];
          if (visited[j]) continue;
          dfs(j);
      }
  }
  ```

  - 这个模板会遍历每个点以及每一条边(双向边会访问两次), 时间复杂度为$O(m + n)$
  
- 模板2 : 只适用于树

  ```cc
  void dfs(int x, int fa) {   // fa为x的父亲节点
      for (int i = h[x]; ~i; i = ne[i]) {
          int j = e[i];
          if (j == fa) continue;
          dfs(j, x);
      }
  }
  ```

  





### 时间戳



在`dfs`过程中，按照每个节点第一次被访问的顺序，依次对节点进行编号，这些编号被称为**时间戳**

![image-20220807142937349](http://www.cdn.liver0377.xyz/typora/202208071429387.png)



### dfs序

- 在`dfs`过程中，在刚进入递归后以及在即将回溯之前记录一次节点的编号，最后产生的长度为2N的节点序列就被称为树的**dfs序**

- 每个节点`x`的编号恰好在`dfs`序中出现两次，设这两次出现的位置在`dfs`序中的下标分别为`L[x]`, `R[x]`, 那么$[L[x], R[x]]$就是以

  `x`为根的子树的`DFS`序

**模板**

```cc
void dfs(int x) {
    a[++ m] = x;
    visited[x] = 1;
    for (int i = h[x]; i != -1; i = ne[i]) {
        int j = e[i];
        if (visited[j]) continue;
        dfs(j);
    }
    a[++ m] = x;
}
```

- 例：

  ![image-20220807142937349](http://www.cdn.liver0377.xyz/typora/202208071435764.png)

  上图的数的`dfs`序为: 1, 2, 8, 8, 5, 5, 2, 7, 7, 4, 3, 9, 9, 3, 6, 6, 4, 1





### 树的深度

- 树根的深度为0

- 深度计算模板

  ```cc
  void dfs(int x) {
      visited[x] = 1;
      for (int i = h[x; i != -1; i = ne[i]]) {
          int j = e[i];
          if (visited[j]) continue;
          d[j] = d[x] + 1;            // d[i]: 点i的深度
          dfs(j);
      }
  }
  ```





### 图的连通块划分

- 每对图进行一次深搜，就会划分出一个连通块，通过多次深搜操作，就可以划分出所有的连通块 

- 连通块划分模板

  ```cc
  void dfs(int x) {
      v[x] = cnt;
      for (int i = h[x]; i != -1; i = ne[i]) {
          int j = e[i];
          if (v[j]) continue;    // v[i]: 点i的连通块编号
          dfs(j);
      }
  }
  
  for (int i = 0; i <= n; i++) {
      if (!v[i]) {
          cnt ++;
          dfs(i);
      }
  }
  ```




