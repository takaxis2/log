-합집합 찾기

-서로소 집합(disjoint set)

-여러개의 노드가 존재할때 두개의 노드를 선택해서, 현재의 노드가 같은 그래프에 속하는지 확인하는 알고리즘

  

node.cs

```C#
namespace DataStructures.DisjointSet
{
    /// <summary>
    /// node class to be used by disjoint set to represent nodes in Disjoint Set forest.
    /// </summary>
    /// <typeparam name="T">generic type for data to be stored.</typeparam>
    public class Node<T>
    {
        public int Rank { get; set; }

        public Node<T> Parent { get; set; }

        public T Data { get; set; }

        public Node(T data)
        {
            Data = data;
            Parent = this;
        }
    }
}
```

disjointSet.cs

```C#
using System.Collections;

namespace DataStructures.DisjointSet
{
    /// <summary>
    /// Implementation of Disjoint Set with Union By Rank and Path Compression heuristics.
    /// </summary>
    /// <typeparam name="T"> generic type for implementation.</typeparam>
    public class DisjointSet<T>
    {
        /// <summary>
        /// make a new set and return its representative.
        /// </summary>
        /// <param name="x">element to add in to the DS.</param>
        /// <returns>representative of x.</returns>
        public Node<T> MakeSet(T x) => new(x);

        /// <summary>
        /// find the representative of a certain node.
        /// </summary>
        /// <param name="node">node to find representative.</param>
        /// <returns>representative of x.</returns>
        public Node<T> FindSet(Node<T> node)
        {
            if (node != node.Parent)
            {
                node.Parent = FindSet(node.Parent);
            }

            return node.Parent;
        }

        /// <summary>
        /// merge two sets.
        /// </summary>
        /// <param name="x">first set member.</param>
        /// <param name="y">second set member.</param>
        public void UnionSet(Node<T> x, Node<T> y)
        {
            Node<T> nx = FindSet(x);
            Node<T> ny = FindSet(y);
            if (nx == ny)
            {
                return;
            }

            if (nx.Rank > ny.Rank)
            {
                ny.Parent = nx;
            }
            else if (ny.Rank > nx.Rank)
            {
                nx.Parent = ny;
            }
            else
            {
                nx.Parent = ny;
                ny.Rank++;
            }
        }
    }
}
```

  

여기서는 rank로 부모를 정했지만 간단하게 값으로 정해도 된다.

보통 작은 쪽이 부모 노드가 된다.