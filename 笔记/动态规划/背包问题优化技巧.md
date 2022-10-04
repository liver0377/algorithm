### 滚动数组

若当前状态只依赖于上一层的状态时, 就可以使用滚动数组

- 写法: 在状态的第一维加上 % 2



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
              f[i % 2][j] = f[(i - 1) % 2][j];
              if (j >= v[i]) {
                  f[i % 2][j] = max(f[i % 2][j], f[(i - 1) % 2][j - v[i]] + w[i]);
              }
          }
      }
      
      cout << f[n % 2][m] << endl;
      return 0;
  }
  ```

  

### 倒序循环

该方法通常应用于二维状态表示

若当前状态只依赖于状态矩阵中左上角的状态, 那么就可以使用倒序循环

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

  