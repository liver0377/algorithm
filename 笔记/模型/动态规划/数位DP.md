### 计数问题

![image-20220627151538845](http://www.cdn.liver0377.xyz/typora/202206271515917.png)

**解题思路**

![image-20220627155753950](http://www.cdn.liver0377.xyz/typora/202206271557016.png)

- 所有处于0 ~ n的数字均位于上面的树中

- 设函数`cnt(int n, int i)`的功能为返回1~n之间数字i出现的次数，注意，这里**不包括前导0**

- 设如下一般情况

  - n = abcdefg

  - 求数字`k`在第4位(自左向右位数增大)上出现的次数

    - 设树$m = {m_7}{m_6}{m_5}{m_4}{m_3}{m_2}{m_1}$

    - 先求左节点, 即$0 <= m4 <= d - 1$ 

      - 若`k`不为0，则$m_7m_6m_4$可以取`0~abc - 1`之间的任意一个数, 即`abc`个数, $m_3m_2m_1$可以随便取，取值范围为0~999, 即1000个数

        所以0 ~ n之间第4位取`k`的数的个数为 **abc * 1000**

      - 若`k`为0，则$m_7m_6m_5$可以取1~abc - 1之间的任意一个数, 即`abc - 1`个数，$m_3m_2m_1$可以随便取, 有1000个数

        所以0 ~ n 之间第4位取`k`的数的个数为**(abc - 1) * 1000**

        > - 当abc == 0时, 此时意味着正在求第1位，左节点是不存在的，直接跳过

    - 再求右节点

      根据`k`与`d`的大小关系进行讨论

      - `k` > `d`, 那么右节点不存在，跳过
      - `k` = `d`, 那么$m_3m_2m_1$的取值范围为`0~efg`, 数的个数为`efg + 1`
      - `k` < `d`, 那么当$m_4$取`k`之后，$m_3m_2m_1$仍然可以随便取，取值个数为1000

  - 将各种状态的次数全部加起来即可
  
  > 注意，这里计算的是每一位取数字`k`的所有可能数，但一个数可能在不同的位同时出现了该数字，因此会发生重复
  >
  > 但是这种重复是正确的，比如数字11，会在第1位以及第2位在求`1`出现的次数时时分别被算一次，这是正确的

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

// 计算n的数字位数
int dgt(int n) {
    int res = 0;
    while (n) res++, n /= 10;
    
    return res;
}

int cnt(int n, int i) {
    if (!n) return 0;
    
    int res = 0, sz = dgt(n);
    
    // n = 1234
    for (int j = 1; j <= sz; j++) { // j = 2， 这里是从右向左计数
        int p = pow(10, j - 1);     // p = 10
        int l = n / p / 10;         // l = 12  
        int r= n % p;               // r = 4
        int x = n / p % 10;         // x = 3
        // 1. 左节点
        // 1.1 i非0
        if (i > 0 && j != sz) { 
            res += l * p;
        }
        // 1.2 i为0
        if (!i && j != sz) {      // 当x == 0时，左边的高位不能够全部为0
            res += (l - 1) * p;  
        }
        
        // 2. 右节点
        // 2.1 右节点不存在
        if (i > x) {
            continue;
        }
        
        // 2.2 当在首位放置i时，i不能为0  
        if ((i < x) && (i || j != sz)) {
            res += p;
        }
        
        // 2.3 
        if ((i == x) && (i || j != sz)) {
            res += r + 1;
        }
    }
    
    return res;
}

int main() {
    int a, b;
    while (cin >> a >> b, a || b) {
        if (b < a) swap(a, b);
        
        for (int i = 0; i <= 9; i++) {
            cout << cnt(b, i) - cnt(a - 1, i) << " ";
        }
        cout << endl;
    }
    
    return 0;
}
```





### 度的数量

![image-20220625171607311](http://www.cdn.liver0377.xyz/typora/202206251716381.png)



**解题思路**

- 题意转换: 给定一个范围`[X, Y]`, 两个整数`K,` `B`求出在该范围内，满足一下条件的数字`Z`的个数

  - Z 的 B进制表示为$({a_{n - 1}}{a_{n - 2}}{a_{n - 3}...{a_2}{a_1}{a_0}})B$
  - 其中**$a_k$只能为0或者1， 并且满足所有$a_{k}$中1的个数为K个**

- 数位DP常采用的一个技巧是**前缀和**

  - 记$dp(n)$表示`0~n`之间所有满足上述条件的整数个数
  - 那么在给定范围`[X, Y]`内满足上面条件的整数个数为$dp(Y) - dp(X - 1)$

- 给定**N**, 其B进制表示为$({a_{n - 1}}{a_{n - 2}}{a_{n - 3}...{a_2}{a_1}{a_0}})B$

  - 任何位于`0~n`范围内的每个数字`X`的每一位$b_k$均满足下面的情况
    ![image-20220625173337300](http://www.cdn.liver0377.xyz/typora/202206251733358.png)
  - 我们要找的方案数就在这些数字当中

- 从前向后遍历**N**的每一位， 设当前正在遍历第`i`位, 数值为`X`，任何一种可能的方案的数字为M, i ~ n - 1中已经放置了`last`个1

  - 若`X > 1`

    - 则M可以在第`i`位放置0或者1

      - 放置0，则后面的0 ~ i - 1位，只要从中任取`K - last`位为1，将其它位置为0, 则都是合法方案

        方案数为$C_i^{k - last}$,  计算完毕

      - 放置1, 则后面的0 ~ i - 1位，只要从中任取`K - last - 1`位为1，将其它位置为0, 则都是合法方案

        方案数为$C_i^{k - last - 1}$, 计算完毕

  - 若`X == 1`

    - 则M可以在第`i`位放置0或者1
      - 放置0，则后面的0 ~ i - 1位，只要从中任取`K - last`位为1，将其它位置为0, 则都是合法方案
      - 放置1，`last ++`
        - 如果此时`last == K`, 表明1的数量已经设置完毕，后面只能够全为0，方案数 + 1
        - 如果此时`last < K`， 此时后面的位就不能够任意设置了，否则可能会超过**N**，应该继续向后遍历

  - 若`X == 0`

    - 则M只能放置0，仍然得继续向后遍历



**代码实现**

```cc
#include<bits/stdc++.h>
using namespace std;
const int N = 35; //位数

int f[N][N];// f[a][b]表示从a个数中选b个数的方案数，即组合数

int K, B; //K是能用的1的个数，B是B进制

//求组合数：预处理
void init(){
    for(int i=0; i< N ;i ++)
        for(int j =0; j<= i ;j++)
            if(!j) f[i][j] =1;
            else f[i][j] =f[i-1][j] +f[i-1][j-1];
}



 // 求区间[0,n]中的 “满足条件的数” 的个数
 // “满足条件的数”是指：一个数的B进制表示，其中有K位是1、其他位全是0
int dp(int n){

    if(n == 0) return 0; // 如果上界n是0，直接就是0种

    vector<int> nums;    // 存放n在B进制下的每一位
    //把n在B进制下的每一位单独拿出来
    while(n) nums.push_back(n % B) , n /= B;

    int res = 0;  //答案：[0,n]中共有多少个合法的数

    // last: 在自左向右枚举n的每一位过程中， 已经使用了多少个1 
    int last = 0; 

    //从最高位开始遍历每一位
    for(int i = nums.size()-1; i>= 0; i--){

        int x = nums[i]; 
         
        // 1. 计算左分支的方案数，仅当x >= 1时，左分支才存在
        if(x > 0){ 
            if (x > 1) {
                res += f[i][K - last];  // 第i为取0
                if (last < K) {         // 第i位取1的方案数
                    res += f[i][K - last - 1];
                }
                break;             // 当前位置不能够放置非0非1数x，结束遍历
            } else { // x == 1
                res += f[i][K - last]; // 第i位为0
                                       
                last ++;               // 放置x, 即1
                if (last > K) break;
            }
        }
        
        // 2. 最后一个右节点
        if (i == 0 && last == K) {  // 数字n本身就是一个合法方案
            res ++;
        }
    }

    return res;
}

int main(){
    init();
    int l,r;
    cin >>  l >> r >> K >>B;
    cout<< dp(r) - dp(l-1) <<endl;  
}

```





### 数字游戏

![image-20220625202857445](http://www.cdn.liver0377.xyz/typora/202206252028524.png)



**解题思路**

- 采取同样的套路，要求[a, b]内的不降数，转而求[0, n]范围内的不降数

- 处于[0, n]范围内的数满足下图

  ![image-20220625203119697](http://www.cdn.liver0377.xyz/typora/202206252031766.png)

  - 对每一层进行方案数的累加，即从高位到低位遍历数字，没遍历一位数字，就求出左节点的方案数

    同时尝试在该位放置$a[i]$

    - 如果$a[i] < last$ 即$a[i] < a[i - 1]$, 那么就表明这一位不能够再放置$a[i]$了，否则接下来的数将会不满足非降数的特征

      直接退出方案数的累计即可

    - 如果$a[i] >= last$, 表明放置$a[i]$是合法的，就可以继续判断下一位的方案数了

  - 这里在计算左节点的过程中，需要提前进行预处理

    - $f[i][j]$:  所有长度为`i`, 首位为`j`的数中，满足非降数定义的方案
    - $f[i][j] = \Sigma_k{f[i - 1][k]}$, 这里`k`表示第`i + 1`位数字，`j <= k <= 9`  

  - 左节点即在该位置放置`last ~ 9`的所有方案数



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 15;

int f[N][10]; // f[i][j]: 长度为i, 首位为j的数中，所有非减数的个数

void init() {
    for (int i = 0; i <= 9; i++) f[1][i] = 1;
    
    for (int len = 2; len <= N; len ++) {
        for (int j = 0; j <= 9; j++) {
            for (int k = j; k <= 9; k++) {
                f[len][j] += f[len - 1][k];
            }
        }
    }
}

int dp(int n) {
    if (!n) return 1;
    
    vector<int> nums;
    while (n) nums.push_back(n % 10), n /= 10;
    
    int last = 0;
    int res = 0;
    for (int i = nums.size() - 1; i >= 0; i--) {
        int x = nums[i];
        
        // 1. 左分支
        for (int j = last; j < x; j++) {
            res += f[i + 1][j];
        }
        
        // 2. 右分支
        if (x < last) { // 不能放置
            break;
        }
        last = x;       // 放置x
        if (!i) res ++; // 最后一位放置完毕, 并且没有break掉, 表明n本身就是一个合法方案
    }
    
    return res;
}

int main() {
    int a, b;
    // 预处理
    init();
    while (cin >> a >> b) {
        cout << dp(b) - dp(a - 1) << endl;
    }
    return 0;
}
```





### Windy数

![image-20220627190812542](http://www.cdn.liver0377.xyz/typora/202206271908614.png)



**解题思路**

- 依旧是套用之前的模板，不过这里要考虑前导0的问题，之前的题目中，前导0都是包含在内的

- 这里在枚举`n`的每一位时，会特判前导0，具体就是当在首位置数时，不能够置0

  ![image-20220627191037202](http://www.cdn.liver0377.xyz/typora/202206271910259.png)

  - 例: 013这个数，在计算时就不会被算作`windy`数，因为| 1 - 0 | < 2

    然而实际上，13确实是一个`windy`数，所以要分开来考虑

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

const int N = 10;

int f[N][10];

void init() {
    for (int i = 0; i <= 9; i++) f[1][i] = 1;
    
    for (int i = 2; i <= N; i++) {
        for (int j = 0; j <= 9; j++) {         // 注意这里可以以0开头
            for (int k = 0; k <= 9; k++) {
                if (abs(j - k) >= 2) {
                    f[i][j] += f[i - 1][k];
                }
            }
        }
    }
}

long long dp(int n) {
    if (!n) return 1;
    
    long long res = 0, last = -2;
    vector<int> nums;
    
    while (n) nums.push_back(n % 10), n /= 10;
    
    for (int i = nums.size() - 1; i >= 0; i--) {
        int x = nums[i];
        
        // 1. 左分支
        for (int j = i == nums.size() - 1; j < x; j++) {
            if (abs(j - last) >= 2)
                 res += f[i + 1][j];
        }
        
        if (abs(x - last) >= 2) 
            last = x;
        else 
            break;
            
        // 2. 右分支
        if (!i) res ++;
    }
    
    // 枚举长度在1~ size - 1范围内的所有windy数
    for (int i = 1; i < nums.size(); i++) {
        for (int j = 1; j <= 9; j++) {    // 这里没有从0开始，主要是因为可能会包含0, 00, 000等数
                                          // 并且1, 01等数会被认为是不同的数而累加多次
            res += f[i][j];
        }
    }
     
    res ++; // 算上0
    return res;
}

int main() {
    int a, b;
    
    init();
    cin >> a >> b;
    
    cout << dp(b) - dp(a - 1) << endl;
    
    return 0;
}
```

- 设`n`的位数为`sz`,在`dp()`函数的主循环中，只会考虑`sz`位数，并且这些数的首位均不为0
- 但是显然还有一些小于`sz`位的数，他们也是`windy`数，如1, 13等等
- 因此，单独的使用循环将他们包含进来



### 数字游戏II

![image-20220627200738036](http://www.cdn.liver0377.xyz/typora/202206272007107.png)



**解题思路**

- 此题的难点在于预处理过程
- 状态表示
  - $f[i][j][k]$: 长度为`i`，首位为`j`, 所有位之和模N之后的值为`k`的数的个数
- 状态计算
  - ![image-20220627201100837](http://www.cdn.liver0377.xyz/typora/202206272011884.png)



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>

using namespace std;

const int N = 12;
const int M = 102;

int f[N][10][M];
int P;

int mod(int a, int b) {
    return (a % b + b) % b;
}

void init() {
    memset(f, 0, sizeof f);
    
    for (int i = 0; i <= 9; i++) f[1][i][i % P] = 1;
    
    for (int i = 2; i < N; i++) {
        for (int j = 0; j <= 9; j++) {
            for (int k = 0; k < P; k++) {
                for (int x = 0; x <= 9; x++) {
                    f[i][j][k] += f[i - 1][x][mod(k - j, P)];
                }
            }
        }
    }
}

int dp(int n) {
    if (!n) return 1;
    
    // memset(f, 0, sizeof f);
    int res = 0, last = 0; // last表示枚举到第i位时，i + 1 ~ sz - 1位的数字总和
    vector<int> nums;
    
    while (n) nums.push_back(n % 10), n /= 10;
    
    for (int i = nums.size() - 1; i >= 0; i--) {
        int x = nums[i];
        
        for (int j = 0; j < x; j++) {
            res += f[i + 1][j][mod(-last, P)];  // 需要保证0~i位的总和m满足(m + last) % P == 0
        }
        
        last += x;
        
        if (!i && last % P == 0) {
            res ++;
        }
    }
    
    return res;
}

int main() {
    int a, b;
    
    while (cin >> a >> b >> P) {
        init();
        cout << dp(b) - dp(a - 1) << endl;
    }
    
    return 0;
}
```



### 不要62



**解题思路**

- 写出状态表示即可

- 状态表示

  $f[i][j]:$ 长度为`i`位，首位为`j`的数中，不包含4， 62的数的个数

- 状态计算

  - $f[i][j] = \Sigma{_k}{f[i - 1][k]}$, j, k 满足以下限制
    - `k`为2时，前面的`j`不能是6
    - `j`, `k`不能为4

**代码实现**

```cc
#include <iostream>
#include <vector>

using namespace std;

const int N = 11;

int f[N][10];

void init() {
    for (int i = 0; i <= 9; i++) f[1][i] = 1;
    f[1][4] = 0;
    
    for (int i = 2; i < N; i++) {
        for (int j = 0; j <= 9; j++) {
            for (int k = 0; k <= 9; k++) {
                if (j == 6 && k == 2) continue;
                if (j == 4 || k == 4) continue;
                f[i][j] += f[i - 1][k];
            }
        }
    }
}

int dp(int n) {
    if (!n) return 1;
    
    int res = 0, last = 0;
    vector<int> nums;
    
    while (n) nums.push_back(n % 10), n /= 10;
    
    for (int i = nums.size() - 1; i >= 0; i--) {
        int x = nums[i];
        
        for (int j = 0; j < x; j++) {
            if (j == 2 && last == 6) continue;
            if (j == 4) continue;
            res += f[i + 1][j];
        }
        
        if (x == 4 || (last == 6 && x == 2)) break;
        if (!i) res ++;
        
        last = x;
    }
    
    return res;
}

int main() {
    int a, b;
    init();
    
    while (cin >> a >> b, a || b) {
        cout << dp(b) - dp(a - 1) << endl;    
    }
    
    return 0;
}
```



### 恨7不成妻

![image-20220628164200515](http://www.cdn.liver0377.xyz/typora/202206281642600.png)

**解题思路**

[题解](https://www.acwing.com/solution/content/47501/)

**代码实现**

![](http://www.cdn.liver0377.xyz/typora/202206281644857.jpeg)

```cc
#include <iostream>
#include <vector>

using namespace std;

typedef long long LL;

const LL N = 20 , P = 1e9 + 7;

struct F{
    LL s0 , s1 , s2;
}f[N][10][7][7];

LL power7[N] , power9[N];//10^i%7,10^i%P

LL mod(LL x , LL y)//避免得到负数
{
    return (x % y + y) % y;
}

void init()
{
    //初始化长度是1的情况
    for(int i = 0 ; i < 10 ; i++)
        if(i != 7)
        {
            auto &v = f[1][i][i % 7][i % 7];
            v.s0++;
            v.s1 += i;
            v.s2 += i * i; 
        }  

    LL p = 10;
    for(int i = 2 ; i < N ; i++ , p *= 10)
        for(int j = 0 ; j < 10 ; j++)
        {
            if(j == 7) continue;
            for(int a = 0 ; a < 7 ; a++)
                for(int b = 0 ; b < 7 ; b++)
                    for(int k = 0 ; k < 10 ; k++)
                    {
                        if(k == 7) continue;
                        auto &v1 = f[i][j][a][b];
                        auto &v2 = f[i - 1][k][mod(a - j * mod(p , 7) , 7)][mod(b - j , 7)];
                        //开启痛苦面具，对照上面的公式，时刻提醒记得及时取模！！！！！！
                        v1.s0 = mod(v1.s0 + v2.s0 , P);
                        v1.s1 = mod(v1.s1 + j * (p % P) % P * v2.s0 % P + v2.s1  , P);
                        v1.s2 = mod(v1.s2 + 
                                v2.s0 * j % P * (p % P) % P * j % P * (p % P) % P + 
                                2 * j % P * (p % P) % P * v2.s1 % P + 
                                v2.s2 , P);
                    }
        }


    power7[0] = power9[0] = 1;
    for(int i = 1 ; i < N ; i++)
    {
        power7[i] = power7[i - 1] * 10 % 7;
        power9[i] = power9[i - 1] * 10 % P;
    }
}

//求出i位，且最高位是j，且本身模7不等于a，且各位数之和模7不等b的集合
//因为预处理的f[i][j][a][b]是本身模7等于a，且各位数之和模7等于b的集合，因此需要两趟for循环来实现
F get(int i , int j , int a , int b)
{
    LL s0 = 0 , s1 = 0 , s2 = 0;
    for(int x = 0 ; x < 7 ; x++)
        for(int y = 0 ; y < 7 ; y++)
        {
            if(x == a || y == b) continue;
            auto v = f[i][j][x][y];
            s0 = mod(s0 + v.s0 , P);
            s1 = mod(s1 + v.s1 , P);
            s2 = mod(s2 + v.s2 , P);
        }

    return {s0 , s1 , s2};
}

LL dp(LL n)
{
    if(!n) return 0;

    LL backup_n = n % P;//备份一个n
    vector<int> nums;
    while(n) nums.push_back(n % 10) , n /= 10;

    LL res = 0;
    LL last_a = 0 , last_b = 0;//last_a表示前缀本身的值，last_b表示前缀各位数之和
    for(int i = nums.size() - 1 ; i >= 0 ; i--)
    {
        int x = nums[i];
        for(int j = 0 ; j < x ; j++)
        {
            if(j == 7) continue;
            int a = mod(-last_a * power7[i + 1] , 7);
            int b = mod(-last_b , 7);

            auto v = get(i + 1 , j , a , b); //求得本身模7不等于a，并且各位数之和模7不等b的集合，此时就可以使用预处理出来的结构体
            //根据公式求s2，时刻提醒记得及时取模！！！！！！
            res = mod(res + 
                    (last_a % P) * (last_a % P) % P * (power9[i + 1] % P) % P * (power9[i + 1] % P) % P * v.s0 % P + 
                    2 * (last_a % P) * (power9[i + 1] % P) % P * v.s1 % P + 
                    v.s2 ,
                P);
        }

        if(x == 7) break;
        last_a = last_a * 10 + x;
        last_b += x;

        //记得特判n本身
        if(!i && last_a % 7 && last_b % 7) res = mod(res + backup_n * backup_n , P);
    }

    return res;
}

int main()
{
    init();

    int T;
    cin >> T;
    while(T--)
    {
        LL l , r;
        cin >> l >> r;
        cout << mod(dp(r) - dp(l - 1) , P) << endl;
    }
}
```

