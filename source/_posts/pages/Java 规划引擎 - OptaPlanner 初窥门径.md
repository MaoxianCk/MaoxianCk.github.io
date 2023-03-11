---
title: Java 规划引擎 - OptaPlanner 初窥门径
date: 2023-03-11 16:59:18
updated: 2023-03-11 16:59:18
categories:
- 程序
tags:
- Java
- 代码
- 程序
- 规划引擎
---

# 前言

前一段时间接了一个单子，主要的需求内容是：对工厂车间中人员、设备、资源的规划和安排进行分析，产出一个类似甘特图状的排产排程结果，可以根据人员和资源的情况进行合理规划，最后我们选用了OptaPlanner这个规划引擎来做这事。这个引擎功能感觉比较强大，但是中文的相关资料信息和讨论都不是很多，所以就有了写本篇文章的想法。

文章的前面部分主要是对于这个引擎概念上的讲解，最后一部分是本人在使用过程中的一些 Tips 或者说遇到的问题和解决方案。

# 概述

[OptaPlanner](https://www.optaplanner.org/) 顾名思义，planner就是规划，这个引擎就是一个规划引擎。规划所有可以规划的，根据给定的问题、约束、数据，在约束限制内为对应问题规划出整体最优的可行解。
举个例子，也是官方文档给出的[示例](https://www.optaplanner.org/docs/optaplanner/latest/quickstart/hello-world/hello-world-quickstart.html#_solution_source_code) ：课程表排课，以下内容也是根据该示例整理出来的，对应各种概念的解释。该文档仅描述概念性的内容，主要基于官方文档的翻译以及个人的理解，具体的示例代码，还请读者结合官方文档中的示例，这里就不再复制。

# 概念讲解


## 问题及内容

**问题**：给课程表排课
**涉及到的所有参数**：课程时间，教室，课程

### 参数说明

课程表排课，共涉及到三个变量，时间、教室、课程。

**时间**：所有课程可以安排的时间段，如周一早8节（8:00 - 9:40），周一34节（10:00 - 11:40），周二早8节，周二34节 等等。
*时间段本身的时间范围是不会变动的*

**教室**：所有课程可以安排的教室，如公教1，公教2等教室。
*教室和时间一样，不会变动*

**课程**：课程就是课程表上的一节课，如周一早8节，由王老师在公教2教室给计算机1班学生上的操作系统课程。课程除了本身具有的属性（教师、学生、科目）外，包含了时间和教室两个数据。
*教师和学生不会变动，时间和教室 可以依据排课而变动*


## 关联问题、参数与OptaPlanner

上文中提到的三个参数，都是将课程表排课涉及到的内容抽象提取出来的*实体类*，这里的实体类就是面向对象中的类，是代码对现实世界的抽象描述。


**时间**：在我们进行排课的时候，时间不会因为排课而被修改，比如周一的早8必定是在8:00-9:40来上，不管怎么排课，时间这个参数的内容是不会被规划引擎修改的。在Opta中把这种不会被修改的实体称为***问题事实 ( Problem Fact )***，问题事实不需要使用任何java注解标记。


**教室**：与时间同理，教室也不会因为排课而被修改，公教1就是公教1，不会因为排课就把公教1变成公教777。所以教室实体也是一个***问题事实 ( Problem Fact )***，不需要java注解标记。


**课程**：我们进行排课的主要对象，就是课程。课程本身带有三个属性（教师、学生、科目），这三个属性在我们这个示例中是不变的（这个例子我们只排时间和教室），而课程除了这三个属性外，还有*时间*和*教室*，这两个参数是我们在进行排课规划的时候需要给课程计算的，这个计算过程，就是我们这个示例的*核心*，计算结果就是我们这个*示例的解*。

所以对于课程这个我们规划所要求的实体，称为***规划实体 ( Planning Entity )***，规划实体需要使用<em>@Planning Entity</em> 注解标记。

课程中的*时间*和*教室*，由于需要在规划中被规划引擎修改（也就是排课的过程），所以它们被称为***规划变量 ( Planning Variable )***，规划变量需要使用<em>@PlanningVariable</em> 注解标记。


### 约束（硬约束、软约束）

排课的过程，也就是规划问题的主要问题，就是**约束 ( Constraints )**，规划任务需要在一定的约束条件下，结合具体的资源（参数）给出一定条件下的最优解。

这个示例中的约束共三个：
1. 同一个时间段内，一个教室只能上一门课。
2. 同一个时间段内，一个老师只能上一门课。
3. 同一个时间段内，一个学生（班级）只能上一门课。

约束分为两类，**硬约束、软约束**

约束，可以看作条件，任务在规划的过程中受到约束的制约，所以约束也是评分器，每一个问题的解决方案（解）都会根据约束进行评分，评分也分为两类。

**硬约束**：很硬，在一个具体问题中，硬约束是不可打破的，也就是说不可以被违背。比如排课，在同一个时间段内，一个教室不可以上两门课；同一个时间段内，一个老师不可以上两门课。

**软约束**：可以被打破，比如老师更喜欢在公教1上课。

由于约束同时也是评分条件，所以合法的解必定符合硬约束，尽量少的违背软约束。即硬约束是不可违背，强制性的定义，所以用来定制规则行为。而软约束由于可以被违反，所以可以由此影响到整体规划的结果偏向，例如在生产规划中，成本即是软约束，在生产规划中，符合生产流程的前提下，成本越低，方案越好（评分越高）。


### 参数的收集

上文都是在描述课程表中的内容，而这些东西都是被课程表包含其中。

对于类似课程表这样的“收集器”，称为***规划方案 （ Planning Solution ）*** ，使用<em>@PlanningSolution</em> 注解标记。

课程表拥有的内容：

一个时间列表（所有可以安排课程的时间段组成的列表）如：\[ '周一（8:00-9:40）, 周一（10:00-11:40）...\]

一个教室列表（所有可以安排课程的教室列表）

时间列表和教室列表等由*问题事实*组成的列表，就是***规划变量范围的提供器（Value Range Provider）***，使用<em>@ValueRangeProvider 和 @ProblemFactCollectionProperty</em> 注解标记。

一个课程列表（所有需要被安排的课程），课程列表由规划实体组成，使用<em>@PlanningEntityCollectionProperty</em> 注解标记。

分数（一个排好的课程表的分数，反映这个课程表的好坏），使用<em>@PlanningScore </em>注解标记。


***规划方案（ Planning Solution ）***，本身由于包含了所有的信息，所以即可以当作规划问题的*输入载体*，也可以当作规划问题的*输出载体*（解）。

在本例中，时间和教室两个列表由于是**问题事实 ( Problem Fact )**，不会改变，在问题求解之前就可以给出；课程列表包含着所有需要被*规划*的课程内容，除了本身的教师、学生、科目这三个属性不变外，课程的时间和课程的教室这两个**规划变量 ( Planning Variable )** 需要在规划中确定，所以在求解之前，是**null**的空值，这两个属性会在求解后被填充。

# 实际使用

这边主要是在系统中调用和使用该引擎的一些问题和经验，至于调度引擎本身的出入参及约束的编写和调整（算法届通常叫炼丹？），还请看官方文档中接近实际需求的示例和讲解。

### Java JDK的版本：

该文章写作日期时的版本为 8.35.0.Final，该版本要求JDK 11+，如需要使用，请确保JDK版本为11及以上。

### 任务调度

引擎提供了两种方式启动一个计算任务：Solver、SolverManager。

#### [Solver](https://www.optaplanner.org/docs/optaplanner/latest/planner-configuration/planner-configuration.html#useTheSolver)

针对单次计算任务的求解器。

```java
// Load the problem 加载规划方案
TimeTable problem = generateDemoData();

// Solve the problem 求解问题
// 创建一个求解器工厂，并设置好参数
SolverFactory<TimeTable> solverFactory = SolverFactory.create(new SolverConfig()
        .withSolutionClass(TimeTable.class)
        .withEntityClasses(Lesson.class)
        .withConstraintProviderClass(TimeTableConstraintProvider.class)
        .withTerminationSpentLimit(Duration.ofSeconds(5)));
// 通过工厂对计算生产一个Solver
Solver<TimeTable> solver = solverFactory.buildSolver();
TimeTable solution = solver.solve(problem);

// Visualize the solution
// 分析结果
printTimetable(solution);
```

这种方式是最简单的使用方式，在调用Solver.solve方法后启动计算任务。注意，该线程将会阻塞直到预设的结束时间，或者在*手动*停止前无限计算。是的，这个引擎针对的规划问题在搜索空间上通常是海量的，所以一般情况下 运算时间相当于*无限*，所以我们需要手动设置一个结束时间或者通过手动停止的方法来强迫结束计算，又或者通过[配置](https://www.optaplanner.org/docs/optaplanner/latest/optimization-algorithms/optimization-algorithms.html#bestScoreTermination)，让它在得到一个较好的答案时停止。

#### [SolverManager](https://www.optaplanner.org/docs/optaplanner/latest/planner-configuration/planner-configuration.html#solverManager)

求解器管理器。

初始化：

```java
SolverConfig solverConfig = new SolverConfig()
		.withSolutionClass(TimeTable.class)
		.withEntityClasses(Lesson.class)
        .withConstraintProviderClass(TimeTableConstraintProvider.class)
        .withTerminationSpentLimit(Duration.ofSeconds(5)));
// 第一个泛型类型对应规划实体，第二个参数是计算任务的Problem ID, 类型通常是一个不可变类型，比如Long、String、UUID等，用来标记每个不同的任务
SolverManager<TimeTable, String> solverManager = SolverManager.create(solverConfig, new SolverManagerConfig());
```

启动计算：

```java
// manager返回的是一个计算任务，类似于js中的Promise机制，调用solve方法会马上返回，并不会阻塞线程的执行
SolverJob<TimeTable, String> solverJob = solverManager.solve(problemId, problem1);

// 会阻塞线程，直到计算停止并返回一个最好的答案
TimeTable solution1 = solverJob.getFinalBestSolution();
```

当然，正常我们在使用的时候，几乎不可能将线程阻塞，SolverManager提供了另一个启动方法：

```java
// Returns immediately 
public void solveLive(Long timeTableId) {
	// 这个方法调用并不会阻塞线程
	solverManager.solveAndListen(
			// 这个计算任务的id，我们自己生成，对应SolverManager的第二个泛型类型
			timeTableId,
			// 在启动的时候调用一次，这个参数传的是一个方法
			// 输入任务id，返回用于求解的规划实体
			this::findById,
			// 回调，在每一次找到一个新的最优解的时候调用，可能会在执行中被调用多次
			this::save,
			// 回调，在结束计算时，返回一个最优解，只会在终止计算的时候返回一次
			this::save);
}

// 作用是根据id找到用于求解的规划实体，用于启动计算，提供入参
public TimeTable findById(Long timeTableId) {...}

// 在每次找到最优解的时候调用，用于自定义每个解的后续业务动作
public void save(TimeTable timeTable) {...}

// 手动根据任务id 停止一个计算任务的方法
public void stopSolving(Long timeTableId) {
	solverManager.terminateEarly(timeTableId);
}
```

实际使用中的样子：

```java
String taskId = IdUtils.simpleUUID();

SolverJob<Schedule, String> solverJob = solverManager.solveAndListen(
	taskId,  
	// 从前端传来的组装好的规划实体
	id -> inputSchedule,  
	schedule -> {  
		// 每次找到新的好方案时，做一个进度展示
		bestSolutionChange(taskId, schedule);  
	},  
	schedule -> {  
		// 表示计算结束了，做实际的存储和分析
		solutionEnded(taskId, schedule);  
		if (taskEndCallback != null) {  
			taskEndCallback.accept(taskId, schedule);  
		}  
	});
```

主要区别就是solveAndListen不会阻塞线程，并且提供了Hook，让我们可以在他计算结束的时候，通过回调方法来处理后续的业务。
并且manager默认就提供了资源平衡处理，在有多个任务计算时，会自动根据系统资源CPU核数等平衡同时计算的任务数，并处理多线程计算的资源消耗。

另外，OptaPlanner官方提供了SpringBoot的[starter](https://www.optaplanner.org/docs/optaplanner/latest/quickstart/spring-boot/spring-boot-quickstart.html#_prerequisites)，引用后会自动注入SolverManager。