**数据结构介绍**

Tries俗称字典树，可用于字符串统计，看起来像下面这样

- Tries主要包含**插入**以及**查询**两个操作

- 每次插入一个字符串时，会一个字母一个字母地进行插入,如果字母在正在遍历的路径上不存在，那么就新创建一个节点
- 当查询一个字符串时，如果字母在正在遍历的路径上不存在，那么就查询失败
- 图中的星号表示单词的结束，如果一个单词砸该路径上的该位置结束，那么就加一个标记

![image-20220414214533009](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220414214533009.png)

**代码实现**

```cpp
int son[N][26], cnt[N], idx; // idx会维护当前节点的数目
// 0号点既是根节点，又是空节点
// son[i][u]存储树中每个节点i的子节点, 如果是英文字符的话，那么u就应该等于25
// cnt[i]表示以i节点结尾的单词数量

// 插入一个字符串
void insert(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';           
        if (!son[p][u]) son[p][u] = ++ idx;     // 不存在结点则创建结点
        p = son[p][u];                        // 指向新结点
    }
    cnt[p] ++ ;
}

// 查询字符串出现的次数
int query(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}
```



**例题**

- [acwing835](https://www.acwing.com/problem/content/837/)

