

- IDA*算法

  IDA*其实就是启发式搜索 + 迭代加深

  IDA*加强了迭代加深的**深度限制**：

  - 当遍历到一个深度时，若当前深度 + 启发函数所计算出的还需要的深度 > 给定深度限制

    那么就可以直接剪枝

  - 这里的启发函数同样需要保证**推测值<=实际值**









### 排书

![image-20220802193104208](http://www.cdn.liver0377.xyz/typora/202208021931267.png)



- 首先分析以下分支数目

  设`i`为抽取的数的数目，`n`为书的总数

  对于任何一个状态(书的任何一种排列), 可以选择`n - i + 1`个起始位置, `n - i`个放置位置

  > 当抽出书之后，应该是由n - i + 1个空档的，但是由于放回到原来抽出来的位置没有意义，因此只需要计算n - i个放置位置即可

  也就是从一个状态转移到另一种状态，总共有$\Sigma_{i = 1}^{n}(n - i + 1) * (n - i)$ = 560 * 2

  注意，上面的求法是有重复的，如下图

  ![image-20220802193611435](http://www.cdn.liver0377.xyz/typora/202208021936482.png)

  该变换既可以看做是将**红色**区域移动到绿色区域后，也可以看做是将**绿色**区域移动到红色区域前面，也就是这两种移动方法的效果是一样的，在代码中，可以只将书向后移动，来避免重复，因此，从一个状态，可以转移到最多560个其它状态

  由于最多只需要遍历4层，因此总的时间复杂度是O(560^4)

- 可以采用IDA*算法进行优化

- 启发函数的定义

  - 正确后继： 若序号为`i`的书后面紧跟着序号为`i + 1`的书，那么就称这两本书构成正确后继关系
  
  - 进行一次移动操作(状态转换)最多可以改变三本书的后继
  
    ![image-20220802183038779](http://www.cdn.liver0377.xyz/typora/202208021942958.png)
  
  - 因此只需要计算出当前状态下，总共的非正确后继关系的数目`tot`，接下来的移动次数不可能会低于$\lfloor \frac{tot}{3} \rfloor$
  - 可以使用这个来作为启发函数







**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 17;

int w[5][N];  // w[i]: 第i层的状态, 最多只遍历5层
int q[N];     // q[i]: 表示当前状态
int n;

// 启发函数，推测从当前状态q走到最终状态需要的步数
int f() {
    int tot = 0;
    for (int i = 1; i < n; i++){
        if (q[i] != q[i - 1] + 1) tot ++;
    }
    return (tot + 2) / 3;
}

bool dfs(int u, int max_depth) {
    if (u + f() > max_depth) return false;
    if (f() == 0) return true;    // 走到最终状态
    
    for (int len = 1; len <= n; len ++) {
        for (int l = 0; l + len <= n; l ++) {
            int r = l + len - 1;
            for (int k = r + 1; k < n; k ++) {
                 memcpy(w[u], q, sizeof q);  // 在w[u]存储当前状态
                int x = r + 1, y = l;
                // 状态变换
                for (; x <= k; x ++, y++) q[y] = w[u][x];
                for (x = l; x <= r; x ++, y++) q[y] = w[u][x];
                if (dfs(u + 1, max_depth)) return true;
                // 恢复到原来u层的状态
                memcpy(q, w[u], sizeof q);
            }
        }
    }
    // 从当前层u，走遍所有的分支都无法在给定限度内到达最终状态
    return false;
}

int main() {
    int T;
    cin >> T;
    while (cin >> n) {
        for (int i = 0; i < n; i++) cin >> q[i];
        int max_depth = 0;  // 从0开始，因为可能一次操作都不用做
        while (max_depth < 5 && !dfs(0, max_depth)) max_depth ++;
        if (max_depth >= 5) {
            cout << "5 or more" << endl;
        } else {
            cout << max_depth << endl;
        }
    }
    return 0;
}
```

- 33, 34行的图示

  ![image-20220802194826961](http://www.cdn.liver0377.xyz/typora/202208021948018.png)



### 



![image-20220803144837106](http://www.cdn.liver0377.xyz/typora/202208031448168.png)

### 回转游戏

![image-20220803152506192](http://www.cdn.liver0377.xyz/typora/202208031525255.png)

![image-20220803152517267](http://www.cdn.liver0377.xyz/typora/202208031525320.png)



**解题思路**

- 同样是采用IDA*算法

- 估价函数

  统计出中间8个格子中，出现次数最多的数字的出现次数`s`，由于每次移动操作，会将一个数字移除，一个数字移入，因此，每次最

  多可以移入一个我们想要的数字，最少也需要经过8 - `s`次操作才能够使得中间8个格子的数字完全一样

- 剪枝

  由于这里的操作是两两相逆的，因此可以在遍历过程中，可以记录下上一次操作，这一次不进行上一次操作的逆操作，因为这样做是

  没有意义的

- 字典序问题

  题干中要求输出最小字典序，因此遍历是按字典序从小到大的方式进行迭代，这样可以保证最先输出的一定是字典序最小的





**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 24;

int operations[8][7] = {
    {0, 2, 6, 11, 15, 20, 22},      // A
    {1, 3, 8, 12, 17, 21, 23},      // B
    {10, 9, 8, 7, 6, 5, 4},         // C
    {19, 18, 17, 16, 15, 14, 13},   // D
    {23, 21, 17, 12, 8, 3, 1},      // E
    {22, 20, 15, 11, 6, 2, 0},      // F
    {13, 14, 15, 16, 17, 18, 19},   // G
    {4, 5, 6, 7, 8, 9, 10}          // H
};                                                           // 每个操作对应的格子
int opposite[8] = { 5, 4, 7, 6, 1, 0, 3, 2};                 // 每个操作的逆操作
int center[8] = {6, 7, 8, 12, 17, 16, 15, 11};               // 中间的8个格子
int q[N];    // 存储整个#型棋盘
int path[100]; // 开足够大的空间

int f() {
    int sum[4] = {0};
    for (int i = 0; i < 8; i ++ ) sum[q[center[i]]] ++ ;

    int s = 0;
    for (int i = 1; i <= 3; i ++ ) s = max(s, sum[i]);
    return 8 - s;
}

void operate(int u) {
    int h = q[operations[u][0]];
    for (int i = 0; i < 6; i++) {
        q[operations[u][i]] = q[operations[u][i + 1]];
    }
    q[operations[u][6]] = h;
}

// last: 上一次操作
bool dfs(int u, int max_depth, int last) {
    if (f() + u > max_depth) return false;
    if (!f()) return true;
    
    // 按照字典序遍历8个操作，可以确保最终答案的字典序最小
    for (int i = 0; i < 8; i++) {
        // 剪枝: 不执行上一次操作的逆操作
        if (opposite[i] == last) continue;
        path[u] = i;
        operate(i);
        if (dfs(u + 1, max_depth, i)) return true;
        operate(opposite[i]);  // 恢复现场
    }
    
    return false;
}
int main() {
    while (cin >> q[0], q[0]) {
        for (int i = 1; i <= 23; i ++) cin >> q[i];
        int max_depth = 0;
        while (!dfs(0, max_depth, -1)) max_depth ++;
        if (!max_depth) {
            cout << "No moves needed";
        } else {
            for (int i = 0; i < max_depth; i++) {
                cout << static_cast<char>(path[i] + 'A');
            }
        }
        cout << endl << q[6] << endl;
    }
    return 0;
}
```

