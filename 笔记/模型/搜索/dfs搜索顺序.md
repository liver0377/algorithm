





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





### 单词接龙

![image-20220712143515679](http://www.cdn.liver0377.xyz/typora/202207121435784.png)



**解题思路**

- 题意解释：

  - 给定一组单词，给定一个开头字母，如果一个单词的后缀与另一个单词的前缀相同，那么这两个单词就可以拼接在一起，

  - 重叠的长度`k`需要满足, $1 <= k < 两个单词的长度中的较小者$
  -  并且还要求每个单词最多只能使用两次
  - 求拼接后最长的单词

- 首先预处理出任意两个单词之间的重叠长度，由于题目要求的是最长的拼接单词，因此当两个单词有多个字母重叠时，选取一个字母进行重叠拼接即可，这样可以保证最终的单词最长

- 接下来使用`dfs`,每次可能会有多个可选的单词进行拼接，因此可以画成一颗树状

  ![image-20220712144206091](http://www.cdn.liver0377.xyz/typora/202207121442132.png)

  和上一题**马走日**是一样的状态类型，因此同样需要恢复现场



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <string>

using namespace std;

const int N = 22;

string words[N];
int g[N][N];  // g[i][j]: 单词i与单词j的最小重合长度
int used[N];  // used[i]: 单词i已经使用过的次数
int ans;
int n;

// 已经拼接完成的单词为word, 其中最后一个使用的单词为last
void dfs(string word, int last) {
    used[last] ++;
    ans = max((int)word.size(), ans);
    for (int i = 0; i < n; i++) {
        if (used[i] < 2 && g[last][i]) {
            dfs(word + words[i].substr(g[last][i]), i);
        }
    }
    used[last] --;  // 恢复现场
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) cin >> words[i];
    char start;
    cin >> start;
    
    // 1. 预处理出每个单词之间的最小重叠长度
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            for (int k = 1; k < min(words[i].size(), words[j].size()); k++) {
                if (words[i].substr(words[i].size() - k, k) == words[j].substr(0, k)) {
                    g[i][j] = k;
                    break;
                }
            }
        }
    }
    
    // 2. dfs
    for (int i = 0; i < n; i++) {
        if (words[i][0] == start) {
            dfs(words[i], i);
        }
    }
    
    cout << ans << endl;
    return 0;
}
```





### 分成互质组

![image-20220712153809739](http://www.cdn.liver0377.xyz/typora/202207121538782.png)



**解题思路**

- 这里的`dfs`遵循以下流程

  - 从前到后遍历所有的元素`x`

  - 按照组1, 组2，.., 组i的顺序

    - 尝试将元素`x`放入组`i`中
    - 如果元素x不能够放入组`i`,那么需要为其开一个新组`i + 1`

    > 为什么首先尝试将`x`放入最后一个组？
    >
    > - 如果将`x`放入新组`i + 1`中能够得到最优解，那么在`x`能够放入组`i`的情况下，将`x`从组`i + 1`移动到组`i`
    >
    >   一定是合法的，因为移动之后，组`i`,组`i + 1`中的所有元素一定依旧全部互质，组的数目只可能减少不可能增加
    >
    >   因此，优先将元素`x`放入当前最后一个组中
    >
    > - 这里是按照组1, 组2, 组3...的顺序尝试将每个元素放入组中，这样的枚举方式是不会遗漏的

![image-20220712153751022](http://www.cdn.liver0377.xyz/typora/202207121537072.png)



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 10;
int a[N];
int g[N][N];   // g[i][j]: 组i中的第j个元素
int st[N];
int n, ans = N;

// 欧几里得算法，判断a, b是否互质
int gcd(int a, int b) {
    return b ? gcd(b, a % b) : a;
}

// 检查元素x是否与g[]中的元素全部互质
bool check(int g[], int x, int len) {
    for (int i = 0; i < len; i++) {
        if (gcd(g[i], x) > 1) return false;
    }
    return true;
}

// gid: 正在尝试将元素加入哪个组
// idx: 组gid中的元素个数
// start: 正在遍历的元素编号
// cnt: 已经将多少个元素加入组中

void dfs(int gid, int idx, int start, int cnt) {
    if (gid >= ans) return;  // 剪枝，入股已经生成了超过ans个组，就没有必要继续遍历了
    if (cnt == n) ans = gid;
    
    bool flag = true;       // 判断是否需要新开一个组
    for (int i = start; i < n; i++) {
        if (!st[i] && check(g[gid], a[i], idx)) {
            st[i] = true;
            int old = g[gid][idx];
            g[gid][idx] = a[i];
            dfs(gid, idx + 1, i + 1, cnt + 1);
            g[gid][idx] = old;
            st[i] = false;
            flag = false;
        }
    }
    
    if (flag) {
        
        dfs(gid + 1, 0, 0, cnt);  // 这里的start参数为0，很关键
                                  // 虽然当前遍历到了第start个位置，但是start之前可能还有元素未加入分组
                                  // 考虑2, 4, 5, 8, 16
                                  // 首先将2加入分组1，在for循环中，会将5加入分组，加下来进入递归判断8
                                  // 从8开始的后面所有元素都无法放入分组1，因此需要开一个新的分组
                                  // 此时start为3, 而4, 也就是下标为1的元素仍然未放入分组，所以这里
                                  // 的参数需要填0
    }
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) cin >> a[i];
    
    dfs(1, 0, 0, 0);
    
    cout << ans << endl;
}
```

