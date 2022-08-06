- `dfs`的基本结构

  ```cc
  dfs(int u, int acc, ...) {    // u表示当前正在访问底第几层 acc为自上向下累计的值，仅当到达给定层数才会返回
      if (u == 给定层数) 返回acc
      for u in v 的相邻节点
      if visited[u] == false then
        visited[u] = true;
        DFS(u + 1, acc + cost);
        visited[u] = true; // 恢复现场
      
      return ...
  }
  ```
  
  
  
  ```cc
  DFS(v) // v 可以是图中的一个顶点，也可以是抽象的概念，如 dp 状态等。
    在 v 上打访问标记
    for u in v 的相邻节点
      if u 没有打过访问标记 then
        DFS(u)
      end
    end
  end
  ```



### 迷宫

![image-20220711161907738](http://www.cdn.liver0377.xyz/typora/202207111619789.png)

- 经典的迷宫问题，采用`dfs`模板即可



**代码实现**

```cc
#include <iostream>
#include <cstring>

using namespace std;

const int N = 110;

char g[N][N];
bool st[N][N];
int ax, ay, bx, by;
int n;
const int dx[] = {-1, 0, 1, 0};
const int dy[] = {0, 1, 0, -1};

// 判断能否从{x, y}走到{bx, by}
bool dfs(int x, int y) {
    if (g[x][y] == '#') return false;
    if (x == bx && y == by) return true;
    st[x][y] = true;  // 将{x, y}标记为已访问，防止其邻居节点再次搜索该节点
    for (int i = 0; i < 4; i++) {
        int nx = x + dx[i];
        int ny = y + dy[i];
        if (nx < 0 || ny < 0 || nx >= n || ny >= n) continue;
        if (st[nx][ny]) continue;
        if (dfs(nx, ny)) return true;
    }
    return false;
}

int main() {
    int T;
    cin >> T;
    while (T --) {
        cin >> n;
        for (int i = 0; i < n; i++) cin >> g[i];
        cin >> ax >> ay >> bx >> by;
        memset(st, 0, sizeof st);
        if (dfs(ax, ay)) puts("YES");
        else puts("NO");
    }
    return 0;
}
```



### 红与黑

![image-20220711164517980](http://www.cdn.liver0377.xyz/typora/202207111645028.png)



**解题思路**

- 此题是使用`dfs`求解`flood fill`模型的模板



**代码实现**

```cc
#include <iostream>
#include <cstring>

using namespace std;

const int N = 22;
int n, m;
char g[N][N];
bool st[N][N];
const int dx[] = {-1, 0, 1, 0};
const int dy[] = {0, 1, 0, -1};
    
int dfs(int x, int y) {
     
    int cnt = 1;              // 本身为黑块
    st[x][y] = 1;
    for (int i = 0; i < 4; i++) {
        int nx = x + dx[i];
        int ny = y + dy[i];
        if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
        if (g[nx][ny] != '.') continue;
        if (st[nx][ny]) continue;
        
        cnt += dfs(nx, ny);  // 加上每个方向的黑块的数目
    }
    return cnt;
}

int main() {
    while (cin >> m >> n, n || m) {
        for (int i = 0; i < n; i++) cin >> g[i];
        int start_x, start_y;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (g[i][j] == '@') {
                    start_x = i;
                    start_y = j;
                }
            }
        }
        memset(st, 0, sizeof st);
        cout << dfs(start_x, start_y) << endl;
    }
    return 0;
}
```

