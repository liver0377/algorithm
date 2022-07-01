### 滑动窗口

![image-20220701143353903](http://www.cdn.liver0377.xyz/typora/202207011446506.png)

**代码实现**

```cc
#include <iostream>

using namespace std;

const int N = 1000010;
int q[N];
int a[N];

int tt = -1, hh = 0;

int main() {
    int n, k;
    cin >> n >> k;
    
    for (int i = 0; i < n; i++) cin >> a[i];    
    for (int i = 0; i < n; i++) {
        
        while (hh <= tt && (i - q[hh] + 1 > k)) hh ++;
        while (hh <= tt && (a[q[tt]] >= a[i])) tt --;
        q[++ tt] = i;
        if (i >= k - 1) cout << a[q[hh]] << " ";
    }
    cout << endl;
    
    hh = 0, tt = -1;
    for (int i = 0; i < n; i++) {
        while (hh <= tt && (i - q[hh] + 1 > k)) hh ++;
        while (hh <= tt && (a[q[tt]] <= a[i])) tt --;
        q[++ tt] = i;
        if (i >= k - 1) cout << a[q[hh]] << " ";
    }
    return 0;
}
```





### 最大子序和

![image-20220629161402322](http://www.cdn.liver0377.xyz/typora/image-20220629161402322.png)

**解题思路**

- 注意这里的序列是指数组，元素必须连续

- 状态表示

  - 集合: $f[i]$表示在以$a[i]$结尾的，长度不超过$m$的所有子序列
  - 属性: 子序列中，所有元素之和最大的序列的元素和

- 状态计算

  这里使用前缀和来计算$f[i]$

  - $s[i]$表示以`a[i]`结尾，以`a[1]`开头的序列的前缀和
  - $f[i] = max(s[i] - s[j]) = s[i] - min(s[j])$, 其中$i - m <= j <= i - 1$

- 将`i`固定，需要穷举上面范围内所有的`j`，计算出最小的$s[j]$, 将所有的$s[j]$构成一个序列$s[i - m], s[i - m + 1]..., s[i - 1]$ ，序列长度为`m`

  那么这题就可以转化为一个**滑动窗口**问题：

  - 在一个长度为`m `的窗口内部，求出最小的$s[j]$
  - 使用单调队列实现即可

**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;
typedef long long LL;

const int N = 3e5 + 10;
LL s[N];
LL a[N];
LL q[N];



int main() {
    int n, m;
    cin >> n >> m;
    
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 1; i <= n; i++) s[i] = s[i - 1] + a[i];
    
    LL res = -1e18;
    int hh = 0, tt = 0; // 队列一开始存了一个0，因为i == 0时，前面s[j]的最小值为0, 因此tt从0开始
    for (int i = 1; i <= n; i++) {
        while (hh <= tt && i - 1 - q[hh] + 1 > m) hh ++;
        res = max(res, s[i] - s[q[hh]]);   // 对于每一个i, s[q[hh]]即为i - m ~ i - 1窗口内部的最小值
        while (hh <= tt && s[i] <= s[q[tt]]) tt --;
        q[++ tt] = i;
    }
    
    cout << res << endl;
    
    return 0;
}
```





### 旅行问题

![image-20220629173551910](http://www.cdn.liver0377.xyz/typora/image-20220629173551910.png)

![image-20220629173601354](http://www.cdn.liver0377.xyz/typora/image-20220629173601354.png)



**解题思路**

**顺时针**

- 破环成链，使用一个长度为2 * n的链来代替原来的环

- 对于每个加油站，算出其每个点对应的添油量 - 耗油量`x`, 计算出x的前缀和`s`

- 若在区间$[i, i + n - 1]$内可以环游一周，那么就需要满足对于$j \in [i, i + n - 1]$, 均有$s[j] - s[i - 1] >= 0$

  即从点`i`到`i + n - 1`的任意一个点`j`，其前缀和`s`不小于0

- 即$min(s[j] - s[i - 1]) >= 0$

  => $min(s[j]) - s[i - 1] >= 0$

- 现在只需要求出$i <= j <= i + n - 1$区间内部的$min(s[j])$即可

- 可以套用滑动窗口的模板, 使用递增单调队列, 队列首部为`min`

**逆时针**

- $s[i]$变为从点$2n$逆时针到点$i$的添油量 - 耗油量的前缀和

  $s[i] = s[i + 1] + p[i] - d[i - 1]$

- 若在区间$[i, i - n +1]$内可以逆时针环游一周，那么需要满足，对于$j \in [i - n + 1, i]$, 均有$s[j] - s[i + 1] >= 0$

- 即$min(s[j] - s[i + 1]) >= 0$

  => $min(s[j]) - s[i + 1] >= 0$

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <cstdio>

using namespace std;

typedef long long LL;

const int N = 2000010;

int n;
int p[N], d[N];  // p[i]: 表示点i的添油量
                 // d[i]: 表示从点i到点i + 1的耗油量
LL s[N];     // 顺时针时表示p[i]-d[i]的前缀和
             // 逆时针: s[i] = p[i] - d[i - 1] + 
             //               p[i + 1] - d[i] + ...
             //               p[2 * n] - d[2 * n - 1] 
             // p[i] - d[i - 1]的意思是点i的添油量 - 从点i 到点i - 1的耗油量
int q[N];    // 单调队列维护长度为n的区间中s[i]的最小值
bool st[N];  // st[i]为true表示从i开始有解

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) scanf("%d%d", &p[i], &d[i]);

    // 顺时针时求p[i]-d[i]的前缀和
    for (int i = 1; i <= n; i++) s[i] = s[i + n] = p[i] - d[i];
    for (int i = 1; i <= n * 2; i++) s[i] += s[i - 1];
    // 从大到小枚举i，然后考虑区间[i,i+n-1]中的最小值是否>=s[i-1]
    int hh = 0, tt = -1;
    for (int i = n * 2; i; i--) { // 区间在i的右边，需要从右向左枚举
        while (hh <= tt && q[hh] > i + n - 1) hh++; // 判断是否滑出[i,i+n-1]的窗口
        while (hh <= tt && s[q[tt]] >= s[i]) tt--;
        q[++tt] = i;
        // 此时队头元素就是区间min
        if (i <= n && s[q[hh]] >= s[i - 1]) st[i] = true; // 表示以i起起始顺时针走有解
    } 

    // 逆时针时求p[i]-d[i-1]的后缀和
    d[0] = d[n]; // p[1]-d[0]时需要用到
    for (int i = n; i; i--) s[i] = s[i + n] = p[i] - d[i - 1];
    for (int i = n * 2; i; i--) s[i] += s[i + 1];
    // 从小到大枚举i，然后考虑区间[i-n+1,i]中的最小值是否>=s[i+1]
    hh = 0, tt = -1;
    for (int i = 1; i <= n * 2; i++) {       // 区间在i的左边, 需要从左向右枚举
        while (hh <= tt && q[hh] < i - n + 1) hh++;
        while (hh <= tt && s[q[tt]] >= s[i]) tt--;
        q[++tt] = i;
        if (i > n && s[q[hh]] >= s[i + 1]) st[i - n] = true; // 注意这里起始位置是i-n+1的前一个位置
    }
    for (int i = 1; i <= n; i++) puts(st[i] ? "TAK" : "NIE");

    return 0;
}

```



### 烽火传递

![image-20220701171043053](http://www.cdn.liver0377.xyz/typora/202207011710115.png)



**解题思路**

- 首先从DP角度思考这个问题

- 状态表示

  - $f[i]$表示从烽火台`1`到烽火台`i`, 不包含连续`m`个未点燃的烽火台，并且点燃第`i`个烽火台的方案
  - 属性：所有方案中的最小代价

- 状态转移

  - 设倒数第二个点燃的烽火台的位置为`j`, 则$i - m <= j <= i - 1$

  - 因此

    $f[i] = min(f[i - 1], f[i - 2], ..., f[i - m]) + w[i]$

- 对于任意一个`f[i]`，都要求前$m$个$f[k]$, 因此可以转换成一个滑动窗口问题



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 2 * 1e5 + 10;

int f[N];
int q[N];
int n, m;
int a[N];

int main(){
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> a[i];
    
    int hh = 0, tt = m - 1; // 可以将第一个烽火台前面看做是已经有了m - 1个代价为0的烽火台
    
    for (int i = 1; i <= n + 1; i++) {
        while (hh <= tt && q[hh] < i - m) hh ++;
        f[i] = a[i] + f[q[hh]];
        while (hh <= tt && f[i] <= f[q[tt]]) tt --;
        q[++ tt] = i;
    }
    
    
    
    cout << f[q[hh]] << endl;
    return 0;
}
```

- 这里使用了一个**Trick**
- 最后一个点燃的烽火台可能的下标`i`满足$n - m + 1<= i <= n$，即最后的答案位于`i`在上述范围内的$f[i]$中最小的一个
- 因此可以在求单调队列时，多求一次，将第`n + 1`个位置看成是一个代价为0的烽火台
- 这样，当最后一次单调队列求完之后，滑动窗口的范围是$[n - m + 1, n]$, 正好为所需要的范围，队列头部即为最小的$f[i]$





### 绿色通道

![image-20220701175202057](http://www.cdn.liver0377.xyz/typora/202207011752117.png)

**解题思路**

- 这题和烽火传递很像
- 基本的思路就是枚举有所可能的最大空题段长度，然后选出长度最短，并且满足时间要求的空题段长度
- 可以在0 ~ n范围内采用二分来进行枚举操作
- 当空题段固定，即窗口大小固定时，这题的做法就和烽火传递完全一样了，只要求出在给定窗口下的最下代价，看其是否满足时间要求即可

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 5 * 1e4 + 10;
int f[N];
int a[N];
int q[N];
int n, m;

bool check(int limit) {                 // limit为窗口大小
    int hh = 0, tt = 0;
    
    for (int i = 1; i <= n + 1; i++) {  // 多算一位
        while (hh <= tt && q[hh] < i - limit) hh ++;
        f[i] = f[q[hh]] + a[i];
        while (hh <= tt && f[q[tt]] >= f[i]) tt --;
        q[++ tt] = i;
    }
    
    return f[q[hh]] <= m;
}

int main() {
   cin >> n >> m;
   for (int i = 1; i <= n; i++) cin >> a[i];
   
   int l = 1, r = n;             // 窗口大小范围为1 ~ n
   while (l < r) {
       int mid = l + (r - l) / 2;
       if (check(mid)) {
           r = mid;
       }else {
           l = mid + 1;
       }
   }
   
   cout << l - 1 << endl;       // 空题段大小应该为窗口大小 - 1
   
   return 0;
}
```



----

`tt`什么时候应该置为0而不是1？

- 通常在循环中需要使用到前缀时置为0



### 修建草坪

![image-20220701195830567](http://www.cdn.liver0377.xyz/typora/202207011958640.png)

**解题思路**

- 状态表示

![image-20220701200120391](http://www.cdn.liver0377.xyz/typora/202207012001448.png)

- 状态计算
  $f[i]=max(f[i-1],f[i - j - 1] - s[i - j] +s[i]) （1<=j<=k）$

  因此需要维护一个单调减的单调队列

- $0 <= i - j <= m$

  => $i - m <= j <= i$

  

**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

typedef long long LL;

const int N = 1e5 + 10;
LL f[N];
LL s[N];
LL a[N];
int q[N];
int n, K;

LL g(int x) {
    return f[max(0, x - 1)] - s[x];
}

int main() {
    cin >> n >> K;
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 1; i <= n; i++) s[i] += s[i - 1] + a[i];
    
    int hh = 0, tt = 0;       // 涉及到前缀和，添加一个标杆0
    for (int i = 1; i <= n; i++) {
        while (hh <= tt && q[hh] < i - K) hh ++;

        f[i] = max(f[i - 1], g(q[hh]) + s[i]);
        while (hh <= tt && g(q[tt]) <= g(i)) tt --;
        q[++ tt] = i;
    }
    
    cout << f[n] << endl;
    
    return 0;
}
```

- 这里使用了一个函数`g`用于计算$f[i - j - 1] - s[i - j]$

- 维护的窗口中的数其实是$f[i - K - 1] - s[i - K], f[i - K] - s[i - K + 1], ..., f[i - 1] - s[i]$

  每次将新的$g[i]$与队尾元素进行比较
