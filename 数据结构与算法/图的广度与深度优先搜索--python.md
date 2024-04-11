本文利用python对图的广度优先搜索与深度优先搜索进行了编写，同时对广度优先搜索的最短路径问题进行了扩展
```python

class Graph(object):
    def __init__(self):
        print("请输入个数")
        self.point_num = 5
        self.graph = [[None for i in range(self.point_num)] for j in range(self.point_num)]
        self.peaks = ['a', 'b', 'c', 'd', 'e']
        print(self.peaks)

    def set_graph(self):
        self.graph[0][1] = 1
        self.graph[1][0] = 1
        self.graph[0][2] = 1
        self.graph[2][0] = 1
        self.graph[1][3] = 1
        self.graph[3][1] = 1
        self.graph[1][4] = 1
        self.graph[4][1] = 1
        print(self.graph)

    def BFS(self, start):
        queue = [] #利用队列的先进先出的原则来编写
        seen = [] # 存储结果存放位置
        queue.append(start)  
        while len(queue) > 0:
            id = self.peaks.index(queue[0])
            for j in range(self.point_num):
                if self.graph[id][j] == 1:
                    if self.peaks[j] not in seen:
                        queue.append(self.peaks[j]) 
            seen.append(self.peaks[id]) #遍历一个节点的所有子节点后将遍历完成的节点放入结果中，并取出
            queue.pop(0)
        print("遍历结果")
        print(seen)

    def shortest_path(self, start, end):
        queue = []
        seen = []
        parent = {start: None} #通过一个字典来存储每一个节点的父节点
        path = {start: 0} #存储每个节点的父节点的id
        queue.append(start)
        while len(queue) > 0:
            id = self.peaks.index(queue[0])
            for j in range(self.point_num):
                if self.graph[id][j] == 1:
                    if self.peaks[j] not in seen:
                        queue.append(self.peaks[j])
                        parent[self.peaks[j]] = self.peaks[id]
                        path[self.peaks[j]] = path[self.peaks[id]] + 1
            seen.append(self.peaks[id])
            queue.pop(0)
        print(f"{start}--{end}的最短路径：", end="")
        line = [end]
        w = parent[end]
        while w:
            line.append(w)
            w = parent[w]
        line.reverse()
        print(line)
        print("父节点：", end="")
        print(parent)
        print("距离起点的最短距离：", end="")
        print(path)

    def DFS(self, start):
        stack = [] #利用栈来进行深度遍历的工具
        seen = []
        stack.append(start)
        seen.append(start)
        while len(stack) > 0:
            start = stack[-1]
            id = self.peaks.index(start)
            for j in range(self.point_num):
                if self.graph[id][j] == 1:
                    if self.peaks[j] not in seen:
                        stack.append(self.peaks[j])
                        seen.append(self.peaks[j])
                        id = j
            stack.pop()
        print("深度优先遍历结果")
        print(seen)


if __name__ == '__main__':
    graph = Graph()
    graph.set_graph()
    graph.BFS('b')
    graph.DFS('a')
    graph.shortest_path('a', 'd')

['a', 'b', 'c', 'd', 'e']
[[None, 1, 1, None, None], [1, None, None, 1, 1], [1, None, None, None, None], [None, 1, None, None, None], [None, 1, None, None, None]]
广度优先遍历结果
['a', 'b', 'c', 'd', 'e']
深度优先遍历结果
['a', 'b', 'd', 'e', 'c']
a--d的最短路径：['a', 'b', 'd']
父节点：{'a': None, 'b': 'a', 'c': 'a', 'd': 'b', 'e': 'b'}
距离起点的最短距离：{'a': 0, 'b': 1, 'c': 1, 'd': 2, 'e': 2}







```
