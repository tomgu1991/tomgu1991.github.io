# Sorting

Arranging things into either ascending or descending order is called **sorting**. You can sort any collection of items that can be compared with one another. Exactly how you compare two objects depends on the nature of the objects.

### Selection Sorting

**Heuristic**

Always select the smallest one in the rest of unordered the part.

**Pseudocode**

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

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/selection)

### Insertion Sorting

**Heuristic**

Put the first one. The the second compare to the first one. The rest of unordered to compare with all the element that sorted.

An insertion sort of an array **partitions**—that is, divides—the array into two parts. One part is sorted and initially contains just the first entry in the array. The second part contains the remaining entries. The algorithm removes the first entry from the unsorted part and inserts it into its proper sorted position within the sorted part.

**Pseudocode**

```
Algorithm insertionSort(a, first, last)
// Sorts the first n entries of an array a

for unsorted = first+1 to last
	nextToInsert = a[unsorted]
	insert(nextToInsert, a, first, unsorted-1)
	// insert into the sorted part which is from first to unsorted-1
```

```
Algorithm insert(entry, a, begin, end)
// insert entry into the sorted entries a[begin] to a[end]
index = end
while(index >= begin && entry < a[index])
	a[index+1] = a[index] // make room that move
	index--
a[index+1] = entry // insert
```

**Efficiency**

```
for -> n
insert -> from begin to end, O(n^2)

Best: n
Averate: n^2
Worst: n^2
```

**Implementation**

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/insertion)

### Shell Sorting

**Heuristic**

Donald Shell devised in 1959 an improved insertion sort, now called the Shell sort. Shell wanted entries to move beyond their adjacent locations. To do so, he sorted subarrays of entries at equally spaced indices. Instead of moving to an adjacent loca- tion, an entry moves several locations away. The result is an array that is almost sorted—one that can be sorted efficiently by using an ordinary insertion sort.

**Pseudocode**

```
Algorithm incrementalInsertionSort(a, first, last, space)
// sort a from first to last by space 
for unsorted = first+space to last by space
	nextToInsert = a[unsorted]
	index = unsorted - space
	while( index >= first && nextToInsert<a[index])
		a[index+space] = a[index] // move
	a[index+space] = nextToInsert
```

```
Algorithm shellSort(a, first, last)
// sort a from first to last
space = n/2
while(space > 0)
	for begin=first to first+space-1
		// sort subarray
		incrementalInsertionSort(a, first, last, space)
	space /= 2
```

**Efficiency**

```
Best:n
Average:n^1.5
Worst:n^1.5
```

**Implementation**

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/shell)

### Merge Sorting

**Heuristic**

The merge sort divides an array into halves, sorts the two halves, and then merges them into one sorted array. The algorithm for merge sort is usually stated recursively.  The strategy is said to be a **divide and conquer**

**Pseudocode**

```
Algorithm mergeSort(a, temArray, first, last)
// sort a from first to last, save into temArray
if first<last
	mid = first+last / 2
	mergeSort(a, temArray, first, mid)
	mergeSort(a, temArray, mid+1, last)
	merge(a, temArray, first, mid, last) // merge into one
```

```
Algorithm merge(a, temArray, first, mid, last)
// merge two sub_array into temArray
bF1 = first, bF2 = mid+1
eF1 = mid, eF2 = last
index=0
while(bF1<=eF1 && bF2<=eF2){
  // merge
  if a[bF1] <= a[bF2]
  	temArray[index]=a[bF1]
  	bF1++
  else
  	temArray[index]=a[bF2]
  	bF2++
  index++
}
if(bF1<eF1) copy left
if(bF2<eF2) copy left
copy from tempArray to a
```

**Efficiency**

```
Best:nlogn
Average:nlogn
Worst:nlogn
```

**Implementation**

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/merge)

### Quick Sort

**Heuristic**

The quick sort divides an array into two pieces, but unlike merge sort, these pieces are not necessarily halves of the array. Instead, quick sort chooses one entry in the array—called the pivot—and rearranges the array entries so that
• The pivot is in the position that it will occupy in the final sorted array 
• Entries in positions before the pivot are less than or equal to the pivot 
• Entries in positions after the pivot are greater than or equal to the pivot

**Pseudocode**

```
Algorithm quickSort(a, first, last)
// sort a from first to last, save into temArray
if first<last
	choose a pivot
	Partition the array about the pivot
	pivotIndex = index of pivot
	quickSort(a, first, pivotIndex-1)
	quickSort(a, pivotIndex+1, last)
```

```
Algorithm partition(a, first, last)
T pivot = a[last]
int left=first
int right=last-1
boolean done=false
while(!done)
	// search swap position
	while(a[left] < pivot)
		left++
	while(a[right] > pivot)
		right--
	if(left < right)
		swap
		left++
		right--
	else
		done=true
swap(left, last)
return left
```

**Efficiency**

```
Best:nlogn
Average:nlogn
Worst:n^2
```

**Implementation**

[here](https://github.com/tomgu1991/Interview_pre/tree/master/source/helloworld/src/com/tomgu/algorithm/sorting/quick)

### 