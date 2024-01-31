깊이 우선 탐색은 루트/임의의 노드의 가장 깊은 부분을 우선적으로 탐색한다.

미로를 탐색할 때 한 방향으로 갈 수 있을 때까지 계속 가다가 더 이상 갈 수 없게 되면 다시 가장 가까운 갈림길로 돌아와서 이곳으로부터 다른 방향으로 다시 탐색을 진행하는 방법과 유사하다.

  

특징

- 자기 자신을 호출하는 순환 알고리즘의 형태를 가지고 있다(스택)
- 순회 순서에 따라 전위, 중위, 후위 순회의 3가지 방법이 있다

  

1. 시작 노드를 스택에 넣고 방문 처리를 한다.
2. 스택의 최상단 노드를 확인한다.
3. 최상단 노드에 방문하지 않은 인접 노드가 있다면 스택에 추가, 방문한다
4. 방문하지 않은 인접 노드가 없다면 스택에서 최상단 노드를 제거  
      
    

java

```Java
void search(Node root) {
  if (root == null) return;
  // 1. root 노드 방문
  visit(root);
  root.visited = true; // 1-1. 방문한 노드를 표시
  // 2. root 노드와 인접한 정점을 모두 방문
  for each (Node n in root.adjacent) {
    if (n.visited == false) { // 4. 방문하지 않은 정점을 찾는다.
      search(n); // 3. root 노드와 인접한 정점 정점을 시작 정점으로 DFS를 시작
    }
  }
}

void dfs(Map<Integer, List<Integer>> graph, int cur, Set<Integer> visited) {
      if (visited.contains(cur)) return;
      visited.add(cur);
      System.out.print(cur + " ");
      for (int next : graph.get(cur)) {
          dfs(graph, next, visited);
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
    /// Depth First Search - algorithm for traversing graph.
    /// Algorithm starts from root node that is selected by the user.
    /// Algorithm explores as far as possible along each branch before backtracking.
    /// </summary>
    /// <typeparam name="T">Vertex data type.</typeparam>
    public class DepthFirstSearch<T> : IGraphSearch<T> where T : IComparable<T>
    {
        /// <summary>
        /// Traverses graph from start vertex.
        /// </summary>
        /// <param name="graph">Graph instance.</param>
        /// <param name="startVertex">Vertex that search starts from.</param>
        /// <param name="action">Action that needs to be executed on each graph vertex.</param>
        public void VisitAll(IDirectedWeightedGraph<T> graph, Vertex<T> startVertex, Action<Vertex<T>>? action = default)
        {
            Dfs(graph, startVertex, action, new HashSet<Vertex<T>>());
        }

        /// <summary>
        /// Traverses graph from start vertex.
        /// </summary>
        /// <param name="graph">Graph instance.</param>
        /// <param name="startVertex">Vertex that search starts from.</param>
        /// <param name="action">Action that needs to be executed on each graph vertex.</param>
        /// <param name="visited">Hash set with visited vertices.</param>
        private void Dfs(IDirectedWeightedGraph<T> graph, Vertex<T> startVertex, Action<Vertex<T>>? action, HashSet<Vertex<T>> visited)
        {
            action?.Invoke(startVertex);

            visited.Add(startVertex);

            foreach (var vertex in graph.GetNeighbors(startVertex))
            {
                if (vertex == null || visited.Contains(vertex))
                {
                    continue;
                }

                Dfs(graph, vertex!, action, visited);
            }
        }
    }
}
```