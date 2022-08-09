### 牛的旅行

![image-20220809191826180](http://www.cdn.liver0377.xyz/typora/202208091918244.png)

![image-20220809191842310](http://www.cdn.liver0377.xyz/typora/202208091918354.png)





**解题思路**

![image-20220809191954815](http://www.cdn.liver0377.xyz/typora/202208091919862.png)

**代码实现**

```cc
#include <iostream>
#include <cstring>
#include <cmath>
using namespace std;

const int N=160,INF=0x3f3f3f3f;
typedef pair<int,int> PII;

double d[N][N];
double maxd[N];
int n;
char g[N][N];
PII q[N];

double get_distance(int i,int j){
    int x1=q[i].first,y1=q[i].second;
    int x2=q[j].first,y2=q[j].second;
    return sqrt(pow((x1-x2),2)+pow((y1-y2),2));
}

void floyd(){
    for(int k=1;k<=n;k++){
        for(int i=1;i<=n;i++){
            for(int j=1;j<=n;j++){
                d[i][j]=min(d[i][j],d[i][k]+d[k][j]);
            }
        }
    }
}

int main(){
    scanf("%d",&n);
    //读入坐标
    for(int i=1;i<=n;i++){
        scanf("%d%d",&q[i].first,&q[i].second);
    }
    //读入邻接矩阵
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            cin>>g[i][j];
        }
    }
    //建图
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(i==j) d[i][j]=0;
            else{
                if(g[i][j]=='1') d[i][j]=get_distance(i,j);
                else d[i][j]=INF;
            }
        }
    }

    floyd();
    //每个连通块中两点的最长距离
    double max1=-INF;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(d[i][j]<INF/2){  //如果i点和j点在一个连通块
                maxd[i]=max(maxd[i],d[i][j]);
            }
        max1=max(max1,maxd[i]);
        }
    }
    //两个连通块连一条边的最大距离的最短距离
    double max2=INF;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(d[i][j]==INF)   //如果当前连通块不连通,才可以连接
                max2=min(max2,maxd[i]+maxd[j]+get_distance(i,j));
        }
    }

    printf("%.6lf\n",max(max1,max2));
    return 0;

}

```

