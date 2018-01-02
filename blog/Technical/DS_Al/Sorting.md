# Sorting

Arranging things into either ascending or descending order is called **sorting**. You can sort any collection of items that can be compared with one another. Exactly how you compare two objects depends on the nature of the objects.

### Selection Sorting

**pseudocode**

```
Algorithm selectionSort(a, n)
// Sorts the first n entries of an array a

for i = 0 to n-1
	indexOfNextSmallest = the index of the smallest among 
							a[index], a[index+1]...a[n-1]
	InterChange the value of a[index] and a[indexOfNextSmallest]
```

**Efficiency**

```
for -> n-1
indexOfNextSmallest -> O(n)

Summary: n^2 comparison, n swaps
Total:O(n^2)
```

**Implementation**

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/seletion)





