### AC自动机

- 定义

  AC自动机是KMP + Trie的结合体, 它用于求解这样的问题:

  - 给定一个字符串集合S, 一个文本串str, 求解S中的字符串在str中的总出现次数





**存储**

AC自动机在存储上基本上和Trie树一模一样, 唯一的区别是每个节点多了一个`ne`指针

![image-20220913124631753](http://www.cdn.liver0377.xyz/typora/202209131246827.png)

- 在AC自动机中, 对于每个节点来说, 需要关注的是它到根节点的连线所构成的字符串`s`, 需要利用该字符串求解该节点对应的`ne`指针

- 设某个节点编号为`i`, 它代表的字符串记为`word[i]`, 且有`ne[i] == j`, 那么则有`word[j]`是`word[i]`的最长后缀

- 对于`s`, 它既代表一个普通的字符串, 也代表其它字符串的(严格)前缀

  在上图中1号节点, 它的后缀`he`在**整颗树**,有前缀`he`, 也就是2号节点和它对应, 因此将1号节点的`ne`指针指向2

  可以发现, 每个节点的`ne`指针指向的节点一定位于其上层, 这里因为严格前缀的长度一定小于字符串本身决定的
  
  因此, 可以对整颗树使用`bfs`宽搜一遍, 逐层修改每个节点的`ne`指针
  
  > 根节点以及第一层节点, ne指针肯定指向根节点, 因为整颗树中没有他们对应的(严格)公共前后缀, 因此在宽搜时, 直接将第一层的节点全部入队即可



**匹配过程**

对于文本串`s`, 其匹配过程完全遵循KMP思想





**模板**

- 时间复杂度: $O(n + m)$

下面给出的模板为[搜索关键词](https://www.acwing.com/problem/content/1284/)的模板
```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>

using namespace std;

const int S = 55;
const int N = S * 1e4 + 10;
const int M = 1e6 + 10;

int tr[N][26], cnt[N];     // tr[i][j]: 节点i的j孩子的编号
                           // cnt[i]: 节点i所代表的单词的出现次数
int ne[N];                 
char str[M];
int T, n, idx;

void insert() {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = str[i] - 'a';
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
    }
    cnt[p]++;//当前节点单词数+1
}

void build() {
    queue<int> q;
    for (int i = 0; i < 26; i++) {//将第二层所有出现了的字母扔进队列
        if (tr[0][i]) q.push(tr[0][i]);
    }

    while (q.size()) {
        int u = q.front();
        q.pop();
        for (int i = 0; i < 26; i++) {//查询26个字母
            int v = tr[u][i];
            if (!v) continue;

            int j = ne[u];
            while (j && !tr[j][i]) j = ne[j];
            if (tr[j][i]) j = tr[j][i];
            
            ne[v] = j;
            q.push(v);
        }
    }
}

int main() {
    cin >> T;
    while (T --) {
        memset(tr, 0, sizeof tr);
        memset(cnt, 0, sizeof cnt);
        memset(ne, 0, sizeof ne);
        idx = 0;
        cin >> n;
        for (int i = 0; i < n; i++) {
            scanf("%s", str);
            insert();
        }
        build();   // 求出每个节点的ne指针
        scanf("%s", str);
        int res = 0;
        for (int i = 0, j = 0; str[i]; i++) { //遍历文本串
            int t = str[i] - 'a';
            while (j && !tr[j][t]) j = ne[j];
            if (tr[j][t]) j = tr[j][t];
            int p = j;
            while (p) {      // 如果j所代表的字符串与str发生匹配, 那么其后缀串(ne)也肯定与str发生匹配, 
                             // 尝试判断其是否出现在集合S中, 
                             // 如果有的话也要算上
                res += cnt[p];
                cnt[p] = 0;  // 题干中指明只匹配一次, 因此匹配一次之后删除S中的该单词
                p = ne[p];
            }
        }
        printf("%d\n", res);
    }
    return 0;
}
```





### Trie图

Trie图是AC自动机的一种优化, 它会修正`tr`数组的含义, 在`Trie`图中, `tr[i][j]`表示:

1.  若`tr[i][j]`在原图中不存在, 设`s`表示`word[i]`后面添加上`j`字符, 则`tr[i][j]`表示`s`的最大后缀节点
2.  若`tr[i][j]`在原图中存在, 那么`tr[i][j]`则正常表示

- 图例

  设插入 `aba`, `aab`, `bbb`三个字符串, 左边红色线为`ne`指针, 右边黑色线为`tr`指针

![1663057603457](http://www.cdn.liver0377.xyz/typora/202209131627805.jpg)



通过上面的优化, 在进行文本串匹配时, 可以不用`ne`数组了, 直接使用`tr`数组进行一步跳转



**模板**

- 时间复杂度: $O(n + m)$, 与原方法相比缩小了常数

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>

using namespace std;

const int S = 55;
const int N = S * 1e4 + 10;
const int M = 1e6 + 10;

int tr[N][26], cnt[N];     // tr[i][j]: 节点i的j孩子的编号
                           // cnt[i]: 节点i所代表的单词的出现次数
int ne[N];                 
char str[M];
int T, n, idx;

void insert() {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = str[i] - 'a';
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
    }
    cnt[p]++;//当前节点单词数+1
}

void build() {
    queue<int> q;
    for (int i = 0; i < 26; i++) {//将第二层所有出现了的字母扔进队列
        if (tr[0][i]) q.push(tr[0][i]);
    }

    while (q.size()) {
        int u = q.front();
        q.pop();
        for (int i = 0; i < 26; i++) {//查询26个字母
            int v = tr[u][i];
            if (!v) tr[u][i] = tr[ne[u]][i];
            else {
              ne[v] = tr[ne[u]][i];
              q.push(v);
            }
        }
    }
}

int main() {
    cin >> T;
    while (T --) {
        memset(tr, 0, sizeof tr);
        memset(cnt, 0, sizeof cnt);
        memset(ne, 0, sizeof ne);
        idx = 0;
        cin >> n;
        for (int i = 0; i < n; i++) {
            scanf("%s", str);
            insert();
        }
        build();   // 求出每个节点的ne指针
        scanf("%s", str);
        int res = 0;
        for (int i = 0, j = 0; str[i]; i++) { //遍历文本串
            int t = str[i] - 'a';
            j = tr[j][t];
            int p = j;
            while (p) {      // 如果j所代表的字符串与str发生匹配, 那么其后缀串(ne)也肯定与str发生匹配, 
                             // 尝试判断其是否出现在集合S中, 
                             // 如果有的话也要算上
                res += cnt[p];
                cnt[p] = 0;  // 题干中指明只匹配一次, 因此匹配一次之后删除S中的该单词
                p = ne[p];
            }
        }
        printf("%d\n", res);
    }
    return 0;
}
```



