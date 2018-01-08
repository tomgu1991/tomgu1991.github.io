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

