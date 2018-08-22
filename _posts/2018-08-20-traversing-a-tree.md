---
layout:     post
title:      Traversing a Tree
summary:    Taking a look at the options we have to traverse a tree or graph.
date:       2018-08-20 13-27-55
categories: cpp
---

Sometimes we find ourselves dealing with an algorithm that involves a binary tree
(each node can only have, at most, two childs) and we need to traverse it in order to
achieve the goal of the algorithm. The goal of this post is to describe the options we have to traverse a tree,
which are:

- [Trees](#tree)
  - [Pre-Order](#pre-order)
  - [In-Order](#in-order)
  - [Post-Order](#post-order)
- [Graphs](#graph)
  - [Breadth First Search (BFS)](#bfs)
  - [Depth First Search (DFS)](#dfs)
- [Example](#example)

# Tree

Before getting into the ways in which we can traverse a tree, a short C++ snippet will be added to show how we would
create the tree in the image:

![Tree](/images/tree/tree1.png)

```cpp
#include <iostream>

template<class T>
class Node
{
 public:
  T data;
  Node<T> *left;
  Node<T> *right;

  Node(T data) {
   this->data = data;
  }
};

int main() {

 Node<int> *root    = new Node<int>(5);
 root->left         = new Node<int>(6);
 root->right        = new Node<int>(2);
 root->left->left   = new Node<int>(7);
 root->right->left  = new Node<int>(0);
 root->right->right = new Node<int>(1);

 std::cout << "Tree has been generated! :)" << std::endl;

 return 0;
}
```

We are using the *int* type for the example but bear in mind that a tree's node can contain any object we'd like.

## Pre-Order

Pre-Order tree traversal consists in checking the given node before the node's childs. Using this technique,
the first node that will be printed will always be the root node, followed by the left and right subtrees.

```cpp
template<class T>
void preOrder(Node<T>* node) {

 // Base case
 if (node == NULL) {
  return;
 }
 std::cout << node->data << std::endl;
 preOrder(node->left);
 preOrder(node->right);
}
```

The output of Pre-Order traversal is: **5 6 7 2 0 1**

## In-Order

In-Order tree traversal consists in visiting the node after visiting the left child. The order is: left -> node -> right.

```cpp
template<class T>
void inOrder(Node<T>* node) {

 // Base case
 if (node == NULL) {
  return;
 }
 preOrder(node->left);
 std::cout << node->data << std::endl;
 preOrder(node->right);
}
```

The output of In-Order traversal is: **7 6 5 0 2 1**

## Post-Order

Post-Order tree traversal consists in checking the childs of a given node before the node itself. Using this technique,
the first node that will be printed will be the bottom left-most node, if it exists.

```cpp
template<class T>
void postOrder(Node<T>* node) {

 // Base case
 if (node == NULL) {
  return;
 }
 preOrder(node->left);
 preOrder(node->right);
 std::cout << node->data << std::endl;
}
```
The output of Post-Order traversal is: **7 6 0 1 2 5**

# Graph

Before getting into the ways in which we can traverse a graph, a java snippet will be added to show
how we would create the graph in the image:

![graph](/images/tree/graph1.png)

This is the `Graph` class, which contains an inner class called `Node`:

```java
class Graph {

    private Collection<Node> nodes = new ArrayList<>();
    private Map<Integer, Node> map = new HashMap<>();

    Node getOrCreateNode(int id) {

        if (!map.containsKey(id)) {
            final Node newNode = new Node(id);
            nodes.add(newNode);
            map.put(id, newNode);
        }
        return map.get(id);
    }

    void addEdge(int startId, int endId) {

        final Node start = map.get(startId);
        final Node end = map.get(endId);

        if (start == null || end == null) {
            throw new IllegalArgumentException("Illegal nodes' id.");
        }

        start.addNeighbour(end);
    }

    int getSize() {
        return this.nodes.size();
    }

    class Node {

        private Collection<Node> neighbours = new ArrayList<>(); // Nodes connected to this node.
        private Map<Integer, Node> map = new HashMap<>();
        final int id;

        Node(final int id) {
            this.id = id;
        }

        void addNeighbour(Node node) {

            if (!map.containsKey(node.id)) {
                neighbours.add(node);
                map.put(node.id, node);
            }
        }

        Collection<Node> getNeighbours() {
            return this.neighbours;
        }
    }
}
```

Now we can use this `Graph` class to create our graph in our `main` function:

```java
public static void main(String[] args) {

        final Graph graph = new Graph();

        final Graph.Node node0 = graph.getOrCreateNode(0);
        final Graph.Node node1 = graph.getOrCreateNode(1);
        final Graph.Node node2 = graph.getOrCreateNode(2);
        final Graph.Node node5 = graph.getOrCreateNode(5);
        final Graph.Node node6 = graph.getOrCreateNode(6);
        final Graph.Node node7 = graph.getOrCreateNode(7);

        node0.addNeighbour(node7);
        node0.addNeighbour(node1);

        node1.addNeighbour(node0);
        node1.addNeighbour(node2);

        node2.addNeighbour(node1);
        node2.addNeighbour(node6);

        node5.addNeighbour(node1);
        node5.addNeighbour(node6);
        node5.addNeighbour(node7);

        node6.addNeighbour(node2);
        node6.addNeighbour(node5);

        node7.addNeighbour(node5);
        node7.addNeighbour(node0);
}
```

Now that we have finished generating the graph, we can start discussing BFS and DFS.

## Breadth First Search

BFS is an algorithm that is mainly used for traversing graphs, although it can also be used with trees.
The idea behind BFS is the following: "_Check all your childs before checking your childs' childs_".

To show an example of how BFS is used, we will consider the path problem: **Are nodes A and B connected?**. This is
the classical algorithm that is used to learn how BFS works. It can be written both in an iterative way and in a
recursive way. The recursive way will probably be easier to read but it will consume stack space in our memory, whereas
the iterative approach will be more difficult to read but it will use the RAM memory in our computer instead of the
stack space. The iterative version will also need an additional data structure, which in this case is a queue.

This is the method that would be accessed from the outside (public method):
See that we need to create a new array in order to keep track of which nodes we have already visited.
Otherwise we would keep looping forever.

```java
public boolean isTherePathBetweenBFS(final Graph g, final Graph.Node start, final Graph.Node end) {

    boolean[] visited = new boolean[g.getSize()]; // All false by default
    return BFS(g, start, end, visited);
}
```

And the (private) BFS method:

```java
private boolean BFS(Graph.Node start, Graph.Node end, boolean[] visited) {

    if (start == end) {
        return true;
    }

    final Queue<Graph.Node> queue = new LinkedList<>();
    queue.offer(start);
    visited[start.id] = true;
    while (!queue.isEmpty()) {
        Graph.Node node = queue.poll();
        visited[node.id] = true;
        for (Graph.Node child : node.getNeighbours()) {
            if (!visited[child.id]) {
                if (child.id == end.id)
                    return true;
                else
                    queue.offer(child);
            }
        }
    }
    return false;
}
```

## Depth First Search

The idea behind BFS is exactly the opposite of BFS: "_Check all your childs' childs before checking your own childs_".
We will write the recursive version of DFS, which is more common.

The public method is:

```java
public boolean isTherePathBetweenDFS(Graph g, Graph.Node start, Graph.Node end) {

    boolean[] visited = new boolean[g.getSize()];
    return DFS(start, end, visited);
}
```

And the private method:

```java
private boolean DFS(Graph.Node start, Graph.Node end, boolean[] visited) {

    if (start == end) {
        return true;
    }

    visited[start.id] = true;
    for (Graph.Node child : start.getNeighbours()) {
        if (!visited[child.id]) {
            if (DFS(child, end, visited)) {
                return true;
            }
        }
    }
    return false;
}

```

DFS is also useful for computing the shortest path between two nodes, whereas BFS isn't.

# Example

We will now solve an algorithm using BFS so you can see how it is used. The algorithm consists in checking **what is
the maximum number of connected number** of a given grid. We can only connect cells using up, down, left and right
directions. Imagine our grid is the following:

```
    {3, 2, 3, 3, 3}
    {2, 1, 3, 1, 3}
    {3, 1, 2, 1, 3}
    {2, 1, 1, 1, 3}
```

The algorithm should return **7** because it is the max number of connected integers, which in this case are all
the **1's**.

We will first create a helper class called `Cell`, in which we will specify the dimensions of the grid. We will
also add a method to retrieve the childs (connected cells) of a given cell:

```java
class Cell {

    private final static int MAX_ROWS = 3;
    private final static int MAX_COLS = 4;
    final int row;
    final int col;

    public Cell(int row, int col) {
        this.row = row;
        this.col = col;
    }

    public Collection<Cell> getChilds() {

        Collection<Cell> childs = new ArrayList<>();
        if (row > 0)        childs.add(new Cell(row-1, col));
        if (row < MAX_ROWS) childs.add(new Cell(row+1, col));
        if (col > 0)        childs.add(new Cell(row, col-1));
        if (col < MAX_COLS) childs.add(new Cell(row, col+1));
        return childs;
    }
}
```

We can now show the algorithm:

```java
class Main {

    public static void main(String[] args) {

        int[][] grid = prepareGrid();
        System.out.println(countMax(grid));
    }

    // Algorithm for counting max connected integers.
    private static int countMax(int[][] grid) {

        int numRows = grid.length;
        int numCols = grid[0].length;

        boolean[][] visited = new boolean[numRows][numCols];
        int count = 0;
        for (int i = 0; i < numRows; i++ ) {
            for (int j = 0; j < numCols; j++) {
                if (!visited[i][j]) {
                    int sum = BFS(grid, i, j, visited);
                    if (sum > count) {
                        count = sum;
                    }
                }
            }
        }
        return count;
    }

    // BFS algorithm, as showed before.
    private static int BFS(int[][] grid, int row, int col, boolean[][] visited) {

        int count = 1;
        final Cell cell = new Cell(row, col);
        final Queue<Cell> queue = new LinkedList<>();
        queue.offer(cell);
        while (!queue.isEmpty()) {
            final Cell c = queue.poll();
            visited[c.row][c.col] = true;
            for (Cell child : c.getChilds()) {
                if (!visited[child.row][child.col] && grid[child.row][child.col] == grid[c.row][c.col]) {
                    queue.offer(child);
                    count++;
                }
            }
        }
        return count;
    }

    // Helper method to create the grid.
    private static int[][] prepareGrid() {

        return new int[][]{
                {3, 2, 3, 3, 3},
                {2, 1, 3, 1, 3},
                {3, 1, 2, 1, 3},
                {2, 1, 1, 1, 3}
        };
    }
}
```

The output of this algorithm is, as expected, **7**.