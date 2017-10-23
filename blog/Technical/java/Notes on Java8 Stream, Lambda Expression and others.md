# Notes on Java8 Stream, Lambda Expression and other interering features

Java9 已经来了，然而我的技能池还停留着Java7。平时的开发会用到Guava，一个google的jar包，里面大量实现了Java8中的函数式思想。比如lambda表达式等等。今天静下来，仔细的琢磨学习几个有用的API。

### Introduction to Lambda Expression

A lambda expression is composed of three parts. 

Argument List  + Arrow Token +  Body(The body can be either a single expression or a statement block.)

```java
(int x, int y) -> x + y
```

In the expression form, the body is simply evaluated and returned. In the block form, the body is evaluated like a method body and a return statement returns control to the caller of the anonymous method. The break and continue keywords are illegal at the top level, but are permitted within loops. If the body produces a result, every control path must return something or throw an exception.

### Functional Interface

- `Predicate`: A property of the object passed as argument
- `Consumer`: An action to be performed with the object passed as argument
- `Function`: Transform a T to a U
- `Supplier`: Provide an instance of a T (such as a factory)
- `UnaryOperator`: A unary operator from T -> T
- `BinaryOperator`: A binary operator from (T, T) -> T

##### Predicate

This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.

```java
boolean	test(T t)
	// Evaluates this predicate on the given argument.
  
Predicate<Integer> selected = p -> p >= 16;
System.out.print(selected.test(17));
// > True

Predicate<Person> drinkingAge = (it) -> it.getAge() >= 21;
    Predicate<Person> brown = (it) -> it.getLastName().equals("Brown");
    people.stream()
      .filter(drinkingAge.and(brown))
      .forEach((it) ->
                System.out.println("Have a beer, " +
                                   it.getFirstName()));
```

As might be expected, and(), or(), and xor() are all available. Make sure to check the Javadoc for a full introduction to all the possibilities.

##### Function

It has only one method `apply` with the following signature:

```java
public R apply(T t){ }
```

Basically, it will be used as a parameter to do a specific action in Type t and return the result.

```java
    // Define Western 
27     
28     Function<Person, String> westernStyle = p -> {
29       return "\nName: " + p.getGivenName() + " " + p.getSurName() + "\n" +
30              "Age: " + p.getAge() + "  " + "Gender: " + p.getGender() + "\n" +
31              "EMail: " + p.getEmail() + "\n" + 
32              "Phone: " + p.getPhone() + "\n" +
33              "Address: " + p.getAddress();
34     };

    // Print Western List
44     System.out.println("\n===Western List===");
45     for (Person person:list1){
46         System.out.println(
47             person.printCustom(westernStyle)
48         );
49     }

  public String printCustom(Function <Person, String> f){
124       return f.apply(this);
125   }
```

### Stream

A `Stream` represents a sequence of elements on which various methods can be chained. By default, once elements are consumed they are no longer available from the stream. Therefore, a chain of operations can occur only once on a particular `Stream`. In addition, a `Stream` can be serial(default) or parallel depending on the method called. 

它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。获取一个数据源（source）→ 数据转换→执行操作获取想要的结果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道。

流的操作类型分为两种：

- **Intermediate**：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- **Terminal**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。
- **short-circuiting**: 
  - 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。
  - 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

在对于一个 Stream 进行多次转换操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。

```java
18     // Make a new list after filtering.
19     List<Person> pilotList = pl
20             .stream()
21             .filter(search.getCriteria("allPilots"))
22             .collect(Collectors.toList());

int sum = widgets.stream()
.filter(w -> w.getColor() == RED)
 .mapToInt(w -> w.getWeight())
 .sum();

```

Stream操作

* construct

  ```java
  // 1. Individual values
  Stream stream = Stream.of("a", "b", "c");
  // 2. Arrays
  String [] strArray = new String[] {"a", "b", "c"};
  stream = Stream.of(strArray);
  stream = Arrays.stream(strArray);
  // 3. Collections
  List<String> list = Arrays.asList(strArray);
  stream = list.stream();
  ```

* collect

  ```java
  // 1. Array
  String[] strArray1 = stream.toArray(String[]::new);
  // 2. Collection
  List<String> list1 = stream.collect(Collectors.toList());
  List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));
  Set set1 = stream.collect(Collectors.toSet());
  Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));
  // 3. String
  String str = stream.collect(Collectors.joining()).toString();
  ```

* Intermediate

  ```java
  map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

  Stream.of("one", "two", "three", "four")
   .filter(e -> e.length() > 3)
   .peek(e -> System.out.println("Filtered value: " + e))
   .map(String::toUpperCase)
   .peek(e -> System.out.println("Mapped value: " + e))
   .collect(Collectors.toList());

  List<String> personList2 = persons.stream().
  map(Person::getName).limit(10).skip(3).collect(Collectors.toList());
  ```

* Terminal

  ```
  forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator
  ```

* Short-circuiting

  ```
  anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit
  ```

  ​

### API Examples

1. Map

   ```java
   // i -> i * 2 是lambda表达式，参数是i，将i作用*2操作。所以map函数将这个lambda表达式逐一应用于前边的的集合元素
   	int[] ia = range(1, 10).map(i -> i * 2).toArray(); 
   // The mutable reduction operation is called collect(), as it collects together the desired results into a result container such as a Collection.
       List<Integer> results = range(1, 10).map(i -> i * 2).boxed().collect(toList());
       out.println("The result is: " + Arrays.toString(ia));
       out.println("The result is: " + results.toString());
   ```

   Result is:

   ```
   The result is: [2, 4, 6, 8, 10, 12, 14, 16, 18]
   The result is: [2, 4, 6, 8, 10, 12, 14, 16, 18]
   ```

   ​

2. Reduce

   ```java
       List<Integer> list = new ArrayList();
       list.add(1);
       list.add(4);
       list.add(7);
   // reduce 操作将初始值$0和第一个元素按照$1的函数操作，之后迭代$n 与 $n+1
       int result1 = list.stream().reduce(0, Integer::sum);
       // 0+1 * 1 -> 1+1 * 4 -> 8 + 1 * 7 = 63
       int result2 = list.stream().reduce(0, (a, b) -> (a + 1) * b);
       out.println("The result is: " + result1);
       out.println("The result is: " + result2);
   ```

   Result is:

   ```
   The result is: 12
   The result is: 63
   ```

   ​

3. Match

   ```java
       final List<String> keywords = Arrays.asList("brown", "fox", "dog", "pangram");
       final List<String> keyword2 = Arrays.asList("pangramx");
       final String tweet = "The quick brown fox jumps over a lazy dog. "
           + "#pangram http://www.rinkworks.com/words/pangrams.shtml";

       boolean result1 = keywords.stream().anyMatch(tweet::contains);
       // init value, lambda expression where param -> function -> reduce result, combiner
   // Performs a reduction on the elements of this stream, using the provided identity, accumulation and combining functions.
       boolean result2 = keyword2.stream().reduce(false,
           (b, keyword) -> b || tweet.contains(keyword), (l, r) -> l || r);
       out.println("Using string search:");
       out.println("The result is: " + result1);
       out.println("The result is: " + result2);
   ```

   Result is:

   ```
   The result is: true
   The result is: false
   ```

4. Building String

   ```java
       // 1. out 2. return 3. out then the next i
       range(1, 5).boxed().map(
           i -> {
             out.print("Happy Birthday " + i);
             if (i == 3) {
               return "dear NAME" + i;
             } else {
               return "to You" + i;
             }
           }
       ).forEach(new Consumer<String>() {
         @Override
         public void accept(String s) {
           out.println(s);
         }
       });
   ```

   Result is:

   ```
   Happy Birthday 1to You1
   Happy Birthday 2to You2
   Happy Birthday 3dear NAME3
   Happy Birthday 4to You4
   ```

5. group

   ```java
       Map<String, List<Integer>> result;
       // Returns a Collector implementing a "group by" operation on input elements of type T,
       // grouping elements according to a classification function, and returning the results in a Map.
       result = Stream.of(49, 58, 60, 76, 82, 88, 90)
           .collect(groupingBy(new Function<Integer, String>() {
             @Override
             public String apply(Integer i) {
               if (i > 60) {
                 return "large";
               } else if (i == 60) {
                 return "equal";
               }
               return "No";
             }
           }));
       out.println(result);
   ```

   Result is:

   ```
   {equal=[60], No=[49, 58], large=[76, 82, 88, 90]}
   ```

   ​

6. filter

   ```java
   // Returns a stream consisting of the elements of this stream that match the given predicate.    
   	List<Integer> result = Stream.of(49, 58, 60, 76, 82, 88, 90).filter(i -> i > 60).
           map(i -> i * 2).collect(toList());
       out.println(result);
   ```

   Result is:

   ```
   [152, 164, 176, 180]
   ```

   ​

7. sort

   ```java
   public static final Comparator<Person> BY_LAST_AND_AGE =
       (lhs, rhs) -> {
         if (lhs.lastName.equals(rhs.lastName))
           return lhs.age - rhs.age;
         else
           return lhs.lastName.compareTo(rhs.lastName);
       };
   Collections.sort(people, Person.BY_LAST_AND_AGE); 

     public static int compareLastAndAge(Person lhs, Person rhs) {
       if (lhs.lastName.equals(rhs.lastName))
         return lhs.age - rhs.age;
       else
         return lhs.lastName.compareTo(rhs.lastName);
     }
   Collections.sort(people, Person::compareLastAndAge); 

   List<Person> personList2 = persons.stream().limit(2).sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).collect(Collectors.toList());

   ```




8. compare

   ```
   public static final Comparator<Person> BY_FIRST =
       (lhs, rhs) -> lhs.firstName.compareTo(rhs.firstName);

   public static final Comparator<Person> BY_FIRST =
       Comparators.comparing(Person::getFirstName);
       
   Collections.sort(people,
         Comparators.comparing(Person::getLastName)
                    .thenComparing(Person::getAge)); 
   ```

   ​

Ref: 

[10 Java One Liners to Impress Your Friends](https://github.com/aruld/java-oneliners/wiki)

[Java SE 8: Lambda Quick Start](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html)

[Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

