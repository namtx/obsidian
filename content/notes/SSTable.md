---
title: "SSTable"
---

#database 
[[Red-black tree]]
[[AVL Tree]]

When a write comes in, add it to an in-memory balanced tree data structure ([[memtable]])
When the [[memtable]] gets bigger than the thresh-hold, typically a few megabytes, write out it to disk as an [[SSTable]]
The new [[SSTable]] becomes the most recent segment of the database, while the [[SSTable]] is being written into disk, writes can continue to a new [[memtable]] instance. 

In order to serve a read request, first try to find the key in the [[memtable]], the most recent on-disk segment, then in the next older segment, etc. 

From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values. 
