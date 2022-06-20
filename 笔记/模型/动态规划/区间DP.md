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
