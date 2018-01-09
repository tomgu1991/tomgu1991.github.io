# Searching

### Searching an Unsorted Array

```java
public boolean contains(T anEntry) {
  boolean found = false;
  for(int index = 0; !found && (index < numberOfEntries); index++) {
    if(anEntry.equals(list[index])) {
      found = true;
    }
  }
  return found;
}
Best: O(1)
Average: O(n)
Worst: O(n)
```



### Searching a Sorted Array

```
Algorithm to search a[0] through a[n-1] for desiredItem
	mid = approximate midpoint between 0 and n-1
	if desiredItem equals a[mid]
		return true
	else if desiredItem < a[mid]
		return the result of searching a[0] through a[mid-1]
	else
		return the result of searching a[mid+1] through a[n-1]
		
Arrays#binarySearch(T[] array, T desiredItem)

Best: O(1)
Average: O(log n)
Worst: O(log n)
```



### Searching an Unsorted Chain

```java
private boolean search(Node currentNode, T desiredItem) {
  boolean found;
  if(currentNode == null)
  	found = false;
  else if(desiredItem.equals(currentNode.getData()))
    found = true;
  else
    found = search(currentNode.getNextNode(), desiredItem)
  return found;
}
Best: O(1)
Average: O(n)
Worst: O(n)
```



### Searching of a Sorted Chain

```java
public boolean contains(T anEntry) {
  Node currentNode = firstNode;
  while((current != null) && anEntry > currentNode.getData())
  	currentNode = currentData.getNextNode();
  return currentNode != null && anEntry==currentNode.getDate();
}

Best: O(1)
Average: O(n)
Worst: O(n)
```



### Binary Search Tree

A binary search tree is a binary tree whose nodes contain `Comparable` objects and are organized as follows. For each node in the tree: 1. The data in a node is greater than the data in the node's left subtree 2. The data in a node is less than the data in the node's right subtree.

```java
public interface TreeInterface<T> {
  T getRootData();

  int getHeight();

  int getNumberOfNodes();

  boolean isEmpty();

  void clear();
}

public interface BinaryTreeInterface<T> extends TreeInterface<T>, TreeIteratorInterface {
  public void setTree(T rootData);
  public void setTree(T rootData, BinaryTreeInterface<T> leftTree, BinaryTreeInterface<T> rightTree);
}

public interface SearchTreeInterface<T extends Comparable<T>> extends TreeInterface<T> {
  
  boolean contains(T entry);
  
  T getEntry(T entry);
  
  T add(T newEntry);
  
  T remove(T entry);
  
  Iterator<T> getInorderIterator();
}

```

```java
AlgorithmbstSearch(binarySearchTree, desiredObject) 
  // Searches a binary search tree for a given object.
  // Returns true if the object is found.
if (binarySearchTree is empty) return false
else if (desiredObject == objectintherootofbinarySearchTree) return true
else if (desiredObject < objectintherootofbinarySearchTree) return bstSearch(leftsubtreeofbinarySearchTree, desiredObject)
else
return bstSearch(rightsubtreeofbinarySearchTree, desiredObject)
```

```java
Algorithmadd(binarySearchTree, newEntry)
// Adds a new entry to a binary search tree.
// Returns null if newEntry did not exist already in the tree. Otherwise, returns the // tree entry that matched and was replaced by newEntry.
result = null
if (binarySearchTree is empty)
Create a node containing newEntry and make it the root of binarySearchTree else
result = addEntry(binarySearchTree, newEntry) return result;

AlgorithmaddEntry(binarySearchTree, newEntry)
// Adds a new entry to a binary search tree that is not empty.
// Returns null if newEntry did not exist already in the tree. Otherwise, returns the 
  // tree entry that matched and was replaced by newEntry.
result = null
if (newEntry matches the entry in the root of binarySearchTree) {
result = entryintheroot
Replace entry in the root with newEntry }
else if (newEntry < entry in the root of binarySearchTree) {
if (the root of binarySearchTree has a leftchild)
result = addEntry(leftsubtreeofbinarySearchTree, newEntry)
else
Give the root a left child containing newEntry }
else // newEntry > entryintherootofbinarySearchTree {
if (the root of binarySearchTree has a rightchild)
result = addEntry(rightsubtreeofbinarySearchTree, newEntry)
else
Give the root a right child containing newEntry return result

```

```java
Algorithmremove(binarySearchTree, entry) 
oldEntry = null
if (binarySearchTree is not empty)
{
if (entry matches the entry in the root of binarySearchTree) {
oldEntry = entry in root
removeFromRoot(root of binarySearchTree) }
else if (entry < entryinroot)
oldEntry = remove(leftsubtreeofbinarySearchTree, entry)
else // entry > entry in root
oldEntry = remove(rightsubtreeofbinarySearchTree, entry)
}
return oldEntry

Algorithm removeFromRoot(rootNode)
// Removes the entry in a given root node of a subtree.
if (rootNode has two children) {
largestNode = node with the largest entry in the left subtree of rootNode Replace the entry in rootNode with the entry in largestNode
Remove largestNode from the tree // that is removeLargest(leftChild)'s result as leftChild
}
else if (rootNode has a rightchild)
rootNode = rootNode’s rightchild 
else
rootNode = rootNode’s left child // possibly null // Assertion: if rootNode was a leaf, it is now null
return rootNode

findLargest(root)
  if hasRight -> findLargest(root.getRight)
   return root
   
// return the new root of the result of remove largest of the tree of rootNode
removeLargest(rootNode)
  if rootNode.hasRight
  	rightChild = rootNode.getRight
  	root = removeLargest(rightChild)
    rootNode.setRight(root)
   else
     rootNode = rootNode.getLeftChild()
   return rootNode 
```

[Implementation](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/search/tree)

