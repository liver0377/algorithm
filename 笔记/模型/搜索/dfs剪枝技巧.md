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





### 木棒

![image-20220715152215859](http://www.cdn.liver0377.xyz/typora/202207151522954.png)



**解题思路**

- 题意转换: 给定一组数据，将它们分成若干组，每组的和相等, 求最小的和

- 思路：

  - **木棍**代表题目给的数据，**木棒**代表每一组

  - 直接枚举每组的和

  - 首先算出所有木根的长度总和`sum`，木棒的长度`length`应该可以被`sum`整除

  - 需要维护三个状态: **已经拼接完成的木棒数量**，**正在拼接的木棒长度**，**木棍的使用状态**

  - 对于正在拼接的木棒而言，尝试使用不同长度的木棍对其进行拼接，递归到不同的状态，如果所有的木棍都能够被使用

    那么表明该`length`为合法解

- 剪枝

  此题的关键在于剪枝

  - 优化搜索顺序(1)

    对于一个木棒而言，优先尝试使用长度较长的木棍进行拼接，这样子节点的状态会比较少

  - 排除等效冗余

    - 首先要限制加入木棍的顺序，对于一根木棒而言，加入3， 4， 5长度的三根木棍与加入4， 3， 5长度的三根木棍的效果是一样的，为了与(1)一致，可以直接对木棍数组按照从大到小的顺序进行排序

    - 如果尝试使用一根长度为`x`的木棍之后发现拼接失败，那么如果下一根木棍的长度仍然为`x`，那么肯定也是拼接失败

      因为两根长度相等的木棍其实是等价的

    - **如果在拼接过程中，尝试给木棒添加的第一根木棍时就失败了，那么可以直接回溯**

      证明：

      - 假设在放置第一根木棍时失败，但是接下来的搜索中发现这个解是合法的，也就是下面的情况

        ![image-20220715153744290](http://www.cdn.liver0377.xyz/typora/202207151537333.png)

        首部的木棍可以被放置到后面的某一根木棒的其他位置，执行如下操作：

        **将木棍移动到首部**

        ![](http://www.cdn.liver0377.xyz/typora/202207151540528.png)

        这个操作肯定是合法的，在一根木棒内部一定木棍时肯定可行的，这样就表明将该根木棍放在木棒首部是可行的，与原来相悖，得证

    - 如果在木棒中插入最后一根木棍之后，木棒恰好为给定的长度，但是在接下来的递归中却返回`false`,直接回溯

      证明：

      - 假设在插入最后一根木棍之后返回的递归中返回`false`, 但是该`length`是合法解

      - 那么原木棍的位置一定可以被被多根其它长度的木棍代替，原木棍可以放置在其它木棒的其它位置

        ![image-20220715154724078](http://www.cdn.liver0377.xyz/typora/202207151547129.png)

        此时很显然可以将原木棍交换回去，这样就得到了原状态，与假设矛盾



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 70;
int w[N];
int sum, length;   // sum: 所有木棍长度总和, length: 当前枚举的木棒长度(答案)
bool st[N];
int n;

// u: 已经拼接完成的木棒数量
// s: 正在拼接的木棒长度
// start: 正在使用的木棍下标
bool dfs(int u, int s, int start) {
    if (u * length == sum) return true;
    if (s == length) return dfs(u + 1, 0, 0);
    
    for (int i = start; i < n; i++) {
        if (st[i]) continue;
        if (s + w[i] > length) continue;
        
        st[i] = true;
        // 剪枝3： 每根木棒的木棍长度递减, 排除冗余
        if (dfs(u, s + w[i], start + 1)) return true;
        st[i] = false;
        
        // 剪枝4: 第一根木棍无解，后续全部无解
        if (s == 0) return false;
        // 剪枝5: 最后一根木棍无解, 后续全部无解
        if (s + w[i] == length) return false;
        // 剪枝6：再次放入相同长度的木棍肯定失败
        int j = i + 1;
        while (j < n && w[j] == w[i]) j ++;
        i = j - 1;
    }
    return false;
}

int main() {
    while (cin >> n, n) {
        memset(st, 0, sizeof st);
        int max_len = 0;
        sum = 0;
        for (int i = 0; i < n; i++) {
            cin >> w[i];
            sum += w[i];
            max_len = max(max_len, w[i]);
        }
        // 剪枝1: 优化搜索顺序
        sort(w, w + n, [&](int a, int b){
            return a > b;
        });
        
        // 木棒长度至少为最长的木棍长度
        for (length = max_len; length <= sum; length++) {
            // 剪枝2: 可行性剪枝
            if (!(sum % length) && dfs(0, 0, 0)) {
                cout << length << endl;
                break;
            }
        }
    }    
    return 0;
}

```







### 生日蛋糕

![image-20220715171124319](http://www.cdn.liver0377.xyz/typora/202207151711381.png)





**解题思路**

- `dfs`搜索每一层蛋糕，对于每一层而言，枚举`r`, `h`

- 需要维护的状态包括，**当前已经累加的面积，当前已经累加的体积，当前正在遍历的层**

- 剪枝

  - 优化搜索顺序

    从下向上递归蛋糕层数，因为下面的蛋糕的高度以及半径都要更大，子节点的状态数目更少

  - 上面界剪枝

    对于`h`, `r`来说，需要尽量缩小他们的范围

    - 设当前枚举的层为`u`,`v`为`u + 1 ~ m`层的体积, `s`为`u + 1 ~ m`层的面积

    - 由于半径以及高度的最小值为1，因此有

      - `v >= u`
      - ``r >= u` 

    - $n - v >= r^2 * h$

      => $r <= \sqrt{(n - v) / h} <= \sqrt{n - v}$

      => $h <= (n - v) / r^2 <= n - v$

    - 同时上面一层的半径以及高度均小于下面一层

      最终可以得到

      $u <= r <= min(\sqrt{n - v}, r[u + 1] - 1)$

      $u <= h <= min((n - v) / r^2, h[u + 1] - 1)$, 这里使用到了`r`,因此枚举时要先枚举`r`,再枚举`h`

  - 可行性剪枝

    **预处理**出从上向下前`i`(1 <= i <= m)的**最小**体积与侧面积，  当半径和高度均取`i`时，有最小值

    在遍历过程中，如果当前体积`v` 加上上面的最小体积超过`n`， 剪枝

  - 最优性剪枝

    如果当前表面积加上上面的最小表面积超过已知最优解，剪枝

    此外

    ![image-20220715172616451](http://www.cdn.liver0377.xyz/typora/202207151726505.png)



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

const int N = 25;

int n, m;
int R[N], H[N];
int minv[N], mins[N];
int ans = 0x3f3f3f3f;

void dfs(int u, int v, int s) {
    // 1. 可行性剪枝
    if (v + minv[u] > n) return ;
    // 2. 最优性剪枝
    if (s + mins[u] >= ans) return ;
    if (2 * (n - v) / R[u + 1] + s >= ans) return ;
    
    if (!u) {
        if (v == n) ans = s;
        return;
    }
    
    // 这里要从大到小枚举，同样是为了优化搜索顺序
    for(int r = min(R[u + 1] - 1,(int)sqrt((n - v - minv[u - 1]) / u)); r >= u; r--)
        for(int h = min(H[u + 1] - 1, (n - v - minv[u - 1]) / r / r); h >= u; h--)
        {
            H[u] = h, R[u] = r;

            //最底层的时候需要加上r*r
            int t = u == m ? r * r : 0;

            dfs(u - 1, v + r * r * h, s + 2 * r * h + t);
        }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        minv[i] = minv[i - 1] + i * i * i;
        mins[i] = mins[i - 1] + 2 * i * i;
    }
    
    R[m + 1] = 0x3f3f3f3f, H[m + 1] = 0x3f3f3f3f;
    dfs(m, 0, 0);
    
    if (ans == 0x3f3f3f3f) ans = 0;
    cout << ans << endl;
    
    return 0;
}
```

