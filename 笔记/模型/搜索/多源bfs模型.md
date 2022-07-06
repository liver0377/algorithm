### 矩阵距离

![image-20220706154121839](http://www.cdn.liver0377.xyz/typora/202207061553387.png)

![image-20220706154130067](http://www.cdn.liver0377.xyz/typora/202207061553337.png)



**解题思路**

- 本题要求的是每个点0到其它最近的点1的距离

- 具体思路就是首先将所有的点1加入队列，然后进行`bfs`，当矩阵中的任意一个顶点首次被某个点1搜索到时，其到该点1的距离就

  是最近的点1的距离



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

#define x first
#define y second

typedef pair<int, int> PII;
const int N = 1e3 + 10;
const int M = N * N;

char g[N][N];
int dist[N][N];
PII q[M];
int n, m;

void bfs() {
    memset(dist, -1, sizeof dist);
    int hh = 0, tt = -1;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (g[i][j] == '1') {
                dist[i][j] = 0;
                q[++ tt] = {i, j};
            }
        }
    }
    
    const int dx[] = {-1, 0, 1, 0};
    const int dy[] = {0, 1, 0, -1};
    while (hh <= tt) {
        PII t = q[hh ++];
        for (int i = 0; i < 4; i++) {
            int nx = t.x + dx[i];
            int ny = t.y + dy[i];
            if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
            if (dist[nx][ny] != -1) continue;
            
            dist[nx][ny] = dist[t.x][t.y] + 1;
            q[++ tt] = {nx, ny};
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) {
        scanf("%s", g[i]);
    }
    
    bfs();
    
    for (int i = 0; i < n; i ++) {
        for (int j = 0; j < m; j++) {
            cout << dist[i][j] << " ";
        }
        cout << endl;
    }
    
    return 0;
}
```





