### 滚动数组

若当前状态只依赖于上一层的状态时, 就可以使用滚动数组

- 写法: 在状态的第一维加上 & 1



**例**

- 普通0/1背包

  ```cc
  int n, m;
  int v[N], w[N];
  int f[N][N];
  
  int main () {
      cin >> n >> m;
      
      for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
      
      for (int i = 1; i <= n; i++) {
          for (int j = 0; j <= m; j++) {
              f[i][j] = f[i - 1][j];
              if (j >= v[i]) {
                  f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
              }
          }
      }
      
      cout << f[n][m] << endl;
      return 0;
  }
  ```

  

- 滚动数组优化

  ```cc
  int n, m;
  int v[N], w[N];
  int f[2][N];
  
  int main () {
      cin >> n >> m;
      
      for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
      
      for (int i = 1; i <= n; i++) {
          for (int j = 0; j <= m; j++) {
              f[i & 1][j] = f[i - 1 & 1][j];
              if (j >= v[i]) {
                  f[i & 1][j] = max(f[i & 1][j], f[i - 1 & 1][j - v[i]] + w[i]);
              }
          }
      }
      
      cout << f[n & 1][m] << endl;
      return 0;
  }
  ```

  - & 的优先级低于算数运算符
  
  > 注意在使用滚动数组的时候, 一定要将初始化语句写在循环里边, 不然初始化的值可能会在滚动过程中被覆盖掉



### 倒序循环

该方法通常应用于二维状态表示

若当前状态[i, j]只依赖于状态矩阵中上一维i以及i左边的状态, 那么就可以使用倒序循环

原理是因为倒序循环时, 当前状态行的前面的部分存储的依旧是上一维的状态



- 用法: 将第二维逆序循环, 删除第一维



**例**

- 普通0/1背包

  ```cc
  int n, m;
  int v[N], w[N];
  int f[N][N];
  
  int main () {
      cin >> n >> m;
      
      for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
      
      for (int i = 1; i <= n; i++) {
          for (int j = 0; j <= m; j++) {
              f[i][j] = f[i - 1][j];
              if (j >= v[i]) {
                  f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
              }
          }
      }
      
      cout << f[n][m] << endl;
      return 0;
  }
  ```

  

- 倒序循环

  ```cc
  int n, m;
  int v[N], w[N];
  int f[N];
  
  int main () {
      cin >> n >> m;
      
      for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
      
      for (int i = 1; i <= n; i++) {
          for (int j = m; j >= v[i]; j--) {
              // f[j] = [j];
              // if (j >= v[i]) {
                  f[j] = max(f[j], f[j - v[i]] + w[i]);
              // }
          }
      }
      
      cout << f[m] << endl;
      return 0;
  }
  ```

  