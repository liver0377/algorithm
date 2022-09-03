### ST算法

- 作用

  给定一个长度为N的数列`A`, `ST`算法能能够在$O(Nlog(N))$时间的预处理之后，以$O(1)$的时间复杂度在线回答

  "数列A中下标在l ~ r之间的数的最大值是多少" 这样的区间最值(RMQ)问题

![image-20220903135828985](http://www.cdn.liver0377.xyz/typora/202209031358048.png)



**预处理模板**

```cc
void ST_prework() {
    for (int i = 1; i <= n; i++) f[i][0] = a[i];
    int t = log(n) / log(2) + 1;
    for (int j = 1; j < t; j++) {
        for (int i = 1; i <= n - (1 << j) + 1; i++) {
            f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1]);
        }
    }
}
```

> log()函数为`cmath`的库函数

![image-20220903140158981](http://www.cdn.liver0377.xyz/typora/202209031401021.png)



**查询模板**

```cc
int ST_query(int l, int r) {
    int k = log(r - l + 1) / log(2);
    return max(f[l][k], f[r - (1 << k) + 1][k]);
}
```

