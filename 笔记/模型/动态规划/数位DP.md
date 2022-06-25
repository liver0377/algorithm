**模板**

```cc
int dfs(int pos, int pre, int lead, int limit) {
    if (!pos) {
        边界条件
    }
    if (!limit && !lead && dp[pos][pre] != -1) return dp[pos][pre];
    int res = 0, up = limit ? a[pos] : 无限制位;
    for (int i = 0; i <= up; i ++) {
        if (不合法条件) continue;
        res += dfs(pos - 1, 未定参数, lead && !i, limit && i == up);
    }
    return limit ? res : (lead ? res : dp[pos][sum] = res);
}
int cal(int x) {
    memset(dp, -1, sizeof dp);
    len = 0;
    while (x) a[++ len] = x % 进制, x /= 进制;
    return dfs(len, 未定参数, 1, 1);
}
signed main() {
    cin >> l >> r;
    cout << cal(r) - cal(l - 1) << endl;
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
                    break;             // 当前位置不能够放置非0非1数x，结束遍历
                }
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

