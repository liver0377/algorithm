### A*算法

- 定义

  **A***算法其实就是在优先队列BFS的基础之上, 加入了一个启发函数`f`, `f[i]`表示从状态`i`到目标状态的估计代价

  使用优先队列来维护所有状态节点, 优先队列按照**当前代价 + 未来估价**进行排序, 每次取出最小的一个进行拓展

- **A*算法**有着一些限制与性质

  - 每个点到终点的估计距离`f`必须要小于等于它到终点的实际距离

  - 当终点出队时，其与起点的距离`dist`最小，即最优解，而中间节点则不一定

  - **A*算法**仅适用于有解的情况，如果无解的话，其仍然会搜索所有的节点，并且由于使用优先级队列来代替普通队列，一次拓展操

    作的时间复杂度为$O(lg(n))$, 相对于普通队列$O(1)$而言更高

  - **A*算法**不适用于有负权边的情况

- 当估计函数`f`返回0时，该算法就等同于堆优化`Dijkstra`





**模板**

下面给出的模板为**八数码**这题的代码模板

```cc
void bfs(string start) {
    heap.push({f(start) + 0, start});
    dist[start] = 0;
    
    
    while (!heap.empty()) {
        auto t = heap.top();
        heap.pop();
        
        string state = t.second;      // 队首状态
        if (state == ed) return;
        int step = dist[state];       // 队首到起点距离
        
        int k = state.find('x');
        int x = k / 3, y = k % 3;
        string old = state;
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];
            if (nx < 0 || ny < 0 || nx >= 3 || ny >= 3) continue;
            swap(state[nx * 3 + ny], state[x * 3 + y]);  // state成为新状态
            if (!dist.count(state) || step + 1 < dist[state]) {
                dist[state] = step + 1;
                pre[state] = {old, op[i]};
                heap.push({f(state) + dist[state], state});
            }
            swap(state[nx * 3 + ny], state[x * 3 + y]);
        }
    }
  
}
```

