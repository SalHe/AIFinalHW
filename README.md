# 自定义启发式函数并应用A*算法解决迷宫寻路问题

## 简介

此项目为《人工智能》课程的大作业。

自定义一个启发式函数，应用于A*算法求解迷宫路径，并要求算法启发式函数具有可采纳性与单调。

程序URL：https://salhe.github.io/AIFinalHW/

若上述链接无法直接点开，请复制粘贴到Chrome浏览器中的地址栏然后按下回车打开。



## 运行说明

因为本项目为在线发布的项目，所以可能会随时更新内容，以实际访问到的页面为主。

并且在实际页面中会有相应的标注提示如何使用，请以实际页面为准。

### 环境

- Google **Chrome** 版本 84.0.4147.135（正式版本） （64 位）
- 可以访问Github的一台主机



## 使用说明

在线使用程序：https://salhe.github.io/AIFinalHW/

##### 自定义迷宫

1. 可在地图数据中自定义迷宫数据
2. 可在出口处定义出口的位置
3. 可在入口处定义入口的位置

##### 搜索路径

1. 勾选“动态展示”，当路径搜索完成后即可动态展示路径走向
2. 单击“开始搜索”，程序便自动开始搜寻路径

##### 生成迷宫

1. 根据需要设置迷宫尺寸，暂时只能设置为方阵
2. 单击”生成迷宫“



## 规则及算法

##### 求解限制

1. 从入口走到出口
2. 转弯次数尽可能少
3. 移动次数尽可能少

##### A*算法思路

1. 首先根据入口创建一个表$Open = \{S_0\}$
2. $Graph = \{S_0\}$
3. 如果$Open = \{\}$则视为失败
4. 在$Open$中取出$f(n)$最小的结点$n$，$n$放到$Closed$表中
5. 若$n \in S_g$即$n$以到达出口，则成功退出，并回溯出正确路径
6. 从$n$扩展结点，并过滤去出现过的结点、无效结点等无意义结点
7. 对最后得到的扩展结点进行处理：
   1. 若出现在$Graph$之中，则在$Graph$保留两者中$f(n)$一个较小者及其一切信息
   2. 若没有出现在$Graph$之中，则加入$Graph$和$Open$
8. 转至第3步



JavaScript代码及其注释：

```javascript
// 定义可扩展方向
const up_vector = [0, -1], right_vector = [1, 0],
    down_vector = [0, 1], left_vector = [-1, 0];

// 1. Open = { S0 }
let open = [{
    location: this.entrance, // 入口位置
    vector: [0, 0],			 // 指示由上一步到该步骤扩展的方向
    f: Number.POSITIVE_INFINITY, // f(n) = +∞
    g: 0, // g(n) = 0
    h: Number.POSITIVE_INFINITY, // f(n) = +∞
    parent: null  // 父节点为null
}];
// 2. Graph = { S0 }
const graph = [...open];  // graph（或者使用closed表）

// 3. 如果open表不为空
while (open.length > 0) {
    // 4. 从open表中拿到f最小的结点，并从open表移除
    open = open.sort((a, b) => a.f - b.f);
    const n_fmin = open.shift();
    
    // 5. 判断是否为出口
    if (n_fmin.location[0] === this.exit[0]
        && n_fmin.location[1] === this.exit[1]) {
        
        // 如果是，则从结点n的parent依次回溯得到由入口到出口的路径
        let p = n_fmin;
        const path = [];
        while (p) {
            path.unshift(p.location);
            p = p.parent;
        }
        resolve(path); // 响应正确路径
    }
    
    // 6. 没有找到出口则需要对结点进行扩展
    const expand_node = [up_vector, right_vector, down_vector, left_vector]
        // 求出下一步可能的位置
        .map(vec => {
            return {
                vector: vec,  // 标记该步走的方向
                location: [n_fmin.location[0] + vec[0], n_fmin.location[1] + vec[1]]  // 移动到的位置
            }
        })
        // 去掉越界、不可走的位置
        .filter(vec_loc => vec_loc.location[0] >= 0 && vec_loc.location[0] < this.cols
            && vec_loc.location[1] >= 0 && vec_loc.location[1] < this.rows
            && this.mapData[vec_loc.location[1]][vec_loc.location[0]] === 0
            && !(vec_loc.location[0] === this.entrance[0] && vec_loc.location[1] === this.entrance[1]))
        // 转换为结点
        .map(vec_loc => {
            // 消耗代价：转弯了为2，不转弯为1
            const g = n_fmin.g +
                ((vec_loc.vector[0] === n_fmin.vector[0] && vec_loc.vector[1] === n_fmin.vector[1]) ?
                    1 : 2);
            // 计算曼哈顿距离
            const h = this.exit[1] - vec_loc.location[1] + this.exit[0] - vec_loc.location[0];
            return {
                location: vec_loc.location,
                vector: vec_loc.vector,
                f: g + h,
                g: g,
                h: h,
                parent: n_fmin
            }
        });
    // 7. 对扩展结点做处理
    expand_node.forEach(node => {
        // 判断扩展结点是否在graph中
        const results = graph.filter(nodeInGraph => nodeInGraph.location[0] === node.location[0]
            && nodeInGraph.location[1] === node.location[1]);
        if (results.length > 0) {
            // 如果在的话，并且当前结点的f小于graph中结点的f
            // 就更新graph中的结点为当前节点
            const found = results[0];
            if (node.f < found.f) {
                found.parent = node.parent;
                found.f = node.f;
                found.g = node.g;
                found.h = node.h;
            }
        } else {
            // 节点不在graph中，加入到open表里
            // 并计入graph
            open.push(node);
            graph.push(node);
        }
    });
    reject(); // 未找到路径
}
```

##### 启发式函数

1. $g(n)$定义为由出发位置到$n$所消耗的代价
   1. 直走一步，代价为1
   2. 转弯一次，代价为2
2. $h(n)$定义为$n$到出口位置的曼哈顿距离
   1. 由于实际解决方案中，转弯的次数和走的路程必定大于以曼哈顿距离估计的路径要多，所以必定有$h(n) \le h^*(n)$，故所求出的解必定是最优解。

