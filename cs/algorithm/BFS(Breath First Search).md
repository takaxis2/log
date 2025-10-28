너비 우선 탐색은 루트/임의의 노드에서 시작하여 인접한 노드를 먼저 탐색을 수행하는 알고리즘이 다.

- 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어진 정점을 나중에 방문한다.
- 두 노드 사이의 최단 경로 또는 임의의 경로를 찾을 때 사용한다.

  

특징

- 어떤 노드를 방문했는지 여부를 검사 해야 한다 → 무한 루프 가능성
- 일반적으로 큐(queue)를 사용하여 구현한다

  

1. 시작 노드를 큐에 넣고 방문 처리를 한다. (방문 처리는 set로 하면 편하다)
2. 인접 노드 중 방문하지 않은 노드를 큐에 추가하고 방문한다.
3. 반복

  

java

```Java
void bfs(Map<Integer, List<Integer>> graph, int node) {
      Deque<Integer> q = new ArrayDeque<>();
      Set<Integer> visited = new HashSet<>();
      q.addLast(node);
      visited.add(node);

      while (q.isEmpty() == false) {
          int cur = q.removeFirst();
          System.out.print(cur + " ");
          for (int next : graph.get(cur)) {
              if (visited.contains(next)) continue;
              q.addLast(next);
              visited.add(next);
          }
      }
  }

//좀 더 깔끔
void search(Node root) {
  Queue queue = new Queue();
  root.marked = true; // (방문한 노드 체크)
  queue.enqueue(root); // 1-1. 큐의 끝에 추가

  // 3. 큐가 소진될 때까지 계속한다.
  while (!queue.isEmpty()) {
    Node r = queue.dequeue(); // 큐의 앞에서 노드 추출
    visit(r); // 2-1. 큐에서 추출한 노드 방문
    // 2-2. 큐에서 꺼낸 노드와 인접한 노드들을 모두 차례로 방문한다.
    foreach (Node n in r.adjacent) {
      if (n.marked == false) {
        n.marked = true; // (방문한 노드 체크)
        queue.enqueue(n); // 2-3. 큐의 끝에 추가
      }
    }
  }
}
```

c#

```C#
using System;
using System.Collections.Generic;
using DataStructures.Graph;

namespace Algorithms.Graph
{
    /// <summary>
    /// Breadth First Search - algorithm for traversing graph.
    /// Algorithm starts from root node that is selected by the user.
    /// Algorithm explores all nodes at the present depth.
    /// </summary>
    /// <typeparam name="T">Vertex data type.</typeparam>
    public class BreadthFirstSearch<T> : IGraphSearch<T> where T : IComparable<T>
    {
        /// <summary>
        /// Traverses graph from start vertex.
        /// </summary>
        /// <param name="graph">Graph instance.</param>
        /// <param name="startVertex">Vertex that search starts from.</param>
        /// <param name="action">Action that needs to be executed on each graph vertex.</param>
        public void VisitAll(IDirectedWeightedGraph<T> graph, Vertex<T> startVertex, Action<Vertex<T>>? action = default)
        {
            Bfs(graph, startVertex, action, new HashSet<Vertex<T>>());
        }

        /// <summary>
        /// Traverses graph from start vertex.
        /// </summary>
        /// <param name="graph">Graph instance.</param>
        /// <param name="startVertex">Vertex that search starts from.</param>
        /// <param name="action">Action that needs to be executed on each graph vertex.</param>
        /// <param name="visited">Hash set with visited vertices.</param>
        private void Bfs(IDirectedWeightedGraph<T> graph, Vertex<T> startVertex, Action<Vertex<T>>? action, HashSet<Vertex<T>> visited)
        {
            var queue = new Queue<Vertex<T>>();

            queue.Enqueue(startVertex);

            while (queue.Count > 0)
            {
                var currentVertex = queue.Dequeue();

                if (currentVertex == null || visited.Contains(currentVertex))
                {
                    continue;
                }

                foreach (var vertex in graph.GetNeighbors(currentVertex))
                {
                    queue.Enqueue(vertex!);
                }

                action?.Invoke(currentVertex);

                visited.Add(currentVertex);
            }
        }
    }
}
```