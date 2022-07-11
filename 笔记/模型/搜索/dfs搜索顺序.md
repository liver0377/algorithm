### 马走日

![image-20220711172452362](http://www.cdn.liver0377.xyz/typora/202207111725310.png)



**解题思路**

- 此题涉及到一个恢复现场的问题， 什么时候需要恢复现场？

  - 在之前的**迷宫**， **红与黑**等连通性问题中，求得是一个图内部的连通性问题，每个节点只需要访问一次

    通过将`st[i][j]`置为`true`可以防止他们被重复访问

  - 然而，**马走日**是一类**状态变化问题**，每走一步都可以产生一个新的不同状态，可以画一个状态图来理解

    ![](http://www.cdn.liver0377.xyz/typora/202207111729019.png)

  - 每一次走法有多种决策，对每一步走法进行`dfs`完成之后，需要恢复到执行`dfs`之前的状态，然后尝试其他走法

**代码实现**

```cc
#include <iostream>
#include <cstring>

using namespace std;

const int N = 10;

int res;     // 可以遍历所有点的走法数目
bool st[N][N];
int n, m, x, y;

const int dx[] = {-2, -1, 1, 2, 2, 1, -1, -2};
const int dy[] = {1, 2, 2, 1, -1, -2, -2, -1};

// 正在搜索(x, y), (x, y)是正在搜索的第cnt个点
void dfs(int x, int y, int cnt) {
    if (cnt == n * m) {
        res ++;
        return;
    }
    
    st[x][y] = true;
    for (int i = 0; i < 8; i++) {
        int nx = x + dx[i];
        int ny = y + dy[i];
        if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
        if (st[nx][ny]) continue;
        
        dfs(nx, ny, cnt + 1);
    }
    st[x][y] = false;  // 恢复到执行dfs()之前的现场
}

int main() {
    int T;
    cin >> T;
    while (T --) {
        cin >> n >> m >> x >> y;
        res = 0;
        dfs(x, y, 1);
        cout << res << endl;
    }
    
    return 0;
}
```

