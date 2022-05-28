**基本思想**

KMP算法的应用是判断一个字符串是否是另一个字符串的子串，较长的串称为**文本串**，较短的串称为**模式串**

首先来看看暴力解法

- 暴力解法

  ```cpp
  const int N = 100010;
  int s[N], p[N], n, m;  // n为s的长度, m为p的长度
  bool flag;
  cin >> s >> p;
  
  for (int i = 0; i < n - m + 1; i++) {
      flag = true;
      for (int j = 0; j < m; j++) {
          if (s[i + j] != p[j]) {
              flag = false;
              break;
          }
      }
  }
  
  if (flag) {
      cout << "匹配" << endl;
  } else {
      cout << "不匹配" << endl;
  }
  ```

  暴力解法的时间复杂度是O((n- m + 1)m)

使用KMP算法可以将时间复杂度优化至**O(m + n)**

KMP算法的最重要的点就是要找到模式串**最长公共前后缀**，通过它，就可以加快模式串的移动步伐

- 例子

  ![image-20220413104019404](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104019404.png)

  上面的串是文本串，下面的串是模式串，指针指向的是模式串，可以看出，前5个字符完全匹配，但是第六个字符不匹配，此时就需要利用到最长公共前后缀

  ![image-20220413104149356](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104149356.png)

  指针指向的模式串左边的最长公共前后缀如图所示，为`AB`,利用它，就可以使模式串一次跨越多个字符

  **直接将模式串的开头移动到后缀的首部**

  ![image-20220413104453013](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104453013.png)

  此时继续对指针所指向的位置进行比较

  - 如果指针所指字符文本串与模式串相同，那么指针向后移动，否则停止

  当指针再次停止时，递归的进行上述操作，即寻找**最长公共前后缀**

  ![image-20220413104732357](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104732357.png)

![image-20220413104752882](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104752882.png)

![image-20220413104821223](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413104821223.png)

上面的图中，模式串已经超出范围，所以就不匹配，但实际中肯定会有更多的情况，这里只是列出KMP的基本思想



- 代码实现

  **next数组**

  - 使用next[]数组来进行字符不匹配情况下j指针的跳转，每次都是向前跳转

  - next[j]表示模式串中以以下标 j 结尾的子串的公共最长前后缀的长度，使用符号表示为:

    next[ j ] :  p[ 1, j ]串中最长公共前后缀的最大长度，满足 **p[ 1, next[ j ] ] = p[ j - next[ j ] + 1, j ]**

    这里的`j`指针十分微妙，`p[1, j]`始终是`p[1, i - 1]`的公共最长前后缀,每次比较都是拿p[i]与p[j + 1]进行比较

  - 如果`next[j] == 0`, 那么代表没有公共前后缀

  - next[]数组的计算

    - 匹配的情况: p[i] == p[j + 1]

    ![image-20220413112911715](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413112911715.png)

    - 不匹配的情况: p[i] != p[j + 1]

      意味着p[i - j + 1, i]与[1, j + 1]不匹配，此时只能寄希望于子串p[1, j]了，看看它有没有公共最长前后缀

      因此令`j = ne[j]`

  ![image-20220413112033914](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220413112033914.png)

  

  **匹配**

  匹配的过程与求next数组的过程一模一样，代码也是基本相同

  注意第7,8行与第15,16行比较的区别

  ```cpp
  // s[]是长文本，p[]是模式串，n是p的长度，m是s的长度
  // 求模式串的Next数组：
  char p[N], s[M], ne[M];
  cin >> n >> p + 1 >> m >> s + 1;
      
      for (int i = 2, j = 0; i <= n; i++) {
          while (j && p[j + 1] != p[i]) j = ne[j];
          if (p[j + 1] == p[i]) j++;
          ne[i] = j;
      }
      
      for (int i = 1, j = 0; i <= m; i++) {
          while (j && s[i] != p[j + 1]) j = ne[j];
          if (s[i] == p[j + 1]) j ++;
          if (j == n) {
              // ...
              j = ne[j];
          }
      }
  ```
  
  - 上面的文本串以及模板串中下标是从1开始的，即下标为0中存储的是无效数据
