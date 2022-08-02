- dfs双向搜索

  dfs双向搜索与bfs双向搜索类似，都是通过将一个高度为n的搜索树变为两颗高度为n / 2的搜索树，时间复杂度由O(2 ^ n)

  变为O(2 * 2^ (n/ 2))

  ![image-20220802173948898](http://www.cdn.liver0377.xyz/typora/202208021739955.png)

### 送礼物

![image-20220802174011140](http://www.cdn.liver0377.xyz/typora/202208021740192.png)

- 解题思路

  这其实是一个背包问题，如果使用普通的背包解法，那么时间复杂度为O(NV) = 46 * 2 ^31，会超时

  此题使用双向`dfs`来求解

- 首先将所有物品一分为二， 先对前面的一半物品进行组合，求出所有的体积不超过$W$的可能的质量组合

  将其记录在表`weights`中，这个过程使用的是普通的`dfs`

- 接下来对另一半物品进行组合，每求出一个不超过$W$的质量组合`s`，就尝试在表`weights`中找到不超过$W - s$

  的最大质量，然后更新全局最大解，这个过程是`bfs` + 二分搜索

- 搜索树

  ![image-20220802175753558](http://www.cdn.liver0377.xyz/typora/202208021757603.png)

- 剪枝

  - 优化搜索顺序

    对物品的质量进行降序排序，然后在进行分半操作，目的是使得表的体积能够小一点(减掉更多的枝)

  - 选出适当的折半划分点

    最终的时间复杂度应该是$2^{k} + 2^{N-k}*lg(2^k)$ = $2^k + 2^{N - k} * k$

    根据数学的话应该k = N/2 + 2是最快(然而过不了)

**代码实现**

```cc
#include <string>
#include <algorithm>
#include <iostream>
#include <set>
#include <functional>

using namespace std;

typedef long long LL;

const int N = 1 << 25;

int m, n, k;           // m: 最大承重量 n: 物品数目 k: 分界点
int weights[N];           // weights: 从前k件物品中进行组合，所有不超过m的重量的组合
int w[N];             // w[i]: 物品i重量
int ans;              // 最终能够拿的最大最大重量
int cnt;

// 从第0层遍历到第k - 1层
void dfs_1(int u, int s) {
    if (u == k) {
        weights[cnt ++] = s;
        return ;
    }
    
    // 不拿第u个物品    
    dfs_1(u + 1, s);
    // 拿第u个物品
    if ((LL)s + w[u] <= m)
        dfs_1(u + 1, s + w[u]);
}

// 从第k层遍历到第n层
void dfs_2(int u, int s) {
    if (u == n) {
        // 对于每一个从后面一半组合出的总质量s, 在前面一半算出的组合数中
        // 找到小于等于m - s且最大的值
        int l = 0, r = cnt - 1;
        while (l < r) {
            int mid = l + r + 1 >> 1;
            if (weights[mid] <= m - s) l = mid;
            else r = mid - 1;
        }
        ans = max(ans, weights[l] + s);
        return ;
    }
    
    // 不拿第u个物品
    dfs_2(u + 1, s);
    // 拿第u个物品
    if ((LL)s + w[u] <= m)
        dfs_2(u + 1, s + w[u]);
}

int main() {
    cin >> m >> n;
    for (int i = 0; i < n; i++) cin >> w[i];
    // 1. 优化搜索顺序: 对w进行降序排列
    sort(w, w + n, greater<int>());
    
    // 从前k件物品中进行组合
    
    k = n / 2;
    dfs_1(0, 0);
    
    sort(weights, weights + cnt);
    cnt = unique(weights, weights + cnt) - weights;
    
    
    dfs_2(k, 0);
    
    cout << ans << endl;
    return 0;
}
```

