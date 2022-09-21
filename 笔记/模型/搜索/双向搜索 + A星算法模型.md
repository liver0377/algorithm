### 字串变换

![image-20220707171000777](http://www.cdn.liver0377.xyz/typora/202207071710860.png)

![image-20220707171010539](http://www.cdn.liver0377.xyz/typora/202207071710585.png)

**解题思路**

- 此题由于字符串长度上限为20, 最多有6种转换规则

- 在最坏情况下，如果从每个位置开始都能够应用所有的转换规则，那么一个状态就可以拓展出120种状态

- 如果之间使用`bfs`, 需要拓展出大量状态，在10步之内，时间复杂度可以达到120^10, 很显然是无法通过的

- 这题可以使用**双向搜索优化**

  - 双向搜索的基本思路是从起点和终点同时开始搜索，使用两个队列来维护它们搜索的集合，如果在搜索过程中产生了交集

    也就是两个搜索路径搜索到了同一个点，那么就表明这两个点之间是连通的

    路径的长度为公共点到起点和终点的距离之和

- 时间复杂度缩减为了2 * 120 ^ 5, 时间复杂度大大减小

  > 由于从两个方向同时搜索，可以将每个方向的路径长度视作原来的一半，也就是指数变为一半



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
#include <unordered_map>
#include <string>

using namespace std;

const int N = 6;
int n;
string a[N], b[N];

// 从q队列拿出一个元素进行拓展，如果拓展过程中发现了公共点，那么就直接返回路径长度
int extend(queue<string> &q, unordered_map<string, int> &da,
              unordered_map<string, int> &db, string a[], string b[]) {
        string t = q.front();
        q.pop();
        
        for (int i = 0; i < t.size(); i ++) {
            for (int j = 0; j < n; j++) {
                if (t.substr(i, a[j].size()) == a[j]) {
                    string state = t.substr(0, i) + b[j] + t.substr(i + a[j].size());   // 新状态
                    if (da.count(state)) continue;
                    // 从起点到点t的距离 + 从点t到state的距离  + 从state到终点的距离
                    if (db.count(state)) return da[t] + 1 + db[state];
                    da[state] = da[t] + 1;
                    q.push(state);
                }
            }
        }
        
    return 11;   // 返回一个长度大于10的值表示没有交集
}


int bfs(string start, string end) {
    if (start == end) return 0;
    queue<string> qa, qb;
    unordered_map<string, int> da, db;  // da[i]: 从起点走到状态i需要的步数
                                        // db[i]: 从终点走到状态i需要的步数
    qa.push(start);
    qb.push(end);
    da[start] = 0;
    db[end] = 0;
    
    // 仅当两个队列都不为空时才继续搜索
    // 如果队列a空了，队列b非空，那么表明a, b一定没有交集，因为
    // 在交集确实存在的情况下, a会持续搜索，直到搜索到与b相交
    // 一旦搜索到交集就会立马return 
    while (qa.size() && qb.size()) {
        int t;
        // 每次从较小的队列拓展，可以减少搜索的次数
        if (qa.size() <= qb.size()) 
            t = extend(qa, da, db, a, b);
        else 
            t = extend(qb, db, da, b, a);
        if (t <= 10) return t;
    }
    
    return -1;
}
int main() {
    string start, end;
    cin >> start >> end;
    while (cin >> a[n] >> b[n]) n ++;
    
    int t = bfs(start, end);
    if (t != -1)
        cout << t << endl;
    else {
        cout << "NO ANSWER!" << endl;
    }
    
    return 0;
}
```





### 八数码

![image-20220707185128195](http://www.cdn.liver0377.xyz/typora/202207071851255.png)

![image-20220707185144041](http://www.cdn.liver0377.xyz/typora/202207071851094.png)

![image-20220707185151488](http://www.cdn.liver0377.xyz/typora/202207071851523.png)



**解题思路**

- 此题使用**A*算法**求解

- **A*算法**的基本思路就是使用**启发式搜索**，当搜索其它点时，会估计每个节点到终点的距离，利用每个节点到起点的实际距离`dist`

  加上到终点的估计距离`f`，优先选出`dist + f`最小的一个点进行拓展，也就是说, 估计距离`dist + f`更小的点会被优先取出，进而

  减少搜索次数

- **A*算法**有着一些限制与性质

  - 每个点到终点的估计距离`f`必须要小于等于它到终点的实际距离

  - 当终点出队时，其与起点的距离`dist`最小，即最优解，而中间节点则不一定

    - 证明

           假设终点第一次出队列时不是最优 
            则说明当前队列中存在点u
               有 d[估计]< d[真实]
            d[u] + f[u] <= d[u] + g[u] = d[队头终点]
            即队列中存在比d[终点]小的值,
           但我们维护的是一个小根堆,没有比d[队头终点]小的d[u],矛盾
       
          证毕

  - **A*算法**仅适用于有解的情况，如果无解的话，其仍然会搜索所有的节点，并且由于使用优先级队列来代替普通队列，一次拓展操

    作的时间复杂度为$O(lg(n))$, 相对于普通队列$O(1)$而言更高

  - **A*算法**不适用于有负权边的情况

- 当估计函数`f`返回0时，该算法就等同于堆优化`Dijkstra`





**代码实现**

```cc
#include <iostream>
#include <cstring>
#include <queue>
#include <unordered_map>
#include <algorithm>

using namespace std;
typedef pair<int, string> PIS;

unordered_map<string, pair<string, char>> pre;    // pre[i].first 状态i的前一个状态
                                                  // pre[i].second 从前一个状态到状态i所经过的操作
unordered_map<string, int> dist;                       
priority_queue<PIS, vector<PIS>, greater<PIS>> heap;   // 按照估计距离dist + f进行排序的小根堆
string ed = "12345678x";
const int dx[] = {-1, 0, 1, 0};
const int dy[] = {0, 1, 0, -1};
char op[] = "urdl";
    
// 返回每个状态到终点的估计距离
int f(string state) {
    int res = 0;
    for (int i = 0; i < 9 ;i++) {
        if (state[i] != 'x') {
            int t = state[i] - '1';
            res  += abs(i / 3 - t / 3) + abs(i % 3 - i % 3);
        }
    }
    return res;
}

void bfs(string start) {
    heap.push({f(start) + 0, start});
    dist[start] = 0;
    
    
    while (!heap.empty()) {
        auto t = heap.top();
        heap.pop();
        
        string state = t.second;      // 队首状态
        if (state == ed) return;
        int step = dist[state];       // 队首到起点距离
        
        int k = state.find('x');
        int x = k / 3, y = k % 3;
        string old = state;
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];
            if (nx < 0 || ny < 0 || nx >= 3 || ny >= 3) continue;
            swap(state[nx * 3 + ny], state[x * 3 + y]);  // state成为新状态
            if (!dist.count(state) || step + 1 < dist[state]) {
                dist[state] = step + 1;
                pre[state] = {old, op[i]};
                heap.push({f(state) + dist[state], state});
            }
            swap(state[nx * 3 + ny], state[x * 3 + y]);
        }
    }
  
}

int main() {
    string start, seq;
    for (int i = 0; i < 9; i++) {
        char c;
        cin >> c;
        start += c;
        if (c != 'x')
            seq += c;
    }
    
    // 逆序对的个数
    int cnt = 0;
    for (int i = 0; i < 8; i++) {
        for (int j = i + 1; j < 8; j++) {
            if (seq[i] > seq[j]) cnt ++;
        }
    }
    if (cnt & 1) {
        puts("unsolvable");
        return 0;
    }
    
    bfs(start);
    
     string path;
    while (ed != start) {
        path += pre[ed].second;
        ed = pre[ed].first;
    }
    reverse(path.begin(), path.end());
    for (auto &e: path) cout << e;
    
    return 0;
}
```

- 这里的估计函数`f`使用的一个状态中每个数字直接移动到目的状态位置所需要的最小步数和

  ![image-20220707200504251](http://www.cdn.liver0377.xyz/typora/202207072005291.png)

  - 这里的步数和为1 + 3 + 3 + 2 + 1  + 3 = 12

    5移动到中间需要1步，4移动到最终位置要3步,其他数字类推...

  - 函数`f`的返回值`res`是一定满足`res`小于等于从当前状态到理想状态所移动的次数(边数)

  - 证明

    每次移动操作最多只会使一个数字向着理想位置移动一个方格，因此理想移动次数一定小于实际移动次数

- 为了判断该题是否有解，这里使用了一个性质

  - 如果给定的状态序列中，逆序对的个数为奇数，则一定无解

    



### 第K短路

![image-20220711143757950](http://www.cdn.liver0377.xyz/typora/202207111437018.png)



**解题思路**

- 采用**A*算法**解决这个问题

- 如果求解第K短路?

  - 不考虑节点是否被访问过，每次拿出堆顶元素时，将其连通的所有点全部加入小根堆，这样做可以使得一个点被访问多次

    也就是被不同的路径所到达

  - 当终点第一次出队时，这条路径与起点的距离最小，当终点第二次出队时，这条路径与起点的路径第二小，当终点第K次出队时，这条路径就是第K短路径

- 如何选取估价函数?

  这里使用每个点到终点的最短距离作为估价函数`f`,`f`满足**h(x) < h*(x)**的条件

  可以首先建立一个反向的图，也就是从终点开始建立单向图，然后使用`Dijkstra`算法预处理出每个点到终点的最短距离



**代码实现**

```cc
#include <cstring>
#include <iostream>
#include <algorithm>
#include <queue>

#define x first
#define y second

using namespace std;

typedef pair<int, int> PII;
typedef pair<int, PII> PIII;
const int N = 1010, M = 200010;

int n, m, S, T, K;
int h[N], rh[N], e[M], w[M], ne[M], idx;
int dist[N], cnt[N];
bool st[N];

void add(int h[],int a,int b,int c)
{
    e[idx] = b;
    w[idx] = c;
    ne[idx] = h[a];
    h[a] = idx++;
}

void dijkstra()
{
    priority_queue<PII,vector<PII>,greater<PII>> heap;
    heap.push({0,T});//终点
    memset(dist, 0x3f, sizeof dist);
    dist[T] = 0;

    while(heap.size())
    {
        auto t = heap.top();
        heap.pop();

        int ver = t.y;
        if(st[ver]) continue;
        st[ver] = true;

        for(int i=rh[ver];i!=-1;i=ne[i])
        {
            int j = e[i];
            if(dist[j]>dist[ver]+w[i])
            {
                dist[j] = dist[ver] + w[i];
                heap.push({dist[j],j});
            }
        }
    }
}

int astar()
{
    // heap {i, {j, k}}
    // k : 点的编号
    // i : 点k到起点的实际距离 + 点k到终点的估价距离
    // j : 点k到起点的实际距离
    priority_queue<PIII, vector<PIII>, greater<PIII>> heap;
    // 谁的d[u]+f[u]更小 谁先出队列
    heap.push({dist[S], {0, S}});
    while(heap.size())
    {
        auto t = heap.top();
        heap.pop();
        int ver = t.y.y,distance = t.y.x;
        cnt[ver]++;
        //如果终点已经被访问过k次了 则此时的ver就是终点T 返回答案

        if(cnt[T]==K) return distance;

        for(int i=h[ver];i!=-1;i=ne[i])
        {
            int j = e[i];
            /* 
            如果走到一个中间点都cnt[j]>=K，则说明j已经出队k次了，且astar()并没有return distance,
            说明从j出发找不到第k短路(让终点出队k次)，
            即继续让j入队的话依然无解，
            那么就没必要让j继续入队了
            */
            
                heap.push({distance+w[i]+dist[j],{distance+w[i],j}});
            }
        }
    }
    // 终点没有被访问k次
    return -1;
}

int main() {
    int n, m;
    cin >> n >> m;
    memset(h, -1, sizeof h);
    memset(rh, -1, sizeof rh);
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(h, a, b, c);
        add(rh, b, a, c);
    }
    cin >> S >> T >> K;
    if (S == T) K ++;   // 由于边数最小是1，所以自环不考虑，需要多出队一次
    // 1. 预处理每个点到终点的最短距离, 用于估价函数
    dijkstra();
    
    // 2. A*算法求解
    cout << astar();
    return 0;
} 
```

