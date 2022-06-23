### 环型石子合并

![image-20220620164542366](http://www.cdn.liver0377.xyz/typora/202206201646812.png)



**解题思路**

- 将n个石子围成一圈，每次合并两堆石子就是在这两堆石子之间加一条线
  ![image-20220620164842162](http://www.cdn.liver0377.xyz/typora/202206201648211.png)

- 由于n堆石子只会合并n - 1次，因此最终形成n - 1条边，即产生一个**缺口**

- 朴素思想是穷举每一个缺口，这样就构成了n个链式石子合并问题了，但是这样做时间复杂度会变成$O(n^4)$

- 因此，这里使用了一个Trick

  - 构造一个长度为2^n的链，即将n个石子复制一份
    ![image-20220620165228646](http://www.cdn.liver0377.xyz/typora/202206201652708.png)

    这样的话，之前穷举缺口的过程就变成了求n遍长度为n的子链问题，时间复杂度为$O((2n)^3)$

**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 210 * 2;
int s[N];
int a[N];
int f1[N][N];
int f2[N][N];
int n;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i + n] = a[i];
    }
    for (int i = 1; i <= 2 * n; i++) {
        s[i] = s[i - 1] + a[i];
    } 
    
    for (int len = 2; len <= 2 * n; len ++) {
        for (int i = 1; i + len - 1 <= 2 * n; i ++) {
            int j = i + len - 1;
            f1[i][j] = 1e8;
            f2[i][j] = 0;
            for (int k = i; k < j; k++) {
                f1[i][j] = min(f1[i][j], f1[i][k] + f1[k + 1][j] + s[j] - s[i - 1]);
                f2[i][j] = max(f2[i][j], f2[i][k] + f2[k + 1][j] + s[j] - s[i - 1]);
            }
        }
    }
    
    int m1 = 1e8, m2 = 0;
    for (int i = 1; i <= n; i++) {
        m1 = min(m1, f1[i][i + n - 1]);
        m2 = max(m2, f2[i][i + n - 1]);
    }
    
    cout << m1 << endl;
    cout << m2 << endl;
    
    return 0;
}
```

> 对于区间DP问题，通常使用两种格式进行求解
>
> - 迭代式
>
>   迭代式的格式通常如下
>   ```cc
>   for (int len = 2; len <= n; len ++) {            // 循环长度
>       for (int i = 1; i + len -1 <= n; i ++) {     // 循环起点
>           int j = i + len - 1;
>           ...
>       }
>   }
>   ```
>
>   
>
> - 记忆化搜索



### 能量项链

![image-20220620173646473](http://www.cdn.liver0377.xyz/typora/202206201737055.png)



**解题思路**

- 这里和上一题的思路十分相似，同样是合并，同样是环型问题，可以采用相同的套路来做
- 首先考虑链式合并，假设给定的序列是2, 3, 5, 10
- 这个序列表明有四个珠子(2, 3), (3, 5), (5, 10), (10, 2)
- 考虑将最后两个珠子合并，也就是(5, 10), (10, 2)合并，发现这个2在序列的首部，难以一般化的写出合并公式
- 因此，将首部的2复制到尾部，形成一个新的序列2, 3, 5, 10, 2
- 接下来，每次合并只需要选择一个中间点k, 就可以将这些序列代表的珠子合并成两个珠子
- 例如，选择中间节点3(下标为2, 下标从1开始)
- 最终形成的两个珠子应该为(2, 3), (3, 2)



- 状态表示

  $f[i][j]$表示对给定的序列的位置i合并到位置j能够释放的最大能量

- 状态计算
  $f[i][j] = max{f[i][j], f[i][k1] + f[k1][j] + a[i] * a[k1] * a[j], ...}$



- 环型问题的解决
  环型问题与上一题类似，同样是采用一个长度为2n的链, 如2, 3, 5, 10, 2, 3, 5, 10

  只需要枚举所有长度为n + 1的链的合并情况，就能够得到最大释放能量



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 220;

typedef long long LL;

int n;
LL f[N][N];
LL a[N];

int main() {
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i + n] = a[i];
    }
    
    for (int len = 3; len <= 2 * n; len ++) {
        for (int i = 1; i + len - 1 <= 2 * n; i ++) {
            int j = i + len - 1;
            // f[i][j]默认为0
            for (int k = i + 1; k < j; k++) {   // 长度为1的链的合并是不合法的，因此k应该从i + 1开始
                f[i][j] = max(f[i][j] , f[i][k] + f[k][j] + a[i] * a[k] * a[j]);
            }
        }
    }
    
    LL res = 0;
    for (int i = 1; i <= n; i++) {
        res = max(res, f[i][i + n]);
    }
    
    cout << res << endl;
    return 0;
}
```



### 凸多边形的划分

![image-20220620194630892](http://www.cdn.liver0377.xyz/typora/202206201946951.png)



解题思路

- 这题的题目背景虽然和之前两题很不一样，但其实做法完全一样

- 如果将一个凸多边形的问题划分成多个子问题？

  考虑下面这种划分方式

  ![image-20220620195009980](http://www.cdn.liver0377.xyz/typora/202206201950021.png)

- 假设以1,6这两个顶点为起始边，那么可以在2, 3，4, 5任意选择一个顶点，构成一个三角形:triangular_ruler:(l, k , r)

- 这样，整个多边形就被分为了左右两个(一个)多边形+中间一个三角形，左右两个(一个)多边形就构成了两个(一个)子问题

  > 当另一个顶点选择与L,R相邻的顶点时，只会产生一个子多边形

- 如果构建序列，使得该问题的分析方式与之前两题类似?

  按照多边形的顶点顺时针构建一个序列，这里就是1, 2, 3, 4, 5, 6, 这里l = 1, r = 6

  - 状态表示
    $f[i][j]$表示有i, i + 1, ...,j 这些顶点所构成的多边形进行划分之后，三角形乘积权值之积最小的方案的权值

  - 任意选择2, 3, 4, 5中的一个顶点k, 与l, r构成中间三角形

  - 状态计算
    $f[i]j] = min(f[i][j], f[i][k1] + f[k1][j] + w[i] * w[k1] * w[j], ...)$

    该状态方程与之前的能量项链完全相同，因此可以采用完全相同的做法进行求解

- 这里为什么没有使用环？

  无论以哪两个相邻的顶点为起始边，最终都会穷举出所有的三角形划分方案

  所以只需要以任意一个边开始就行了

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 55;
typedef long long LL;

vector<int> f[N][N];
LL w[N];
int n;


bool cmp(vector<int>& a, vector<int>& b) {
    if (a.size() != b.size()) 
        return a.size() > b.size();
        
    for (int i = a.size() - 1; i >= 0; i--) {
        if (a[i] != b[i]) {
            return a[i] > b[i];
        }
    }
    
    return true;
}


vector<int> add(vector<int> a, vector<int> b) {
    int t = 0;
    vector<int> ans;
    for (int i = 0; i < a.size() || i < b.size(); i++) {
        if (i < a.size()) t += a[i];
        if (i < b.size()) t += b[i];
        ans.push_back(t % 10);
        t /= 10;
    }
    if (t) ans.push_back(t);
    return ans;
}

vector<int> multi(vector<int> a, LL b) {
    LL t = 0;  // 必须是LL， 否则会爆int
    vector<int> ans;
    for (int i = 0; i < a.size() || t; i++) {
        if (i < a.size()) t += a[i] * b;
        ans.push_back(t % 10);
        t /= 10;
    }
    
    while (ans.size() > 1 && ans.back() == 0) ans.pop_back();
    return ans;
}

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> w[i];
    
    for (int len = 3; len <= n; len ++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
    
            for (int k = i + 1; k < j; k++) {
                // 下面两行等价于:
                // add_sum = f[i][k] + f[k][j] + w[i] * w[k] * w[j]
                auto multi_product = multi(multi({w[i]}, w[k]), w[j]);
                vector<int> add_sum = add(add(multi_product, f[k][j]), f[i][k]);
                // f[i][j]默认为空，因此不能够直接比较大小进行判断，这里直接特判空
                if (f[i][j].empty() || cmp(f[i][j], add_sum)) {
                    f[i][j] = add_sum;
                }
            }
        }
    }
    
    // 记得倒过来输出
    for (int i = f[1][n].size() - 1; i >= 0; i--) {
        cout << f[1][n][i];
    }
    puts("");
    return 0;
}
```

- 由于每个点的权值为10^9, 因此三个点相乘之后会爆long long, 因此必须使用高精度



### 加分二叉树

![image-20220621152531263](http://www.cdn.liver0377.xyz/typora/202206211525325.png)



**解题思路**

- 此题看似是一个树的问题，其实是一个区间dp问题

- 已知二叉树的中序遍历的节点序列为1, 2, 3, ..., n

- 设其根节点为k, 则

  - 其左子树的中序遍历序列为1, 2, 3, .., k
  - 其右子树的中序遍历序列为k + 1, k + 2, ..., n

- 根据左右子树是否为空，可以判断出整颗树的加值满足下列关系
  设w为根节点权值，A为左子树的加值，B为右子树的加值

  - 左右子树均为空：w
  - 左子树为空: w + B
  - 右子树为空: w + A
  - 左右子树均不为空: w + A * B

- 若想要整颗树的加值最大，就要满足左右两颗子树的加值分别最大，而左右两棵子树的加值求解就是两个子问题的求解，

  它们完全独立

- 状态表示
  ![image-20220621153154232](http://www.cdn.liver0377.xyz/typora/202206211531323.png)

  



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 35;

int f[N][N]; // f[i][j]: 表示从第i个点到第j个点构成的子树的加值
int g[N][N]; // 在范围i ~ j内，取得最大价值的方案的根节点
int w[N];    // w[i]: 点i的权重
int n;

void dfs(int l, int r) {
    if (l > r) return;
    int root = g[l][r];
    cout << root << ' ';
    dfs(l, root - 1), dfs(root + 1, r);
}

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> w[i];
    
    for (int len = 1; len <= n; len++) {   // 这里len从1开始，是因为f[i][i], g[i][i]都需要初始化
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            if (len == 1) {     // 仅在左右子树均为空的情况下需要特判, 加值为w
                f[i][j] = w[i];
                g[i][j] = i;
            } else {
                for (int k = i; k <= j; k++) {
                    int left = (k == i) ? 1 : f[i][k - 1];   // 左子树的加值
                    int right = (k == j) ? 1 : f[k + 1][j];  // 右子树的加值
                    int score = left * right + w[k];
                    if (score > f[i][j]) {
                        f[i][j] = score;
                        g[i][j] = k;
                    }
                }
            }
        }
    }
    
    cout << f[1][n] << endl;
    dfs(1, n);
    return 0;
}
```



### 棋盘分割

![image-20220621170555477](http://www.cdn.liver0377.xyz/typora/202206211705542.png)



**解题思路**

- 本题的目标是将一个8 * 8的矩阵切割成n个块，共切割n - 1刀，得到每一块的总权值x1, x2..., xn

- 使得
  ![image-20220621170821144](http://www.cdn.liver0377.xyz/typora/202206211708189.png)

  最小, 这里对上述公式进行转化, 要使得$\sigma$最小，**只需要对于给定的n, 使得$\Sigma_{i = 1}^n(x_i - \bar{x})$最小即可**

**状态表示**

![image-20220621171530373](http://www.cdn.liver0377.xyz/typora/202206211715439.png)

- 注意，每一对一个矩阵进行切割，有横切和竖切两种方式，并且切一刀得到两块矩阵之后，要选择一块继续进行切割
- 因此，只需要穷举所有的切割方案即可, 这里使用递归 + 记忆化搜索来做, 属于二维区间dp问题



**状态计算**

直接看代码

**代码实现**

```cpp
#include <iostream>
#include <cmath>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 16;
const int M = 9;

double f[N][M][M][M][M];  // f[k][x1][y1][x2][y2]: 对(x1, y1, x2, y2)构成的矩阵，
                          // 切下n - k刀后，得到的n - k + 1个矩阵的(xi - X)^2最小和
int s[M][M];              // 二维前缀和
double X;  // X拔
int n;

// 返回矩形对应的(xi - X)^2
double get(int x1, int y1, int x2, int y2) {
    double t = s[x2][y2] - s[x2][y1 - 1] - s[x1 - 1][y2] + s[x1 - 1][y1 - 1];
    t -= X;
    return t * t;
}

double dp(int k, int x1, int y1, int x2, int y2) {
    // 记忆化搜索
    if (f[k][x1][y1][x2][y2] >= 0) return f[k][x1][y1][x2][y2];
    if (k == n) {
        // 已经切完了最后一刀, 即对该矩阵不用操作了，直接返回其对应的(xi - x)^2
        return f[k][x1][y1][x2][y2] = get(x1, y1, x2, y2);
    }
    
    // 1. 横着切
    double t = 0x3f3f3f3f;
    for (int i = x1; i < x2; i++) {
        // 1.1 选择上面一块
        t = min(t, dp(k + 1, x1, y1, i, y2) + get(i + 1, y1, x2, y2));
        // 1.2 选择下面一块
        t = min(t, dp(k + 1, i + 1, y1, x2, y2) + get(x1, y1, i, y2));
    }
    
    // 2. 竖着切
    for (int i = y1; i < y2; i++) {
        // 2.1 选择左边一块
        t = min(t, dp(k + 1, x1, y1, x2, i) + get(x1, i + 1, x2, y2));
        // 2.2 选择右边一块
        t = min(t, dp(k + 1, x1 ,i + 1, x2, y2) + get(x1, y1, x2, i));
    }
    // t是最小的一种方案
    return f[k][x1][y1][x2][y2] = t;
}

int main() {
    cin >> n;
    // 1.获取二维前缀和
    for (int x = 1; x <= 8; x++) {
        for (int y = 1; y <= 8; y++) {
            cin >> s[x][y];
            s[x][y] = s[x][y] + s[x][y - 1] + s[x - 1][y] - s[x - 1][y - 1];
        }
    }
    
    // 2.初始化
    memset(f, -1, sizeof f);  // 对于double, 将每个字节初始化为-1即为负无穷
    
    // 3. dp
    X = (double) s[8][8] / n;
    printf("%.3lf\n", sqrt(dp(1, 1, 1, 8, 8) / n));
    return 0;
}
```



