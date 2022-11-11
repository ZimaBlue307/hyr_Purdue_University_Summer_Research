# Automated Classification of Software Bug Reports

classifiers:

​	naive bayes(NB); SVM(support vector machines); random trees(RT);

their input are preprocessed bug reports with different fearures.

but it only classify all bugs into to classes: corrective (defect fixing) report and perfective (major maintenance) report.

# Analyzing bug fix for automatic bug cause classification

In this paper, we use the bug cause category as the classification system/architecture for the historical fixed bugs, and use the knowledge extracted from bug fix of the historical fixed bugs to automatically identify the causes of these bugs.

existing bug categorization criteria (Chillarege et al., 1992; Orthogonal Defect Classification; ODC), (Ieee, 1994; Common Weakness Enumeration; CWE);

propose an approach based on the representation of fix trees that can automatically classify the bug causes of the historical fixed bugs;

First, preprocess the commits of historical bugs to extract the bug fixes. Then, construct the ﬁx trees for these extracted bug ﬁxes and represent the **ﬁx trees** based on the encoding method of TBCNN (an unsupervised approach). Finally, we use the classiﬁer to automatically classify the bug ﬁxes into the corresponding bug cause category.

![image-20220725235244955](img\image-20220725235244955.png)

![image-20220726002313944](img\image-20220726002313944.png)

**corresponding fix trees: **

![image-20220725235338347](img\image-20220725235338347.png)



with the fix tree representation obtained from the above, use the classiﬁcation model for supervised learning to classify bugs into their bug cause category. Three classification techniques:

​	standard output layer of CNN;

​	Bayesian learning;

​	support vector machine (SVM);

each fix tree is represented as a 600-dimensional vector + category label——>601-dimensional vectors(the input of Bayesian learning model and SVM)

# A study of persistent memory bugs in the Linux kernel

not all related with PM-bug classification;

 ![image-20220726003536127](img/image-20220726003536127.png)

![image-20220726005520570](img/image-20220726005520570.png)

![image-20220726010706438](img/image-20220726010706438.png)

specification bugs (subtype of hardware dependent):

![image-20220726011402873](img/image-20220726011402873.png)

In this case, the PM device uses Address Range Scrubbing (ARS) [1] to communicate errors to the kernel. ACPI 6.1 specification [1] requires defining the size of the output buffer, but it is ambiguous if the size should include the 4-byte ARS status or not. As a result, when the nvdimm driver should have been checking for ‘out_field[1] - 4’, it was using ‘out_field[1] - 8’ instead, which may lead to a crash.



# Orthogonal Defect Classification

正交缺陷分类； maybe not useful for our project;

# Common Weakness Enumeration

related with security vulnerability, may be not concerned with our project;

# How existing tools automaticlly classify bugs

## PMRace

**focus on how to filter all false positives**: propose a post-failure validation scheme to verify if detected inconsistencies are fixed in the recovery stage.

PM Inter-thread Inconsistency: check if all inconsistent data are overwritten during the recovery stage;

PM Synchronization Inconsistency: in the post-failure stage, PMRace checks if the detected synchronization variable update is correctly restored to the expected value;

## PMDebugger

**No durability guarantee:** it means that a persistent memory location is not persisted after the last write to it. It happens when the programmer misses CLF or fence instruction. after the PM program finishes, PMDebugger checks if there are any remaining memory locations in the bookkeeping space: If the flushing state of a memory location is flushed, then the PM program is missing a fence. If the flushing state of a memory location is not flushed, then the PM program is missing a CLF.

**Multiple overwrites:** writes to the same persistent memory location multiple times before the durability of the memory location is guaranteed. examine if the memory location which the store instruction writes to already exists in the array or tree.(not in relaxed persistency models)

**No order guarantee:** PMDebugger asks the programmer to specify in a debugger configuration file which variable 𝑋 must be persisted before 𝑌 and at which application function. When the application function is called and PMDebugger processes a fence instruction, PMDebugger checks whether the order guarantee is satisfied.

**redundant flushes:** It will cause performance loss. PMDebugger checks if the flushing state of the memory location is flushed when processing a CLF instruction that persists the memory location.

**useless flush:** It happens when a CLF instruction does not persist any prior store. When processing a CLF instruction, if PMDebugger finds that the memory location persisted by the instruction does not exist in the bookkeep space, then PMDebugger reports a bug.

## Jaaru

"extend Jaaru with additional functionality to help developers quickly determine why a program crashed", but didn't tell how they extend Jaaru in detail.

## DURINN

DURINN's Durable Linearizability Validation part first runs recovery code before running validation operations, and run validation operations to check whether all operations before crash take effects and whether crashed operation is either fully executed or not at all executed.

validation operations:

![image-20220801092923531](img\image-20220801092923531.png)

## Witcher

![image-20220726200836010](img/image-20220726200836010.png)

**To detect correctness bugs:** 

compare two results between a program without a crash and the same program recover from a crash. if they are different—>report a bug(may be too general for us?);

**To detect performance bugs:** 

1. If a store still remains in the cache (not persisted) at the end of simulation yet it passes an output equivalence checking;
2. when simulating a flush instruction, it will check whether all prior stores have already been flushed by previous flush instructions;
3. when simulating a fence instruction, it will check if there are no preceding flush instructions.



# 会议小结

在Px86中也是有2个buffer，但是没有对read and write进行分类；why lens要对read and write进行分开解释呢？

we only need to know whether vans is feasible to simulate a model; so don't spend too much time on that. we need to think from a high level about:

1. front-end to randomly generate the program as the input of the simulator;(not complicated, just simple programs)
2. Replace PM operations such as store and load with operations to our simulator. 
3. need a simulator, be able to control the internal events and add crash and so on. 
4. detector to detect when failure happens
5. bug pattern reduction, whether they are the same bug;

当下：assume我们已经有了123，如何使用123去detect all the bug patterns?

## bug pattern simplification

x=1;//如果没有用在recovery就不算；

there are no undefined function in simplified program;

remove the bug about transaction;

# how entire design works

focus on the bug caused by PM operations rather than PM library.

## example 1

**no durability guarantee** in **PMDebugger / AGAMOTTO**. A persistent memory location is not persisted after the last write to it, similar to missing flush/fence pattern.

1. Front-end generate a valid example program:

```C++
x := 1;
//x := 1; y := 1;
```

2. the program assigns 1 to x, which is equivalent to the write operation in the simulator. So in general, the second step helps to transform the raw program to the  "equivalent program" of our simulator.
2. The simulator follows the semantic of Px86. It is able to add crash to the program, the method of adding crash effectively can be referred to the paper PMFuzz (crash image generation). 
3. To detect when failure happens: get the results of executing raw valid example program without a crash. Then, get the results of executing raw valid example program with crash(maybe executes for multiple times). There are two situations that we can detect that there maybe a bug: 1. when executing raw valid example program with crashes, we get different results. 2. compare the results of executing the raw valid program w/o crashes and get different results. 
4. Bug pattern reduction, get to know whether they are the same bug. In terms of the bug pattern about no durability guarantee, we can check if there are any remaining memory locations in the bookkeeping space after the PM program finishes: If the flushing state of a memory location is flushed, then the PM program is missing a fence. If the flushing state of a memory location is not flushed, then the PM program is missing a CLF (the way PMDebugger uses). (detection and bug reduction may be done at the same time)

## example 2

**redundant flushes:** It is more like a performance bug. 

1. Front-end generate a valid example program:

```C++
int x := 1;
flush(&x);
flush(&x);//or extra fence
```

this is a performance bug. the way to detect this bug can use the knowledge gained from paper WITCHER and PMDebugger.

2. assigning 1 to x, similar to example 1, can transform to the write operation in the simulator. and **flush operation, don't know how it transforms equally yet**.
3. The simulator follows the semantic of Px86. It is able to add crash to the program, the method of adding crash effectively can be referred to the paper PMFuzz (crash image generation). But in terms of this performance bug, maybe adding crash is not necessory.

4\5. report an extra flush performance bug: we check if the flushing state of the memory location is flushed when processing a CLF instruction that persists the memory location.(similar to the method of Witcher)

also, when reporting a extra fence performance bug: we check if there are no preceding flush.



## example 3

**No order guarantee**: one of the bug patterns in paper PMDebugger.

To detect this bug, we should ask the programmer to specify in a debugger configuration file/some specific file that describes constraints of the program, such as: variable 𝑋 must be persisted before 𝑌 and at which application function. 

1. Front-end generate a valid example program: (specific example programs require specific analysis)

```C++
//if we have to make sure x+y = 10
x := A;
y := 10 - x;
//line 2&3 have to be executed atomicly. if crash happens after x := A but before y := 10 - x, there might be persistence atomicity violation.
flush(&x);
flush(&y);
```

2. Replace PM operations with operations to our simulator. (flush, read and write). During the process, the simulator needs to map the variables to their addresses based on symbol tables or getting the dynamic memory allocations.(but don't know about the specific details)
3. when the program executes, the simulator can add fence instruction(for example), and check whether the order constraints is guaranteed by the program.

Question: this type of bug patterns seems to be too general, since too many example programs(such as PM Inter-thread Inconsistency in paper PMRace) maybe can also be classified as No order guarantee (variable *X* must be persisted before *y*).

```C++
Thread 1  | Thread 2
x := A;   | 
          | y := x;
          | clwb(&y);
          | sfence();
#If crash happens here, y != x in PM;
clwb(&x); |
sfence(); |
```





# 小结

**假设我们不知道目前已知的所有bug pattern**, how to automaticly find those bug patterns?

进一步：

使用已有的simplified bug patterns, 先用这些做；（make a design, a design that can be applied）

关于原子破坏的bug，see from the recovery function. 

先交上谷歌云盘；
