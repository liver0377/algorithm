

### Hello leetcode

![image-20221008201817307](http://www.cdn.liver0377.xyz/typora/202210082018374.png)



**解题思路**

![image-20221008201903975](http://www.cdn.liver0377.xyz/typora/202210082019019.png)

**代码实现**

```cc
class Solution {
    static const int MAXN = 24;
    static const int MAXM = 8;
    const long long INF = 1e12;

    // 出现 1 次：c, d, h, t
    // 出现 2 次：o
    // 出现 3 次：l
    // 出现 4 次：e
    static const int SIGMA = 7;
    int times[SIGMA] = {2, 2, 2, 2, 3, 4, 5};
    int mp[26] = {
        -1, -1, 0, 1, 6, -1, -1, 2, -1, -1, -1, 5, -1,
        -1, 4, -1, -1, -1, -1, 3, -1, -1, -1, -1, -1, -1
    };
    static const int MAXA = 2 * 2 * 2 * 2 * 3 * 4 * 5;

    // 把二进制 mask 转为每种字符已经拿取的数量
    void msk2Cnt(int msk, int *cnt) {
        for (int i = 0, j = 1; i < SIGMA; i++) {
            cnt[i] = msk / j % times[i];
            j *= times[i];
        }
    }

    // 把每种字符已经拿取的数量转为二进制 mask
    int cnt2Msk(int *cnt) {
        int ret = 0;
        for (int i = 0, j = 1; i < SIGMA; i++) {
            ret += cnt[i] * j;
            j *= times[i];
        }
        return ret;
    }

public:
    int Leetcode(vector<string>& words) {
        int n = words.size();

        // 预处理 g[i][mask] 表示从长度为 i 的单词里拿走 mask 代表的字符需要的最少代价
        static long long g[MAXM + 1][1 << MAXM];
        static int gInited = false;
        if (!gInited) {
            for (int i = 1; i <= MAXM; i++) {
                for (int j = 0; j < (1 << i); j++) g[i][j] = INF;
                g[i][0] = 0;
            }
            for (int i = 1; i <= MAXM; i++) 
                for (int j = 0; j < (1 << i); j++) {
                // 枚举拿走哪个字符
                    for (int k = 0; k < i; k++) if (j >> k & 1) {
                    int jj = j ^ (1 << k);
                    // 统计该字符左边和右边的字符数
                    int L = 0, R = 0;
                    for (int pos = 0; pos < k; pos++) if (jj >> pos & 1 ^ 1) L++;
                    for (int pos = k + 1; pos < i; pos++) if (jj >> pos & 1 ^ 1) R++;
                    // 更新答案
                    g[i][j] = min(g[i][j], g[i][jj] + L * R);
                    }
            }
            gInited = true;
        }

        static long long f[MAXN + 1][MAXA];
        for (int i = 0; i <= n; i++) for (int j = 0; j < MAXA; j++) f[i][j] = INF;
        f[0][0] = 0;

        for (int i = 1; i <= n; i++) {
            int m = words[i - 1].size();
            // 枚举从第 i 个单词里拿走哪些字符
            // 注意一定要在外层枚举 msk，内层枚举 j
            // 否则无法跳过大量非法状态，容易被力扣卡总用时，垃圾题目
            for (int msk = 0; msk < (1 << m); msk++) {
                // cnt 表示拿走的字符中，cdhtole 各出现几次
                int cnt[SIGMA] = {0};
                bool failed = false;
                for (int pos = 0; pos < m; pos++) if (msk >> pos & 1) {
                    int x = words[i - 1][pos] - 'a';
                    if (mp[x] == -1) { failed = true; break; }
                    x = mp[x];
                    cnt[x]++;
                }
                if (failed) continue;

                // 枚举拿走字符后，现在已有的字符情况
                for (int j = 0; j < MAXA; j++) {
                    int oldCnt[SIGMA];
                    msk2Cnt(j, oldCnt);
                    bool failed = false;
                    for (int i = 0; i < SIGMA; i++) {
                        oldCnt[i] -= cnt[i];
                        if (oldCnt[i] < 0) { failed = true; break; }
                    }
                    if (failed) continue;
                    int jj = cnt2Msk(oldCnt);
                    // 更新答案
                    f[i][j] = min(f[i][j], f[i - 1][jj] + g[m][msk]);
                }
            }
        }

        long long ans = f[n][MAXA - 1];
        return ans >= INF ? -1 : ans;
    }
};

```

