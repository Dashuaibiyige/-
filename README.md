# A*算法的基本原理

## 用图的形式表现地图

这里说到的图是graph,而不是map,图(graph)是由一系列的节点和连接节点的边来描述的一种模型.我们使用graph来描述代价地图的时候,只需要把地图中的位置转化为graph中的节点,用连接关系代表位置之间的可到达性,这样以来,在进行A* 算法的时候就可以直接对图(graph)进行操作,而完全抛弃原来的几何地图.
在A* 算法的实现中,是这样将地图(map)转化成图(graph)的:
* 首先,定义节点
```c++
typedef struct
{
    int x, y;
    unsigned char reachable, sur, value;
} MapNode;
```
节点中包括该点在地图中的坐标(x,y),地图到相邻点的可到达性(reachable),在代价地图中是否被占用(value).
* 定义图graph
```c++
static MapNode graph[Height][Width];
```
各节点之间的连接关系是通过每个节点的可达性(reachable)表示的.
* 将地图Map表示成图Graph的形式通过以下代码实现

```c++
void initGraph(const int map[Height][Width], int sx, int sy, int dx, int dy)
{    //地图发生变化时重新构造地
    int i, j;
    srcX = sx;    //起点X坐标
    srcY = sy;    //起点Y坐标
    dstX = dx;    //终点X坐标
    dstY = dy;    //终点Y坐标
    for (i = 0; i < Height; i++)
    {
        for (j = 0; j < Width; j++)
        {
            graph[i][j].x = i; //地图坐标X
            graph[i][j].y = j; //地图坐标Y
            graph[i][j].value = map[i][j];
            graph[i][j].reachable = (graph[i][j].value == Reachable);    // 节点可到达性
            graph[i][j].sur = 0; //邻接节点个数
            if (!graph[i][j].reachable)
            {
                continue;
            }
            if (j > 0)
            {
                if (graph[i][j - 1].reachable)    // left节点可以到达
                {
                    graph[i][j].sur |= West;
                    graph[i][j - 1].sur |= East;
                }
                if (i > 0)
                {
                    if (graph[i - 1][j - 1].reachable
                        && graph[i - 1][j].reachable
                        && graph[i][j - 1].reachable)    // up-left节点可以到达
                    {
                        graph[i][j].sur |= North_West;
                        graph[i - 1][j - 1].sur |= South_East;
                    }
                }
            }
            if (i > 0)
            {
                if (graph[i - 1][j].reachable)    // up节点可以到达
                {
                    graph[i][j].sur |= North;
                    graph[i - 1][j].sur |= South;
                }
                if (j < Width - 1)
                {
                    if (graph[i - 1][j + 1].reachable
                        && graph[i - 1][j].reachable
                        && map[i][j + 1] == Reachable) // up-right节点可以到达
                    {
                        graph[i][j].sur |= North_East;
                        graph[i - 1][j + 1].sur |= South_West;
                    }
                }
            }
        }
    }
}
```
这样以来,我们可以完全抛弃之前的地图的概念,不去考虑地图几何特征,也可以抛开距离长度等要素,而是直接对图Graph进行操作.
## Openlists和Closelists
A* 算法是一种基于搜索的路径规划算法,在进行搜索的时候,我们期望有一个"域"来限定我们的搜索范围,以免工作量过大限制效率,也就是说我们希望知道哪些节点是有必要搜索的,哪些节点是不需要搜索而直接忽略的.
