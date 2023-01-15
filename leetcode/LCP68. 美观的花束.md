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
        unordered_map<int, int> mp;
        int left = 0;
        int right = 0;
        int n = flowers.size();
        int ans = 0;

        for (; right < n;) {
            int& t = ++ mp[flowers[right]] ;
            right ++;

            while (t > cnt) {
                mp[flowers[left]] --;
                left ++;
            }

            ans = (ans + (right - left) % mod) % mod;
        }

        return ans;
    }
};
```

