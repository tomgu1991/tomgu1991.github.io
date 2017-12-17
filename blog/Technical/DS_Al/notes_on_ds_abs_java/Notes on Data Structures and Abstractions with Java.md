# Notes on Data Structures and Abstractions with Java

> Frank M. Carrano 

[TOC]

### Operator Precedence

When operators have equal precedence, binary operators execute in left-to-right order, and unary operators execute in right-to-left order.

![](image/1.png)

### Introduction

1. ADT: a specification that describes a data set and the operations on that data.
2. Data structure: is an implementation of an ADT within a programming language.
3. Collection: is a general term for an ADT that contains a group of objects.
4. Container: is a class that implements a collection.
5. Bag: consists of an unordered collection that allows duplicates.
6. List: ordered items in a variety of ways
7. Stack|Queue: order their items chronologically
8. Dictionary: pairs of items
9. Tree: organized its entries according to some hierarchy
10. Graph: generalization of the ADT tree that focuses on the relationship among its entries instead of hierarchical organization.



### Bag

A bag is a finite collection of objects in no particular order. A bag can contain duplicate items.

Since a bag is an abstract data type, **we only describe its data and specify its operations. We do not indicate how to store the data or how to implement its operations.** Don’t think about arrays, for example. You first need to clearly know what the bag operations do: Focus on what the operations do, not on how they do them. That is, you need a detailed set of specifications before you can use a bag in a program. In fact, you should specify the bag operations before you even decide on a programming language.

![](image/2.png)

**Writing Java statements that test a class’s methods will help you to fully understand the spec- ifications for the methods.** Obviously, you must understand a method before you can imple- ment it correctly. If you are also the class designer, your use of the class might help you see desirable changes to your design or its documentation. You will save time if you make these revisions before you have implemented the class. Since you must write a program that tests your implementation sometime, why not get additional benefits from the task by writing it now instead of later?

**Usining an ADT is like using a vending machine**. When you look at the front of a vending machine, you see its interface. By inserting coins and pressing buttons, you are able to make a purchase. Here are some observations that we can make about the vending machine:
● You can perform only the specific tasks that the machine’s interface presents to you.
● You must understand these tasks—that is, you must know what to do to buy a soda.
● You cannot access the inside of the machine, because a locked shell encapsulates it.
● You can use the machine even though you do not know what happens inside.
● If someone replaced the machine’s inner mechanism with an improved version, leaving the interface unchanged, you could still use the machine in the same way.



### Bag implementation use arrays

The definition for the class ArrayBag could be fairly involved. The class certainly will have quite a few methods. For such classes, you should not define the entire class and then attempt to test it. **Instead, you should identify a group of core methods to both implement and test before continu- ing with the rest of the class definition.** By leaving the definitions of the other methods for later, you can focus your attention and simplify your task. But what methods should be part of this group? In general, such methods should be central to the purpose of the class and allow reasonable testing. We sometimes will call a group of core methods a core group.

**The core methods**
● Constructors
● public boolean add(T newEntry) ● public T[] toArray()
● public boolean isFull()

![](image/3.png)

```
// the cast is safe because the new array contains null entries
@SuppressWarnings("unchecked")
T[] tempBag = (T[])new Object[capacity]; // unchecked cast bag = tempBag;

To suppress an unchecked-cast warning from the compiler, you precede the flagged state- ments with the instruction
    @SuppressWarnings("unchecked")
Note that this instruction can precede only a method definition or a variable declaration.
```

An incomplete definition of a method is called a **stub**. The stub needs only to keep the syntax checker happy.

Remember that we are not required to maintain any particular order for a bag’s entries. So instead of shifting array entries after removing an entry, we can replace the entry being removed with the last entry in the array。

When a classroom is full, one way to accommodate additional students is to move to a larger room. In a similar manner, when an array becomes full, you can move its contents to a larger array. This process is called **resizing** an array.

Doubling the size of an array each time it becomes full is a typical approach.

#### Outline

```java
public class ArrayBag<T> implements BagInterface<T>{
  private T[] bag;
  private static final int DEFAUTL_CAPACITY = 25;
  private int numberOfEntries;
  
  public ArrayBag(){this(DEFAULT)}
  
  public ArrayBag(int cap){
    numberOfEntries = 0;
    @SuppressWarnings("unchecked")
    T[] tem = (T[])new Object[cap];
    bag = tem;
  }
  
  // add:
  	// check full
  	// bag[numberOfEntries] = T; num++ 
  // isFull
  	// num == bag.length
  // toArray
  	// new Object
  	// forEach assign
  // isEmpty
  	// num is 0
  // getCurrentSize()
  	// return num
  // getFrequencyOf(T)
  	// ForEach count
  // contains
  	// while(not found and has more)
  		// if equals -> return,break
  // clear
  	// not empty() -> remove()
  // remove
  	// bag[num-1] = null; num--;
  // remove(T)
  	// getIndex
  	// remove index
  // removeEntry(index)
  	// num--; bag[index] = bug[num];bug[num]=null; // change index and the last entry
  // getIndex
  	// forEach check
}
```



### Bag implementation use linked data

**Address**: each element has a number identify itself

Reference: when an element has the address of the other element, we say this element **references** that and they are **linked**.

Chain: all these linked elements form a **chain**.

Therefore, the linked data has to keep a head of the bag, which is usually the most recently added element.

#### Outline

```java
public class LinkedBag<T> implements BagInterface<T> {
  private Node firstNode; // address of the most recently element
  private int num;
  
  private class Node{
    T data;
    Node next;
  }
  // begin
  	// newNode references a new instance of Node, place data in newNode，num++
  	// firstNode = address of newNode
  // add
  	// newNode references a new instance of Node, place data in newNode, link to firstNode
  	// firstNode = address of newNode,num++
  // toArray | getFrequencyOf | contains
  	// while -> from firstNode
  // remove
  	// firstNode = firstNode.next, num--
  // remove(T)
  	// locate node N, replace with firstNode, remove()
  // 
}
```



### The efficiency of algorithms

**Complexity** : an algorithm has both time and space requirements. The process of measuring the complexity of algorithms is called analysis of algorithms. Your measure of the complexity of an algorithm should be easy to compute, certainly easier than implementing the algorithm. You should express this measure in terms of the size of the prob- lem. This problem size is the number of items that an algorithm processes. The value of the function is said to be directly proportional to the time requirement. Such a function is called a growth-rate function because it measures how an algo- rithm’s time requirement grows as the problem size grows.

An algorithm’s **basic operation** is the most significant contributor to its total time requirement. 

![](image/4.png)

Algorithm A has a time requirement proportional to n, we say that A is O(n). We call this notation Big Oh since it uses the capital letter O. We read O(n) as either “Big Oh of n” or “order of at most n.”

In simple terms, f(n) is O(g(n)) means that c x g(n) provides an upper bound on f(n)’s growth rate when n is large enough. For all data sets of a sufficient size, the algorithm will always require fewer than c x g(n) basic operations.

### Stack

The ADT stack organizes its entries according to the order in which they were added. All additions are to one end of the stack called the top. The top entry—that is, the entry at the top—is thus the newest item among the items currently in a stack.

#### Outline

```java
public interface StackInterface<T> {
  void push(T);
  T pop();
  T peek();
  boolean isEmpty();
  void clear();
}
```

#### Check balanced delimiters

![](image/5.png)

#### Infix to postfix

Notice that the **order of the operands a, b, and c** in an infix expression is the same in the corre- sponding postfix expression. **One way to determine where the operators should appear in a postfixexpression begins with a fully parenthesized infix expression.** For example, we write the infix expres-sion (a + b) * c as ((a + b) * c). By adding parentheses, we remove the expression’s dependence on therules of operator precedence. Each operator is now associated with a pair of parentheses. We nowmove each operator to the right so that it appears immediately before its associated close parenthesisto get ((a b +) c *). Finally, we remove the parentheses to obtain the postfix expression a b + c *.

When we encounter an operand, we place it at the end of the new expression that we are creating. When we encounter an operator, we must save it until we determine where in the output expression it belongs. We saved the operator until we processed its second operand. In general, wehold the operator in a stack at least until we compare its precedence with that of the next operator. We need to distinguish between operators that have a left-to-right association—namely +, -, *, and /—and exponentiation, which has a right-to-left association.

Parentheses override the rules of operator precedence. We always push an open parenthesis onto the stack. Once it is in the stack, we treat an open parenthesis as an operator with the lowest precedence. That is, any subsequent operator will get pushed onto the stack. When we encounter a close parenthesis, we pop operators from the stack and append them to the forming postfix expression until we pop an open parenthesis. The algorithm continues with no parentheses added to the postfix expression.

![](image/7.png)

![](image/6.png)

#### Evaluate Postfix

![](image/8.png)



#### Evaluate Infix

![](image/9.png)

### Stack Implementations

#### Linked

```
public class LinkedStack<T> implements StackInterface<T>{
  private Node topNode;
  // constructor
  // push(T)
  	// newNode; topNode = newNode
  // T peek()
  	// topNode#data
  // T pop()
  	// tem = peek();topNode = topNode#next
}
```





