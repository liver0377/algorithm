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