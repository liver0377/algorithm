**题干**

![image-20220423143102959](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220423143102959.png)



**题解**

- 解法1

  将矩阵翻转，看成是一颗树,以左下角为根节点

  - 如果该节点< target, 那么该列即可全部排除, j++
  - 如果该节点>target, 那么该行可全部排除, i--

  ![image-20220423143349279](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220423143349279.png)

  ```cpp
  bool Find(int target, vector<vector<int> > array) {
          if (array.size() == 0) {
              return false;
          }
          int i = array.size() - 1;
          int j = 0;
          while (i >= 0 && j < array[0].size()){
              if (array[i][j] < target) {
                  j++;
              } else if (array[i][j] > target) {
                  i--;
              } else {
                  return true;
              }
          }
          return false;
      }
  ```

  