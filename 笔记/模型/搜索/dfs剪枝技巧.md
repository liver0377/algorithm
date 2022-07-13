**剪枝**

剪枝的目的是为了减小搜索树的规模，尽早排除搜索树中不必要的分支，在`dfs`中，有以下几种常见的剪枝方法

- **优化搜索顺序**

  通过优化搜索树的顺序，来改变搜索树的结构，通常是从分支数量较小的节点开始搜索，目的是剪枝的时候能够一次性能够减掉更多的点

  ![image-20220713144742488](http://www.cdn.liver0377.xyz/typora/202207131447596.png)

- 排除等效冗余





- **可行性剪枝**

  在`dfs`过程中，应该及时对当前状态进行检查，如果无法到达最终目的地，就应该提前进行回溯，以避免遍历不必要的节点

- **最优性剪枝**

  在`dfs`过程中通常会维护一个全局最优解，如果发现当前花费的代价已经超过了最优解，那么当到达边界时，代价只会比现在大，因此应该提前进行回溯

- 记忆化搜索

  也就是空间换时间，记录搜索结果



### 小猫爬山

![image-20220713145518369](http://www.cdn.liver0377.xyz/typora/202207131455466.png)





**解题思路**

- 从前到后遍历所有的猫, 维护已经使用的车的数量`k`, 以及每辆车的当前载重`sum`

- 对于任意一只猫`i`, 尝试将其放在1 ~ k的任意一辆车上， 还可以为其设置一辆新车

  这样，一个节点最多有`k + 1`个子节点

- 剪枝操作

  - 如果将较重的猫先放入车中，这样后面的节点肯定会减小，因为有的猫可能放不上这辆车

    也就是**优化搜索顺序**

  - 如果车的数量已经超过最优解，可以提前回溯，即**最优性剪枝**



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 20;

int w[N];
int sum[N];
int n, m;
int ans = N;

// u: 当前正准备放置的小猫的下标
// k: 已经使用的车的数目
void dfs(int u, int k) {
    // 最优性剪枝
    if (k >= ans) return;
    if (u == n) {
        ans = k;
        return ;
    }
    
    for (int i = 0; i < k; i++) {
        // 可行性剪枝
        if (sum[i] + w[u] > m) continue;
        sum[i] += w[u];
        dfs(u + 1, k);
        sum[i] -= w[u];  // 恢复现场
    }
    
    // 使用新的缆车
    sum[k] += w[u];
    dfs(u + 1, k + 1);
    sum[k] -= w[u];
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) cin >> w[i];
    
    sort(w, w + n, [&](int a, int b) {
        return a > b;
    });
    
    dfs(0, 0);
    
    cout << ans << endl;
}
```





### 数独

![image-20220713164339858](http://www.cdn.liver0377.xyz/typora/202207131643913.png)



**解题思路**

- 基本的流程如下

  每次找到一个空格，尝试在该位置放置不同的数字，保证放置的数字合法，`dfs`递归判断

- 关键在于如何进行剪枝

  - 优化搜索顺序

    每次选取空格时，有多个空格可以选择，不同的空格可以选择的数字的个数也是不同的，每次选择

    **备选数字数目最小的**一个空格，这样可以保证它的后续节点数目更小

  - 位运算优化

    当判断一个空格有哪些备选数字可供选择时，可以使用一个二进制串来表示每一行，列，九宫格可以放置的数字

    只要锁定了空格所在的行列，九宫格，然后对二进制串进行与运算，就能够判断该空格可以放置那些数字

    



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

#define lowbit(x) ((x) & (-x))  // lowbit算法，求解最小的一位1
using namespace std;

const int N = 9;
const int M = 1 << N;

char s[100];// 整个9 * 9方格所代表的字符串   
int ones[M];  // ones[i]: 数字i的二进制表示中1的数目
int map[M];   // map[i]: log_2(i)
int rows[N], cols[N]; // 存储每一行/列的状态
                      // 例:
                      // rows[i], rows[i]的二进制表示为1000 0001 0
                      // 那么就表明第i行的数字1,8还可以使用 
int cells[3][3];       // 总共有3 * 3个九宫格, cell[i][j]表示九宫格(i, j)中那些数字还能够使用
                      // 同样是使用二进制表示

// flag == true: 将(i, j)位置的数字改为t + 1
// flag = false: 将(i, j)位置的数字t + 1删除掉
// 0 <= t <= 8
void draw(int i, int j, int t, bool flag) {
    if (flag) 
        s[i * N + j] = '1' + t;
    else 
        s[i * N + j] = '.';
    int v = 1 << t;
    if (!flag) 
        v = -v;
    // 更新(i, j)所在的行, 列, 九宫格的状态    
    rows[i] -= v;
    cols[j] -= v;
    cells[i / 3][j / 3] -= v;
}

// 初始化整个方格, 并且返回方格中空格的数目
int init() {
    for (int i = 0; i < N; i++) rows[i] = cols[i] = (1 << N) - 1;
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            cells[i][j] = (1 << N) - 1;
        }
    }
    
    int cnt = 0;
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            if (s[i * N + j] != '.') {
                draw(i, j, s[i * N + j] - '1', true);  // 这里的目的是为了更新rows, cols, cells
            } else {
                cnt ++;
            }
        }
    }
    
    return cnt;
}

// 返回(i, j)位置所能够填的数字
int get(int i, int j) {
    return rows[i] & cols[j] & cells[i / 3][j / 3];
}

// cnt: 方格还剩下多少空格
bool dfs(int cnt) {
    if (!cnt) return true;
    
    int minv = 10;
    int x, y;     // 优化搜索顺序，找到分支数目最小的空格
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            if (s[i * N + j] == '.') {
                int state = get(i, j);
                if (ones[state] < minv) {
                    minv = ones[state];
                    x = i, y = j;
                }
            }
        }
    }
    
    // 尝试填充不同的数字
    for (int i = get(x, y); i != 0; i -= lowbit(i)) {
        int t = map[lowbit(i)];    // 待填充的数字 - 1, 
        draw(x, y, t, true);
        if (dfs(cnt - 1)) return true;
        draw(x, y, t, false);
    }
    return false;
}

int main() {
    // 打表
    for (int i = 0; i < 1 << N; i++) {
        for (int j = i; j != 0; j -= lowbit(j)) {
            ones[i]++;
        }
    }
    for (int i = 0; i < N; i++) map[1 << i] = i;
    
    while (cin >> s, s[0] != 'e') {
        int k = init();
        dfs(k);
        puts(s);
    }
    return 0;
}
```

