---
title: "Union Find"
---

 Given the below figure shows list of vertices and edges connected between them, how can quickly check if vertices[0] and vertices[3] are connected?
 
![[notes/images/Pasted image 20221222000955.png]]

We can do so by using the #union-find data structure, it is also known as #disjoint-set data structure.

The primary use of #disjoint-set is to address the connectivity between the components of a network. The _network_ here can be a computer network or a social network. For instance, we can use #disjoint-set to determine if two people share a common ancestor.

### Implementing Union Find

A Union Find supports two basic method `find` and `union` methods.

- `find(x: Int): Int` Allows determining which set that item `x` belongs to.
- `union(x: Int, y: Int)` Merge two sets, where item `x` and item `y` belongs to.

```kotlin
interface UnionFind {  
    fun find(x: Int): Int  
    fun union(x: Int, y: Int)  
}
```


### Quick Find 

```kotlin
class QuickUnionFind(size: Int): UnionFind(size) {  
    override fun find(x: Int): Int {  
        return parent[x]  
    }  
  
    override fun union(x: Int, y: Int) {  
        val setX = find(x)  
        val setY = find(y)  
        if (setX != setY) {  
            for (i in parent.indices) {  
                if (parent[i] == setY) {  
                    parent[i] = setX  
                }  
            }  
        }  
    }  
}
```

##### Time Complexity
- Constructor: **O(n)**
- Find: **O(1)**
- Union: **O(n)**


### Quick Union

```kotlin
class QuickUnion(size: Int) : UnionFind {  
    private val parent: IntArray = IntArray(size)  
  
    init {  
        for (i in 0 until size) {  
            parent[i] = i  
        }  
    }  
  
    override fun find(x: Int): Int {  
        if (parent[x] == x) return x  
  
        return find(parent[x])  
    }  
  
    override fun union(x: Int, y: Int) {  
        val setX = find(x)  
        val setY = find(y)  
  
        if (setX != setY) {  
            parent[setX] = setY  
        }  
    }  
}
```

##### Time Complexity
- Constructor: **O(n)**
- find: **O(n)**
- union: **O(n)**

##### Why Quick Union is more efficient than Quick Union?
The `union` operation consists of two `find` operations which (only in the worst case) will take **O(N)** time, and two constant time operations, including the equality check and updating the array value at the given index. Therefore, the `union` operation also cost **O(N)** in the worst case.


### Union By Rank

```kotlin
class UnionByRank(size: Int) : UnionFind {  
    private val parent = IntArray(size)  
    private val rank = IntArray(size)  
  
    init {  
        for (i in 0 until size) {  
            parent[i] = i  
        }  
    }  
  
    override fun find(x: Int): Int {  
	    var currentX = x;  
	    while (currentX != parent[currentX]) {  
	        currentX = parent[currentX]  
	    }  
	    return currentX  
	}
  
    override fun union(x: Int, y: Int) {  
        val setX = find(x)  
        val setY = find(y)  
  
        if (rank[setX] > rank[setY]) {  
            parent[setY] = parent[setX]  
        } else if (rank[setX] < rank[setY]) {  
            parent[setX] = parent[setY]  
        } else {  
            parent[setY] = parent[setX]  
            rank[setX]++  
        }  
    }  
}
```

##### Time Complexity

- Constructor: **O(n)**
- find: **O(log n)**
- union: **O(log n)**

### The Path Compression Optimization

