# Tsmart Unity 参考

> @author Zuxing Gu
>
> 这将是一份旷日持久的文档。。。

### 前期准备

1. 配置环境：Ubuntu 16.04，llvm 3.9.0([源代码编译llvm](http://tomgu1991.github.io/blog/Technical/LLVM/LLVM_Introduction.html))，Gitlab添加权限、下载项目源[代码](http://sts.thss.tsinghua.edu.cn:5080/)
2. 理解这篇文章: [CAV 2007 CPA](https://www.semanticscholar.org/paper/Configurable-Software-Verification-Concretizing-Beyer-Henzinger/1929ed09538c988552ae92435c54509370bcbd4d) (CPA 算法原理)
3. 理解这篇文章: [CAV 2011 CPAChecker](https://www.semanticscholar.org/paper/CPAchecker-A-Tool-for-Configurable-Software-Beyer-Keremoglu/c84086d352be8b87b2266aeb82540602b20ec0c2) (我们工具参考框架结构)



### 代码结构

我们的代码是由**陈光**大神基于Gradle搭建，理论上这是一个Gradle项目。具体Gradle相关，参看其他参考部分。我们用的IDE是[IntelliJ IDEA](https://www.jetbrains.com/idea/?fromMenu) ，这个有教育版，学生可以免费试用。

我们有三个模块
* Checker 负责顶层算法和分析流程控制
* Engine 负责底层解析ll文件和构造CFA
* Mod-commons 负责基础通用的组建，比如Configuration, Logger, Shutdown, Annotations, IO and so on.




##### Mod-commons 负责基础通用的组建 Prefix: cn.edu.thu.sse.common

* annotations
* collect
* concurrency
* configuration
* io
* log
* time
* util
* TODO




##### Engine 负责底层解析ll文件和构造CFA Prefix: cn.edu.thu.tsmart.core.cfa

* llvm 
* model
* util




##### Checker 负责顶层算法和分析流程控制 Prefix: cn.edu.thu.tsmart.checker

* cmdline 处理command line执行时的参数信息，通常我们返回一个Properties Map
* core 分析的核心算法
  - algorithm 算法相关类（算法构造器，算法执行状态，某个算法的构造器、算法执行策略、算法执行中的统计信息）
  - defaults 所有默认的类（默认的CPA里面的元素，Domain，Stop，Merge，PrecisionAdjustment）
  - interfaces 所有的接口，注意，具体CPA的实现在cpa包中，但是cpa的接口放在这里
  - phase 我们的分析过程被划分为不同的phase，这里面负责将phase初始化、创建依赖关系、执行、统合结果
  - config 构造phase
  - reachedset 实现各种各样的数据结构用来管理分析过程中的状态集合，保存已经分析过的状态、获取下一个状态
  - waitlist 状态集合中的待分析集合，在这里我们通常设置分析策略，比如DFS、BFS
  - witness TODO
  - Other:
    * CoreComponentsFactory.java 构造CPA，algorithm，reachedSet
    * TsmartCPAChecker.java 分析的主入口，通过TsmartCPAPhaseManger管理执行过程
    * TsmartCPACheckerResult.java 记录分析的结果
    * TsmartCPAPhaseManager.java 负责从配置文件组装phase，配置依赖关系，根据执行策略执行phase
    * TsmartMainCPAStatistics.java 负责记录分析的统计信息
* cpa 所有具体的cpa(通常：需要提供构造器factory、domain、state、transferRelation、stop、merge、precision)
  - arg 最顶层的容器cpa，只能有一个孩子，通常是CompositeCPA
  - composite 顶层容器cpa，通常包含多个独立的cpa
  - location 最基本的cpa，记录每个CFANode的信息，即每个原子语句之间顺序关系 Node -> Edge(statement) -> Node
  - TODO
  - Other:
    * CPABuilder.java 通过配置文件构造cpa，基于反射，调用cpa中的factory方法
* exception 所有checker相关的异常
* util 工具类
  * error 负责TsmartCheckerMain中所有错误信息，这种信息通畅直接导致程序崩溃
  * globalinfo 全局数据结构
  * helper 辅助包，为相应接口对象提供辅助函数，比如对于AbstractState、CPA
* Other:
  - TsmartCheckerMain.java 程序主入口，负责解析参数，用得到的参数执行TsmartCPAChecker
  - CONSTANTS.java 所有常量




### Checker执行顺序（一个从主函数入口开始的顺序关系）

总体来说我们的分析工具有三个步骤

1. 写配置
2. 跑分析
   * 解析命令行参数，初始化配置
   * 创建各种phase，并设置依赖关系
   * 根据phase顺序，依照配置（目前只有串行分析）进行分析。通常是：构造cfa，分析
3. 看结果

##### 第一步：写配置

我们的配置文件一共有三类：top configuration，phase config，concreate properties

* default.properties 全局的配置信息，以及下一级的配置信息

   ```shell
   # 指定phase的配置文件在哪里
   phase.manager.config = config/phase/default/default.config
   # 指定分析策略，我们目前只有这一个策略
   phase.manager.executionType = SEQUENTIAL
   # TODO
   phase.result.outputMode = XML
   # TODO
   output.disable = true
   # 指定待分析程序
   input.programs = config/phase/default/default.ll
   ```

* default.config 所有和phase有关的配置信息，以及下一级的配置信息

  ```shell
  # 设置基本路径，因为我们的程序会用到类加载机制
  .SET
  DIR = cn.edu.thu.tsmart.checker.core.phase
  .TES
  # 创建的第一个phase，构造cfa
  .DEF cfa # 名字是cfa
  .TYPE = $(DIR).CFACreatePhase # phase的类型 -> 会对应一个具体的phase类来加载
  analysis.summaryEdges = true # 这个phase用的配置项
  .FED
  # 创建的第二个phase，分析
  .DEF basic # 名字是basic
  .TYPE = $(DIR).BasicAnalysisPhase # 用BasicAnalysisPhase加载
  # 指定具体cpa配置文件的位置
  phase.singleAlgorithm.analysis = config/phase/default/uninitialized.properties 
  phase.singleAlgorithm.stopAfterError = false # TODO
  phase.singleAlgorithm.initialStatesFor = ENTRY # 指定分析的入口，用来获得初始分析的位置
  .FED
  # 设置依赖关系
  .RUN
  basic : cfa;
  ```

* uninitialized.properties 具体cpa的配置文件

  ```shell
  # 这部分通常只替换CompositeCPA.cpas后面的内容和后面具体的参数信息
  # top cpa
  cpa = cpa.arg.ARGCPA
  # *.cpa child of *
  ARGCPA.cpa = cpa.composite.CompositeCPA
  # *.cpas children of *
  CompositeCPA.cpas = cpa.location.LocationCPA, cpa.uninitvars.UninitializedVariablesCPA

  analysis.traversal.order = bfs

  ```

##### 第二步：跑checker

目前，主入口是***TsmartCheckerMain***，需要提供的参数是***Top configuration***。运行我们的程序，指定参数如下

```shell
TsmartCheckerMain.java -config=config/phase/default/default.properties
```

* TsmartCheckerMain.main
  * initOptionParser - 初始化命令行参数解析器 
  * convertArguments - 构造参数，形式为[key, value,...] 
  * createConfiguration - 转化参数为Configuration类的形式，key-value properteis 
  * new TsmartCPAChecker - 构造TsmartCPAChecker
  * TsmartCPAChecker.run - 执行run开始分析
* TsmartCPAChecker.run

  * new TsmartCPAPhaseManager - 构造Phase管理器，将权限交给phase处理(这个时候会读取phase.manager.config 和 phase.manager.executionType)
* TsmartCPAPhaseManager.initialized
  * new PhaseConfigManager - 读取phase.manager.config内的内容，构造phase，并建立依赖关系
* TsmartCPAPhaseManager.execute
  * 这个部分实际是由phase.manager.executionType构造的策略执行。这里面是SequentialExecStrategy执行。其策略是检查所有依赖，都完成了再执行自己。所以如果遇到父phase未执行则递归的向上判断。




下面就是各个phase执行的过程：CFACreatePhase, BasicAnalysisPhase

CFACreatePhase

* prevAction
* runPhase
  * 构造cfa，这部分交给CfaCreator
* postAction


BasicAnalysisPhae 默认使用SequentialExecStrategy, 包含两个个子Phase：SingleAlgorithmRunPhase，AnalyzingResultPhase

* prevAction

  * 创建SingleAlgorithmRunPhase，AnalyzingResultPhase 

* runPhase

  * 执行相应的phase

  * printResultAndStatistics - 打印统计信息

* postAction




**SingleAlogrithmRunPhase 算法的核心在这里**

- prevAction
  - check cfa
  - create reachedSet -> ReachedSetFactory.create, 主要是设置TraversalMethod[BFS/DFS]
  - create algorithm
  - create cpa -> CPABuilder： 先ARG{Composite{location, uninitialized}}
  - initialize reachedSet
- runPhase
  - 将权限交给algorithm.run
- postAction



CPAAlgorithm.run 对于一个state，我们获得基于CFANode的所有出边对应的后继状态。再根据规则判断，要不要把这些后继状态加入的waitlist里面。同时，我们将边上的操作对应的语义基于当前state更新到后继状态中。具体的CPA对state的操作不同，参看具体模块参考部分。

```java
while(reachedSet.hasWaitingState()){
  handleState(state, precision, reachedSet)
}

handleState(){
  // 1. we compute the successors of the state
  successors = transferRelation.getAbstractSuccessors(state, Lists.<AbstractState>newArrayList
        (), previousPrecision);
  
  // for each successor, check following: (basically, the successor should add into waitlist)
    // 1. precision adjustment
  			// break
  			// continue
    // 2. merge this successor with all the states which are previously reached
  			// if merged remove old and add new into waitlist, respectively
    // 3. stop
  			// if stop continue
  			// else add into waitlist
}
```



**AnalyzingResultPhase**

TODO



### 具体模块参考

这部分具体讲解每个模块内部的逻辑关系

#### Configuration - TODO：我们需要所有可能配置的清单

Configuration部分是我们工具非常重要的模块之一。同时，也是我们主动保留的内容。

* CPA算法的精髓在于多个子cpa协同工作，各自处理好自己的事情，然后组合在一起发挥作用，所以需要进行配置不同的cpa针对于不同的应用场景。
* 同时，为了方便在分析过程中，加入多个分析模块，或者说为了能够模块化分析过程，所以我们引入Phase的概念。在不同的Phase中，做不同的事情，比如CFA构造、分析算法、结果处理等等。这些不同的phase也需要各自独立的配置和一些参数。

##### 配置文件结构

目前我们的配置文件有三类，我称之为：Top，Phase，CPA，可以参考上文中写配置那部分

* Top(default.properties)

  ```shell
  这个配置文件的功能：1.phase文件的位置 2.输入文件位置 3.全局的一些信息：phase执行策略，输入输出等
  必须要提供的内容：
  # 指定phase的配置文件在哪里
  phase.manager.config = config/phase/default/default.config
  # 指定分析策略，我们目前只有这一个策略
  phase.manager.executionType = SEQUENTIAL
  # 指定待分析程序
  input.programs = config/phase/default/default.ll
  ```

* Phase(default.config)：

  ```shell
  在这个文件文件中设置具体要操作的phase过程，通常如下：
  1. 配置DIR
  2. cfa - 必须要有
  3. 分析的phase
  4. 指定phase之间的依赖关系
  # 设置基本路径，因为我们的程序会用到类加载机制
  .SET
  DIR = cn.edu.thu.tsmart.checker.core.phase
  .TES
  # 创建的第一个phase，构造cfa
  .DEF cfa # 名字是cfa
  .TYPE = $(DIR).CFACreatePhase # phase的类型 -> 会对应一个具体的phase类来加载
  analysis.summaryEdges = true # 这个phase用的配置项
  .FED
  # 创建的第二个phase，分析
  .DEF basic # 名字是basic
  .TYPE = $(DIR).BasicAnalysisPhase # 用BasicAnalysisPhase加载
  # 指定具体cpa配置文件的位置
  phase.singleAlgorithm.analysis = config/phase/default/uninitialized.properties 
  phase.singleAlgorithm.stopAfterError = false # TODO
  phase.singleAlgorithm.initialStatesFor = ENTRY # 指定分析的入口，用来获得初始分析的位置
  .FED
  # 设置依赖关系
  .RUN
  basic : cfa;
  ```

* CPA(uninitialized.properties)：

  ```shell
  这个配置指定具体分析的cpa构成，通常
  1. cpa = cpa.arg.ARGCPA -> 指定最顶层的wrapper cpa是ARGCPA
  2. ARGCPA.cpa = cpa.composite.CompositeCPA -> ARGCPA有孩子而且是single孩子，孩子是CompositeCPA
  3. CompositeCPA.cpas = cpa.location.LocationCPA,XXX -> CompositeCPA有多个孩子，他们是LocationCPA，XXX
  4. 其他的一些配置：分析顺序[bfs|dfs],等等
  # top cpa
  cpa = cpa.arg.ARGCPA
  # *.cpa child of *
  ARGCPA.cpa = cpa.composite.CompositeCPA
  # *.cpas children of *
  CompositeCPA.cpas = cpa.location.LocationCPA, cpa.uninitvars.UninitializedVariablesCPA

  analysis.traversal.order = bfs
  ```

##### config解析使用

我们的所有解析配置文件的操作是使用CPAChecker的一些工具类。其实，Configuration中是一些

< key:String, value:String >对

###### 从文件中创建

```java
ConfigurationBuilder configBuilder = Configuration.builder();
configBuilder.loadFromFile(configFile);
Configuration config = configBuilder.build();
```

###### 从已有的config创建

```java
Configuration.builder().copyFrom(config)
```

###### 手动存入配置

```
ConfigurationBuilder#setOptions(Map<String, String>)
```

###### config设置类变量

所有需要通过config配置的内容需要如下：

1. 类头标签+前缀

   ```java
   @Options(prefix = "phase.manager")
   public class TsmartCPAPhaseManager{}
   那么之后这个类里面所有的变量都是phase.manager.NAME来配置
   ```


2. 成员变量标签

   ```java
   @Option(secure = true, name = "config", description="XXX") //指定通过config来配置
   private String managerConifg = "XXX"; // 默认值
   所有这个变量的标签是phase.manager.config
   ```

   ​

3. 构造函数调用方法

   ```java
   config.inject(this);
   所以执行后的行为就是将配置中的
   phase.manager.config = config/phase/default/default.config
   配置为
   managerConifg = config/phase/default/default.config
   ```



#### Statistics

* Result 每个Phase对应的结果，可能包含Statistics，Status。每个Phase都有一个currentResult

  * CPAPhaseEmptyResult - phase初始的值


* Status 程序进行的状态
  * CPAPhaseStatus - 用于Phase中，SUCCESS|FAIL|INTERRUPT|OTHER
  * AlgorithmStatus - （precision， sound）
* Statistics 分析过程中的统计信息，比如时间等
  * TsmartMainCPAStatistics

  * ARGStatistics

  * CPAAlgorithmStatistics




#### Phase

phase是我们分析的大脑。他负责调控所有的分析过程。

##### Phase管理器 - TsmartCPAPhaseManager

```
phase管理器，两个操作。(构造的时候会配置执行策略)
1. initialize
	// 读入phase配置文件
	// PhaseConfigManager(这一步是将config解析): 
		(1) read in raw text 
		(2) build phase collection 
		(3)set up dependency
	// createPhases(这一步是根据config构造phase)，例如如下配置
.DEF basic # 名字是basic
.TYPE = $(DIR).BasicAnalysisPhase # 用BasicAnalysisPhase加载
# 指定具体cpa配置文件的位置
phase.singleAlgorithm.analysis = config/phase/default/uninitialized.properties 
phase.singleAlgorithm.stopAfterError = false # TODO
phase.singleAlgorithm.initialStatesFor = ENTRY # 指定分析的入口，用来获得初始分析的位置
.FED
		// (1) 通过Type获得类用BasicAnalysisPhase
		// (2) 反射的构造类
2. execute
	交给配置中的策略执行phase
```

##### Phase执行策略 - CPAPhaseExecStrategy

* SequentialExecStrategy

  ```
  1. 获得一个没有父依赖的phase作为entry
  2. 执行entry， 检查后继者s
  	ready -> 执行s
  	not -> 执行s的ancestor
  		ready -> 执行s'ancestor
  		not -> 执行s'ancestor'ancestor
  3. 所有phase都执行完
  ```

##### 单个Phase执行过程 - CPAPhase

```
1. 依赖的phase是否都ok？
2. 执行自己
每个phase自己的操作如下
prevAction
runPhase
postAction
```

* CFACreatePhase

  ```
  构造CFA，所有CFA相关参考CFA部分
  ```

* BasicAnalysisPhase

  ```
  最常用的分析过程，是一个复合phase，内部含有两个phase
  1. SingleAlgorithmRunPhase
  2. AnalyzingResultPhase
  以及SequentialExecStrategy执行策略
  ```

* SingleAlgorithmRunPhase

  ```
  - prevAction
    - check cfa
    - create reachedSet -> ReachedSetFactory.create, 主要是设置TraversalMethod[BFS/DFS]
    - create algorithm
    - create cpa -> CPABuilder： 先ARG{Composite{location, uninitialized}}
    - initialize reachedSet
  - runPhase
    - 将权限交给algorithm.run
  - postAction
  ```

* AnalyzingResultPhase

  ```
   TODO
  ```

  ​



#### ReacheSet

ReachedSet模块是分析过程中的一个很重要的数据结构。存放了我们分析过程中的每一步的结果。在分析后，通常是便利这个用来获得分析过程的具体内容。主要的两个内容是：reacheSet(容器用来存储)，waitlist(等待队列，控制分析策略比如DFS／BFS)

继承关系如下：

* UnmodifiableReachedSet
  * ReachedSet
    * DefaultReachedSet
      * PartitionedReachedSet
* Waitlist
  * AbstractWaitlist
    * SimpleWaitlist


##### 容器 - ReachedSet

* UnmodifiableReachedSet

  ```java
  接口，用来规定所有reached set需要的操作，且这些操作是不可改变已有内容的操作。
  我一开始对这个认识不够深刻，他和ReachedSet的区别就是，这里面的行为是突出Unmodifiable
  所以你里面都是getter
  ```

* ReachedSet extends UnmodifiableReachedSet

  ```java
  这个是ReachedSet的核心，定义所有增删的操作。
  其中几个点很关键：
  1. 包含一个Waitlist， 这个Waitlist也是控制分析过程如何选取下一个状态的关键
  2. 这个是一个ordered set，所以加入的顺序就定了，其实所有的add操作是在waitlist上
  ```

* DefaultReachedSet implements ReachedSet

  ```java
  实现了ReachedSet的最基础的类，几个核心的数据结构
    // for all reached abstract states with precision. 好吧，至今我都不知道这个Precision是做什么的
    // 道理上，这个precision是所有用于标记分析过的abstractState的结果，但是吧，key是abstractState，value是precision。这个是一一对应关系。每次add的时候，abstractState应该不同。如果存在这么一个abstractState了，那么它的Precision也应该是一样的。感觉上是双保险的感觉。
    protected final LinkedHashMap<AbstractState, Precision> reached;
    // for all reached abstract states, unmodifiable
    protected final Set<AbstractState> unmodifiableReached;
    protected final Waitlist waitlist;
    protected AbstractState firstState = null;
    protected AbstractState lastState = null;
  ```

* PartitionedReachedSet

  ```java
  这个是DefaultReachedSet的一个特殊情况
   * Special implementation of the reached set that partitions the set by keys that
   * depend on the abstract state.
   * Which key is used for an abstract state can be changed by overriding
   * {@link #getPartitionKey(AbstractState)} in a sub-class.
   * By default, this implementation needs abstract states which implement
   * {@link Partitionable} and uses the return value of {@link Partitionable#getPartitionKey()}
   * as the key.
   *
   * Whenever the method {@link PartitionedReachedSet#getReached(AbstractState)}
   * is called (which is usually done by the CPAAlgorithm to get the candidates
   * for merging and coverage checks), it will return a subset of the set of all
   * reached states. This subset contains exactly those states, whose partition
   * key is equal to the key of the state given as a parameter.
   
   其实是这样子的，我们的分析中，并不是那么随意的。也就是说，我们必须要有一个locationState这样的元素，标记我们在那里。所以分析时，我们很有可能又到了这个location。根据CPA算法呢，我们就要看看是不是做merge或者stop。那如何获得之前的abstractState呢？DefaultReachedSet里面是直接用abstractState检索，所以返回的是全部。这个显然是不对的。所以我们用这个类就很好，每个AbstractState我们搞一个Partition Key，所以检索就用这个Key。通常来说就是location和callStack就能标记一个位置了。
   
   ！！！注意这个东西肯定是有自己的数据结果所以一定要注意clear()这类操作不然会有数据结构不回收，内存就炸了
     @Override
    public boolean clear() {
      super.clear();
      partitionedReached.clear();
      return true;
    }
  ```

##### 等待队列 - Waitlist

* Waitlist

  ```
  控制者如何获取下一个需要分析的状态
  Implementations differ in the strategy they use for pop(). Such as DFS, BFS
  ```

* AbstractWaitlist implements Waitlist

  ```
  public abstract class AbstractWaitlist<T extends Collection<AbstractState>> implements Waitlist
  所以所有的子类需要提供一个Collect作为存储的数据结构
  ```

* SimpleWaitlist

  ```
  目前我们实现的一个waitlist，通过TraversalMethod来控制pop的过程
  内部存储用的是Deque，add不变
  pop是DFS或者BFS来操控，一个是从前边一个是从后边
  ```

* TraversalMethod

  ```
  一个enum，
  根据不同的TraversalMethod，来创建不同的Waitlist
  ```



#### Algorithm

我们的算法就是实现了论文里面的算法。

##### 分析算法 - CPAAlgorithm

```java
while(reachedSet.hasWaitingState()){
  handleState(state, precision, reachedSet)
}

handleState(){
  // 1. we compute the successors of the state
  successors = transferRelation.getAbstractSuccessors(state, Lists.<AbstractState>newArrayList
        (), previousPrecision);
  
  // for each successor, check following: (basically, the successor should add into waitlist)
    // 1. precision adjustment
  			// break
  			// continue
    // 2. merge this successor with all the states which are previously reached
  			// if merged remove old and add new into waitlist, respectively
    // 3. stop
  			// if stop continue
  			// else add into waitlist
}
```

#### CPA

简单的来说CPA可以认为是一套符号执行的过程。原则上，我们希望每个CPA只管理一个内容。但是一个可怕的后果会是，状态空间的爆炸。比如说CPA~a~ 和 CPA~b~ 分别在一条语句后产生m和n个可能的后继状态。那么他们的组合就是$$ m * n $$。

我们的这个系统中，一个CPA饱含如下几个内容

##### CPA组成

```

1. domain 
	抽象域，这个是你自己定义的一个规则，做两件事情（其实用到了Lattice，这个有些形式化，我最初也是不理解）
	（1）join - 给你两个state，你怎么合并 （这个给Merge用）
	（2）isLessOrEqual - 给你两个state，你怎么判断谁大谁小，也就是给出两个状态的偏序关系（这个给Stop用）
	比如：LocationCPA中，我们认为lattice是一个三层的，TOP - 其他 - BOTTOM，所以呢join其实没有什么意义，isLessOrEqual就是比较TOP
	比如：看UninitializedCPA，我们认为top是所有变量，bottom是没有，所以呢join是把两个state中为初始化的取并集合，isLessOrEqual呢就是看看第一个的内容是不是都被第二个包含了。比如A={a,b} B={b}那么join(A,B) = {a,b}。而且
	B isLessOrEqual A

2. precision 
	这个我也不太清楚，至少目前我们没有遇到这个的操作。道理上，这个是用于精度调节的结果。一般都是给SingletonPrecision

3. precisionAdjustment // 对于一个wrapper内的要一致
	精度调节操作符，一般都是StaticPrecisionAdjustment，也就是说不调节

4. abstract state
	这个是抽象状态，是存储你这个CPA负责的任务的数据结构
	比如：LocationCPA就是存每一个程序节点，也就是CFANode
	比如：UninitializedCPA，就是存分析在一条边上的语句的作用下，边前边的statePrev 应该怎么变化，结果放在stateNew中。比如说，
	statePrev = {a,b}, edge = "int c;" then, stateNew = {a,b,c}
	statePrev = {a,b}, edge = "a = 1;" then, stateNew = {b}

5. transferrelation
	这个就是针对与一条边和当前的状态，我们如何符号执行这条边上的语句，获得新的内容。

6. merge(A,B) // 对于一个wrapper内的要一致
	把两个状态按照配置进行合并。
	如果：sep，就是不合并
	如果：join，就合并，用的是domain#join

7. stop(A,B)
	判断是否停止分析，也就是说B能够cover A，也就是说A isLessOrEqual B
	为什么说cover就不用分析了呢？
	想想我们的分析过程，其实就是从程序开始，更新状态，更新的原则就是边上的语句。那么如果之前到过一个点，状态为A，第二次到的时候是B。如果A涵盖了B，也就说明，A的信息比B还要多，所有A的后继的信息就是比用B再进行分析的后继的信息多。那么我们就不用再分析了。因为CPA算法是over approximation。所以越多越好
8. factory
	这个是不需要有的，用于读取配置的时候，基于反射的构造一个CPA。而且固定模式
	public static CPAFactory factory() {}
	在创建CPA的时候，首先会通过CPAFactory，set进去各种需要的内容。然后createInstance。
	其实这个factory分两类
	1. 只依赖一般的的内容，那么AutomaticCPAFactory.forType(ARGCPA.class);就可以。参看ARGCPA
	2. 需要有特殊的构造函数，那么自己写一个这样的函数，返回一个自己的factory，这个factory根据预先放进去的内容选择不同的cpa构造函数。参见LocationCPA
```

##### 必有CPA

###### ARGCPA

```
这个是最外层的包装CPA，孩子是CompositeCPA。可以看作一个傀儡CPA，很多操作是在CompositeCPA做。主要对外
```
###### CompositeCPA

```
中层CPA，掌权者
```
###### LocationCPA

```
标记CPA，通过location 来获得下一个要分析的点
```
###### CallStackCPA

```
标记函数调用情况
1. domain - FlatLattice
2. Stop - sep
3. Merge - sep
4. factory - auto
5. PrecisionAd - static
6. Precision - single
7. state
(1) previous state
(2) current function name
(3) caller cfaNode (function entry node)
(4) depth
8. trans
这个只要处理两种
（1）如果是funCall边
		A. 正常创建新的state
		B. 如果是递归函数（之前的state里面出现过函数名），还要看是否超过配置的递归层数。如果超过了，那么就停止不进入函数，但是这个时候需要判断是否是有相应的summary边可以让你接着向后分析
（2）如果是call return边
		A. 现在所在的函数是否是当前状态的函数 -> 应该是
		B. 检查通配符wildcast也就是看这个是不是正常的funcCall 
			a. main - caller.fName = state.fName
            b. other fun call - caller.succs.filter(fEntryNode).anyMactch(state.fName)
		B. 应为一个函数返回会有很多条边到不同的调用处，所以选择state中保存的callerNode对应的位置，返回上一个state
```
##### 其他CPA

###### UninitializedVariableCPA

```java
在这个CPA里面，我们的目的是找出那些使用了未初始化的变量，比如
int a;
int b = a; // error

首先：我们这个CPA中存储的信息是所有声明了的变量，如果一个变量被定义，那么就移除。每次遇到一个变量被使用时，我们会去看是否在存储中，如果在，那也就是说还没有定义，那么就出错了。

目前我们的版本是基于数据流，不考虑控制条件。

发现：对于 FuncA -> FuncB -> FuncC
(1) 我们要有一个Global 的数据结构给全局变量
(2) fCall 我们要单独维持一个local Frame
(3) 我们发现，由于callStack和location共同的作用，到达同一个点的两个执行路径，其callStack一定是相同的，因为getReached时候这两个是key

1. domain
	(1) join - 只要执行global和当前local就行。试想这个join是开了merge才会执行。那么FuncA->FuncB的时候，一定有某个点让这两条路径都到了这里。才会有后面的FuncB->FuncC，所以呢，前边的LocalFrame也都是合并过的。
	(2) stop - 需要比较所有。试想既然到了这里，那么一定是前边没有stop，也就是说前边!(a isLessOrEqual b)，所以，到这里的时候要都判断。
2. Stop - sep/join
3. Merge - sep/join
4. factory - auto
5. PrecisionAd - static
6. Precision - single
7. state目前我们没有把出错信息放在这里
(1) global varialbes names - Collection<String>
(2) local variables names - Deque< Pair<String, Collection<String>> >
  Pair[functionName::String : localVarNames::Collection<String>] -> 
  Pair[functionName::String : localVarNames::Collection<String>] -> ...
8. trans
  我们先考虑在C里面的情况
  (1) varialbe Declaration
  	A. global?local?
  	B. initializer?
  	C. array? struct?
  (2) FunctionCall
  	A. parameters? -> new local frame
  (3) FunctionReturn
  	A. drop local frame -> return value
  (4) statement
  	A. functionCallSummary? check parameters
  	B. assignment? a = b; check b and set a
  	C. treat as expression
  (5) assume
  	A. treat as expression
  
      
  For LLVM
      alloca -> store -> load
  (1) declaration
      alloca -> add into local name|*
      store -> remove from local 

  
```






#### CFA

CFA部分用于构造Control-Flow-Automata，这个是我们分析的基础。所有的分析都是在图上进行，可以理解为图上可达性分析。CFA的构造依赖于LLVM模块。

##### 基础数据结构

- CFANode - CFA图中的所有节点，本身没有信息存在点上。所有的操作存在CFAEdge上。本身含有(nodeNumber, leavingEdges, enteringEdges, basicBlock, functionName)

  - FuncitonExitNode - 包含(entryNode)
  - FunctionEntryNode - 一个LlvmFunction对应的进入节点包含(exitNode, parameterNames)

- CFAEdge - CFA图上所有边信息，存储着实际的操作(predecessor, successor, rawCode, fileLocation)，目前我们有八种边

  - BLANK_EDGE - 空边
  - INSTRUCTION_EDGE 最常用的指令边
  - ASSUME_EDGE 判断条件，含有一个AssumeCondition(Branch 或者 Switch)
  - PHI_EDGE - 包含phi指令的边
  - FUNCTION_CALL_EDGE 
  - FUNCTION_RETURN_EDGE 

  所有的Summary Edge 用来记录摘要信息，可以加速分析过程

  - FUNCTION_SUMMARY_EDGE  // 这条边也用于函数调用中
  - LOOP_SUMMARY_EDGE 

##### 构造过程

* CfaCreator

  ```java
  与外部分的接口，一般我们通过

  CfaCreator#parseFileAndCreate 获得CFA

  构造的过程:
  1. IRReader解析ll文件，产生Llvmodule{BasicBlock{Struction}}
  2. CfaBuilder#parseToCfa 转化LlvmModule到内部的数据结构中
    	for(LlvmFunction : module){
        handleFunction()
    	}
  3. build -> 获得结果
  ```

* CfaBuilder#handleFunction

  ```java
  // exit node for function
  // entry node for function
  // prepare
  	for(BasicBlock)
        	// 没有顺序关系和依赖关系， 但BasicBlock中的指令是顺序执行没有分支的
    		generate -> (List<PhiNode>, List<Instruction>, TerminartorInst) 
  // 1. handle normal instructions -> build CFAEdge
  	for(List<PhiNode>, List<Instruction>, TerminartorInst){ 
        // create startNode
        // create endNode
        // create InstructionEdge as startNode->edge->endNode
  	}

  // 2. link BasicBlock by TerminartorInst
  	for(List<PhiNode>, List<Instruction>, TerminartorInst){ 
        case ReturnInst: 创建空边，从BlockExit -> FunctionExit
        case BranchInst: 
        	if(conditional){
            1. true branch - transitToBlock // 可以拿到trueBlock
            2. false branch - transitToBlock // 可以拿到falseBlock
        	}else{
            transitToBlock
        	}
        case SwitchInst:
        	for(switchInst#cases)
            依次构造cases
            transitToBlock
  	}
  	
  // 3. handle first edge
  	函数的第一条边 functionEntryNode -> BlankEdge -> firstBlockEntryNode
  // 4. definedFunctions#add
      add into definedFunctions
        
  // 这个函数将不同分支和前后Block链接起来
  transitToBlock predNode, predBlock, succNode, succBlock, phiList
        1. which phiNode is?
        2. create a new Node as next
        3. PhiEdge (predNode -> phiEdge -> next)
        4. BlankEdge (next -> BlackEdge -> succNode)
        
  ```



#### LLVM





#### ErrorReport





### 其他参考

* Gradle: https://www.jetbrains.com/help/idea/2017.1/gradle.html
  * 其实，参考其他模块的gradle文件即可，最主要的就是compile依赖怎么写的问题
* 常见LLVM IR指令的语法与语义信息: http://llvm.org/docs/LangRef.html



### Q&A



