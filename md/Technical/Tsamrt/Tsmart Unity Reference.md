# Tsmart Unity 参考

> @author Zuxing Gu
>
> 这将是一份旷日持久的文档。。。

### 前期准备

1. 配置环境：Ubuntu 16.04，llvm 3.9.0([源代码编译llvm](http://tomgu1991.github.io/blog/Technical/LLVM/LLVM_Introduction.html))，Gitlab添加权限、下载项目源代码
2. 理解这篇文章: [CAV 2007 CPA](https://www.semanticscholar.org/paper/Configurable-Software-Verification-Concretizing-Beyer-Henzinger/1929ed09538c988552ae92435c54509370bcbd4d) (CPA 算法原理)
3. 理解这篇文章: [CAV 2011 CPAChecker](https://www.semanticscholar.org/paper/CPAchecker-A-Tool-for-Configurable-Software-Beyer-Keremoglu/c84086d352be8b87b2266aeb82540602b20ec0c2) (我们工具参考框架结构)



### 代码结构

我们的代码是由**陈光**大神基于Gradle搭建，理论上这是一个Gradle项目。具体Gradle相关，参看其他参考部分。我们用的IDE是[IntelliJ IDEA](https://www.jetbrains.com/idea/?fromMenu) ，这个有教育版，学生可以免费试用。

我们有三个模块
* Checker 负责顶层算法和分析流程控制
* Engine 负责底层解析ll文件和构造CFA
* Mod-commons 负责基础通用的组建，比如Configuration, Logger, Shutdown, Annotations, IO and so on.




Mod-commons 负责基础通用的组建 Prefix: cn.edu.thu.sse.common

* annotations
* collect
* concurrency
* configuration
* io
* log
* time
* util
* TODO




Engine 负责底层解析ll文件和构造CFA Prefix: cn.edu.thu.tsmart.core.cfa

* llvm 
* model
* util




Checker 负责顶层算法和分析流程控制 Prefix: cn.edu.thu.tsmart.checker

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

  cfa.useMultiEdges = false
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



AnalyzingResultPhase

TODO



### 具体模块参考

这部分具体讲解每个模块内部的逻辑关系

#### Configuration

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

#### ReacheSet

#### CPA

#### CFA

#### LLVM

#### ErrorReport

### 其他参考

* Gradle: https://www.jetbrains.com/help/idea/2017.1/gradle.html
  * 其实，参考其他模块的gradle文件即可，最主要的就是compile依赖怎么写的问题
* 常见LLVM IR指令的语法与语义信息: http://llvm.org/docs/LangRef.html

### Q&A

