###  算法步骤
首先需要了解Prime算法的基本思想：
通过每次添加一个新节点到集合，直到所有点加入停止。
原理：每次连出该集合到其它所有点的最短边保证最小生成树的权值最小
1，首先随便选一个点加入集合
2，在其余点能够与这个集合相连的条件下，遍历这个集合中的点到其它点的权值
3，找出一个距离这个集合的任意一点距离最小的边
4，把这条边加入到这个集合
5，更新这个集合之后继续重复2步骤
6，直到所有的点都加入了这个集合

### 构造一个图
Prime解决的是一个图的问题，我们首先要构造一个图，图的要素有三个：点的个数，点的名称，线的权值，因此可以写出如下的函数

```
class Graphic{
    int vertexSize;
    char[] vertex;
    int[][] edges;
    public Graphic(int vertexSize,char[] vertex, int[][] edges){
        this.vertexSize = vertexSize;
        this.vertex = vertex;
        this.edges = edges;
    }
}
```
上面的vertexSize代表点的个数；vertex代表点的名称集合；edges是这个图的邻接矩阵。

### 接下来需要对于Prime算法进行解析
先上代码
```
class MinTree2{
    private Graphic graph;
    public MinTree2(Graphic graph) {
        this.graph = graph;
    }
    public void showGraph(){
        for(int[] edge:graph.edges){
            System.out.println(Arrays.toString(edge));
        }
    }
    public void Prime(int v){
        int[] visisted = new int[graph.vertexSize];
        int minV = -1;
        int minI = -1;
        visisted[v] = 1;
        int minDistance = Integer.MAX_VALUE;
        for(int i = 0; i < graph.vertexSize-1; i++){
            for(int j = 0; j < graph.vertexSize; j++){
                for(int k = 0; k < graph.vertexSize; k++){
                    if(visisted[j] == 1 && visisted[k] == 0 && graph.edges[j][k] != 0
                            && graph.edges[j][k] < minDistance){
                        minV = j;
                        minI = k;
                        minDistance = graph.edges[j][k];
                    }
                }
            }
            System.out.println(graph.vertex[minV] + "--"+graph.vertex[minI] +":" +minDistance);
            visisted[minI] = 1;
            minDistance = Integer.MAX_VALUE;
        }
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200822193229929.png#pic_center)

首先对于showGraph（）方法是显示当前图的邻接矩阵，Prime算法可以按照顺序依次输出我们选择的点。
对于Prime算法要点在于：
1，记录访问过的点  int[] visisted = new int[graph.vertexSize];对于visisted的下表对应着点，初始值自动设置为0，代表着没有访问过，当满足访问条件后置为1.
2，访问条件需要注意，每次确定一个点需要进入两重循环，需要满足的是一个点访问过了，也就是加入了已访问的集合，另一个点吗没有访问过，同时这个这两个点是相连且小于我们在这个循环已经确认的最小值，注意：当权值为0的时候，他是不相连的。
3，我们经过两重循环已经确认了最小值的点和最小值，还需要在保存了这个最小值后重新设置为Inter.MAX_VALUE，为了下次使用。


