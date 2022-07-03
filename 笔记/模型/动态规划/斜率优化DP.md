### 任务安排1

![image-20220702161819795](http://www.cdn.liver0377.xyz/typora/202207021618856.png)



**解题思路**

- 这题并没有使用到斜率优化，是一个样例题

- 状态表示

  ![image-20220702162339612](http://www.cdn.liver0377.xyz/typora/202207021623661.png)

- 状态计算

  设倒数第二批的最后一个位置为`j`, 则1 ~ i 的任务分配图如下

  ![image-20220702162852813](http://www.cdn.liver0377.xyz/typora/202207021628850.png)

  - 每个批次开始时会有一个启动时间`s`,这个`s`会让后面的所有任务的结束时间加上`s`
  - 在实际计算过程中，**提前将这个启动时间`s`对后面所有的任务的影响算上**

  $f[i] = min(f[j] + sumt[i] * (sumc[i] - sumc[j]) + s * (sumc[n] - sumc[j]))$ 

  - 因为这里启动时间的影响在之间的计算过程中已经算过了，因此最后一批的完成时间可以直接使用$sumt[i]$来表示

  

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 5e3 + 10;
typedef long long LL;

LL f[N], sumt[N], sumc[N];
int s, n;

int main() {
    cin >> n >> s;
    
    for (int i = 1; i <= n; i++) {
        int t, c;
        cin >> t >> c;
        sumt[i] = sumt[i - 1] + t;
        sumc[i] = sumc[i - 1] + c;
    }
    
    memset(f, 0x3f, sizeof f);
    f[0] = 0;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            f[i] = min(f[i], f[j] +  sumt[i] * (sumc[i] - sumc[j]) + s * (sumc[n] - sumc[j]));
        }
    }
    
    cout << f[n] << endl;
    
    return 0;
}
```

- 时间复杂度为:$O(n^2)$





### 任务安排2

![image-20220703150204395](http://www.cdn.liver0377.xyz/typora/202207031502464.png)



**解题思路**

- 这题只是和上一题的数据范围不一样而已，如果是$O(n^2)$的时间复杂度则过不了

- 任务安排1中给出了递推公式
  $f[i] = min(f[j] + sumt[i] * (sumc[i] - sumc[j]) + s * (sumc[n] - sumc[j]))$

  去掉$min$, 并且合并`j`的同类项可以得到

  $f[j] = (sumt[i] + s) * sumc[j] +f[i] - sumt[i] * sumc[i] - s * sumc[n]$

- 将$j$看做变量，$i$看做常量，令$y = f[j], x = sumc[j]$

  则

  $y = (sumt[i] + s) * x +f[i] - sumt[i] * sumc[i] - s * sumc[n]$

  对于任意一个$j$，都可以构造出一个点对(x, y), 做出x-y图像

  ![image-20220703150730159](http://www.cdn.liver0377.xyz/typora/202207031507207.png)

  上面的黑色点是所有可以取的(x, y)点集， 当$i$固定时，斜率就是固定的, 最终的目的是使得$f[i]$最小，效果就是截距最小

  通过平移直线可以直到，若想要使得直线的截距最小，那么该直接所经过的点集中的点一定在点集所构成的**下凸包**上，即图中所给出的黑色线上的点

- 对于一个斜率固定为$k_i$ (`i`固定)的直线而言，其对应的截距最小的点`j`一定满足

  $(f[j] - f[j - 1]) / (sumc[j] - sumc[j - 1]) <= k_i <= (f[j + 1] - f[j - 1]) / (sumc[j + 1] - sumc[j])$

  也就是**第一个斜率大于$k_i$的点**

  ![image-20220703151853078](http://www.cdn.liver0377.xyz/typora/202207031518120.png)

  

- 遍历所有点`i`,也就是逐个加入(x, y)对，维护一个单调队列，队列中的点构成一个凸包，并且满足队首的点就是第一个斜率大于$k_i$的点

- 由于$k_i = sumt[i] + s$, 是随着`i`单调递增的，因此后面的直线斜率只会越来越大，如果队列中存在点`x`的斜率小于当前点`i`，就应该尽快把它删掉，因为他不能是后面的`i + 1`, `i + 2`.. 等直线的最小截距点，所以使用单调队列来维护

  具体公式就是

  $(f[q[hh + 1]] - f[q[hh]]) / (c[q[hh] + 1] - c[q[hh]]) <= sumt[i] + s$

  

- 同时凸包的斜率肯定是单调递增的，这是其定义满足的要求，所以每次加入点`i`的时候都要判断一下是否会破坏凸包的性质

  如果会的话，就要持续删掉队尾的点，直到凸包性质恢复

  具体公式就是

  $(f[i] - f[q[tt]]) / (c[i] - c[q[tt]]) <= (f[q[tt]] - f[q[tt - 1]]) / (c[q[tt]] - c[q[tt - 1]])$

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 3e5 + 10;
typedef long long LL;

LL f[N], c[N], t[N], q[N];
int n, s;

int main() {
    cin >> n >> s;
    
    for (int i = 1; i <= n; i++) {
        int a, b;
        cin >> a >> b;
        t[i] = t[i - 1] + a;
        c[i] = c[i - 1] + b;
     }
     
     int hh = 0, tt = 0;
     q[0] = 0;
     for (int i = 1; i <= n; i++) {
         // hh < tt 表明队列中至少要两个元素
         while (hh < tt && f[q[hh + 1]] - f[q[hh]] <= (c[q[hh + 1]] - c[q[hh]]) * (t[i] + s)) hh ++;
         int j = q[hh];
         f[i] = f[j] + t[i] * (c[i] - c[j]) + s * (c[n]  - c[j]);
         while (hh < tt && (f[q[tt]] - f[q[tt - 1]]) * (c[i] - c[q[tt]]) >= 
                           (f[i] - f[q[tt]]) * (c[q[tt]] - c[q[tt - 1]])) tt --;
         q[++ tt] = i;
     }
     
     cout << f[n] << endl;
}
```

- 为了表明除法的取整问题，统一将除法改为了乘法