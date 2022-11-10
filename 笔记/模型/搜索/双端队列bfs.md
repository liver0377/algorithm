- 双端队列bfs

  对于边权要么是0，要么是1的双端队列的无向图，可以使用双端队列bfs来计算



### 电路维修

![image-20220706190112198](http://www.cdn.liver0377.xyz/typora/202207061901290.png)

![image-20220706190122765](http://www.cdn.liver0377.xyz/typora/202207061901817.png)

![image-20220706190136977](http://www.cdn.liver0377.xyz/typora/202207061901020.png)





**解题思路**

- 此题的关键是将题意转换成一个**0,1边权图**问题

- 所谓**0, 1边权图**问题是指图中存在边权为0，1两种情况

- 0, 1边权图相对于边权唯一的图来说有着很多特点

  - 每次从队头取出元素，并进行拓展其他元素时

    - 若拓展某一元素的边权是0，则将该元素插入到队头
    - 若拓展某一元素的边权是1，则将该元素插入到队尾

    这样做是为了维护队列的**两阶段性**以及**单调性**

  - 当一个节点第一次`bfs`搜索到时，其距离并不一定是最短的

    - 假设有a, b两个节点，他们距离起点的距离均为`x`， 其中`a`节点与`c`的边权为1，`b`节点与`c`的边权为0,并且`a`节点排在`b`节点前面

      ![image-20220706190837310](http://www.cdn.liver0377.xyz/typora/202207061908340.png)

    - 显然`c`经过`b`到达起点的距离更小，因此当`xa`出完队列，`xb`再出队的时候，应该更新`c`到起点的距离
    
    因此, 在一次遍历中, 一个节点可以被重复加入队列

- 就具体题目而言，如果从某个格点`i`能够直接走到格点`j`，那么这两个格点之前就连一条边权为0的边

  否则，如果从节点`i`走到节点`j`需要旋转方格，那么这两个格点之间需要连接一条边权为1的边

- 同时，对于一个具体的方格而言，总有一些节点是永远都到达不了的，只要横纵坐标之和为偶数的才能都到达，因此起点横纵坐标之和为偶数(0 + 0), 并且只能走斜线意味着横纵坐标总是同时加一或减一



**算法流程**

- 首先将所有点的`dist`初始化为无穷，然后将起点的`dist`设置为0，将起点加入队列
- 若队列不空，取出队头元素，若该元素已经访问过，取出下一个元素，直至取出一个未访问过的元素
- 使用队头元素尝试更新周围的所有节点的`dist`，设队头元素与其出顶点的边权为`w`
  - 若`w == 0`，则将出顶点加入队列首部
  - 若`w == 1`, 则将出顶点加入队列尾部

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <deque>

using namespace std;

#define x first
#define y second
const int N = 510;

typedef pair<int, int> PII;
char g[N][N];
int dist[N][N];
bool st[N][N];

const int dx[] = {-1, -1, 1, 1};
const int dy[] = {-1, 1, 1, -1};
const int ix[] = {-1, -1, 0, 0};
const int iy[] = {-1, 0, 0, -1};
const char cs[] = "\\/\\/";   // cs[i]: 从当前点出发，到达格点i所应该经过的边的形状
    
int n, m;

int bfs() {
    deque<PII> q;            // (x, y)
    
    memset(st, 0, sizeof st);
    memset(dist, 0x3f, sizeof dist);
    dist[0][0] = 0;
    q.push_back({0, 0});
    
    while (!q.empty()) {
        auto t = q.front();
        q.pop_front();
        
        if (t.x == n && t.y == m) return dist[t.x][t.y];  // 当一个格点出队时，其dist最小
        if (st[t.x][t.y]) continue;   // 因为一个点可能会被多次放入队列，当它第一次出队时，其dist已经是最小了
                                      // 不用再管了
        st[t.x][t.y] = true;
        for (int i = 0; i < 4; i++) {
            int nx = t.x + dx[i];
            int ny = t.y + dy[i];  // 周围的格点坐标
            int gx = t.x + ix[i];
            int gy = t.y + iy[i];  // 周围的斜线在g数组中的坐标
            if (nx < 0 || ny < 0 || nx > n || ny > m) continue;
            int w = g[gx][gy] != cs[i];
            
            if (dist[t.x][t.y] + w <= dist[nx][ny]) {   // 只要有新点出队，就用它尝试更新与它相连的点
                dist[nx][ny] = dist[t.x][t.y] + w;
            
            }
            if (w) {
                q.push_back({nx, ny});
            } else {
                q.push_front({nx, ny});
            }
        }
    }
    return -1;
}
int main() {
    int T;
    cin >> T;
    while (T--) {
        scanf("%d%d", &n, &m);
        
        for (int i = 0; i < n; i++) {
            scanf("%s", g[i]);
        }
        if (n + m & 1) 
            puts("NO SOLUTION");
        else 
            cout << bfs() << endl;
        
    }
    
    return 0;
}
```

- `ix, iy`的确定方法

  可以直接举一个具体的例子来倒求出`ix`, `iy`数组

  假设当前点的坐标为(1, 2)

  ![image-20221110210205369](http://www.cdn.liver0377.xyz/typora/202211102102950.png)

  其周围四条边在`g`数组中的下标分别为(0, 1), (0, 2), (1, 2), (1, 1)

  拿这些边在`g`数组中的下标-当前点的坐标即可得到点的坐标与边的坐标的关系: (-1, -1), (-1, 0), (0, 0), (0, -1), 即`ix, iy`

  

  

  

  
