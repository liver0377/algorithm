### 美观的花束



![image-20221008112018369](http://www.cdn.liver0377.xyz/typora/202210081120428.png)



**解题思路**

- 此题使用滑动窗口求解

- 窗口滑动过程中, 保持累加答案时, [i, j]区间总是合法解

- 使用一个全局的`map`来存储所有种类的花的出现次数

- 每次右移`j`, 令`map[flowers[j]] ++`

  在右移过程当中, 如果区间[i, j]由合法变为非法, 那么只有可能是`flowers[j]`的数量超了1个

  接下来需要不断右移`i`, 修正该区间使得该区间变为合法

- 累加`j - i + 1`, 进入下一次循环



**代码实现**

```cc
class Solution {
public:
    static const int mod = 1e9 + 7;

    int beautifulBouquet(vector<int>& flowers, int cnt) {
        int n = flowers.size();
        int res = 0;
        unordered_map<int, int> mp;
        int boom = 0; // 当前区间中非法的那种花朵的超出数目
        for (int i = 0, j = 0; j < n; j ++) {
            int old = mp[flowers[j]] ++;
            if (old <= cnt && old + 1 > cnt) boom ++;
            while (i <= j && boom > 0) {
                int old = mp[flowers[i]] --;
                if (old > cnt && old - 1 <= cnt) boom --;
                i ++;
            }
            res = (res + j - i + 1) % mod;
        }
        return res;
    }
};
```

