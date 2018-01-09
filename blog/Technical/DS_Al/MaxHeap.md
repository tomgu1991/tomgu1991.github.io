# A heap implementation

Heap is a complete binary tree whose nodes are ordered in a certain manner. When a binary tree is complete, you can use an array to represent it in an efficient and elegant way.

```java
public interface MaxHeapInterface<T extends Comparable<? super T>> {
  void add(T newEntry);
  T removeMax();
  T getMax();
  boolean isEmpty();
  int getSize();
  void clear();
}
```

We begin by using an array to represent a complete binary tree. A complete tree is full to its next-to-last level, and its leaves on the last level are filled in from left to right. Thus, until we get to the last leaf, a complete tree has no holes.

Since the tree is complete, we can locate either the children or the parent of any node by performing a simple computation on the node’s number. This number is the same as the node’s corresponding array index. Thus, the children of the node i—if they exist—are stored in the array at indices 2i and 2i + 1. The parent of this node is at array index i/2, unless of course the node is the root. In that case, i/2 is 0, since the root is at index 1. To detect the root, we can watch for either this index or a special value—called a sentinel—that we place at index 0.

![max heap](maxheap_1.png)

### Add

compare with the parent

```java
Algorithm add(newEntry) 
if (the array heap is full)
	Double the size of the array
newIndex = index of next available array location
parentIndex = newIndex/2 //index of parent of available location
while (parentIndex > 0 and newEntry > heap[parentIndex])
{
heap[newIndex] = heap[parentIndex] //move parent to available location
// update indices
newIndex = parentIndex 
parentIndex = newIndex/2
}
heap[newIndex] = newEntry

Worst: O(log n)
```

### Delete

delete first, move last to first, reheap that is use child to replace the parent

```java
Algorithm remove
if not empty
	root = heap[1]
	heap[1] = heap[lastIndex]
	lastIndex--
  	reheap(1)
return root

Algorithm reheap(rootIndex)
// Transforms the semiheap rooted at rootIndex into a heap
done = false
orphan = heap[rootIndex]
while (!done and heap[rootIndex] has a child) {
largerChildIndex = index of the larger child of heap[rootIndex] 
if (orphan < heap[largerChildIndex])
{
heap[rootIndex] = heap[largerChildIndex]
rootIndex = largerChildIndex
}else
done = true
}
heap[rootIndex] = orphan
```

### Create heap

using add

### Heap sort

creat a heap

swap heap[1] and heap[lastIndex] and reheap

Average(O(nlogn))



[Implementation](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/heap)

