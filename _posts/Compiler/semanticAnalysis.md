# 1. Attributes and attribute grammars

## 1.1 Some Conceptions

### A. Attributes

那些是属性？

* The ***data type*** of a variable
* The ***value*** of an expression
* The ***location*** of a variable in memory
* The ***object code*** of a procedure
* The ***numbers*** of significant digits in a number

属性可以在编译过程中确定也可以在程序执行过程中确定

### B. Binding

绑定是计算属性的值并且将属性值绑定到属性名

Binding Time:  绑定发生的时间 compilation/execution



根据绑定时间可以将属性分类

* ***static attributes***: be bound prior to execution
* ***dynamic attributes***: be bound during exectuion



### C. Type check

A type checker的工作

* computes the data type attribute of all language entities for which data types are defined(给一个变量a绑定了整数类型，我们判断给a的值是否是整数类型)
* verifies that these types conform to the ***type rule***s of the language. (比如int和double的数值运算全部转化为double计算)

<br>

{:.warning}

Type checking=set of rules that ensure the type consistency of different constructs in the program

<br>

一些rule的例子：

* The type of a variable must match the type from its  declaration

* The operands of arithmetic expressions (+, *, -,/)must have integer types; the result has integer type

* The operands of comparison expressions (==, !=)  must have integer or string types; the result has boolean type



## 1.2 Attribute grammars

### A. 定义

`X.a`: the value of a associated to `X`

`X`是语法符号,`a`是`X`绑定的一个属性

<br>

所谓***Syntax-directed semantics***: 属性直接和语法符号关联绑定

给定属性$a_1,a_2,\cdots$, 对于一个语法规则$X_0\rightarrow X_1X_2\cdots X_n$($X_0$是非终结符)，$X_i.a_j$和其他符号的属性相关

因为这种相关，我们可以写出数学表达式

$X_i.a_j=f_{ij}(X_0.a_1,\cdots,X_0.a_k,X_1.a_1,\cdots,X_1.a_k,\cdots,X_n.a_1,\cdots,X_n.a_k)$

我们称这样的语法规则为为attribute grammar

<br>

### B. 形式

| Grammar Rule | Semantic Rules                 |
| ------------ | ------------------------------ |
| Rule 1       | Associated attribute equations |
| ...          | ...                            |
| Rule n       | Associated attribute equations |

Example1：

Given grammar rules

$number\rightarrow number\;digit\vert digit$

$digit\rightarrow 0\vert 1\vert 2\vert 3\vert 4\vert 5\vert 6\vert 7\vert 8\vert 9$

| Grammar Rule                        | Semantics Rules                            |
| ----------------------------------- | ------------------------------------------ |
| $number1\rightarrow number2\;digit$ | number1.val = number2.val * 10 + digit.val |
| $number\rightarrow digit$           | number.val = digit.val                     |
| $digit\rightarrow 0$                | digit.val=0                                |
| $digit\rightarrow 1$                | digit.val=1                                |
| $digit\rightarrow 2$                | digit.val=2                                |
| $digit\rightarrow 3$                | digit.val=3                                |
| $digit\rightarrow 4$                | digit.val=4                                |
| $digit\rightarrow 5$                | digit.val=5                                |
| $digit\rightarrow 6$                | digit.val=6                                |
| $digit\rightarrow 7$                | digit.val=7                                |
| $digit\rightarrow 8$                | digit.val=8                                |
| $digit\rightarrow 9$                | digit.val=9                                |

**注意第一行$number1\rightarrow number2\;digit$, 因为左右两边number有重复，因此改写为number1, number2来做一个区分**

give number 345

<img src="../../assets/images/image-20200425192538263.png" alt="image-20200425192538263" style="zoom:50%;" />





## 1.3 Simplifications and Extensions to Attribute Grammars

### A. 一些概念

* attribute grammar的***Metalanguage***(元语言): the collection of expressions allowable in an attribute equation

  * 限制算术型，逻辑型表达式
  * if-then-else表达式，switch-case表达式

* ***Functions***可以被加入到元语言中，他的定义可以在另外的地方

  $digit\rightarrow D$

  用numval函数digit.val=numval(D)



### B. 简化的方法

1. 使用歧义文法(所有歧义都在parser阶段处理了)

<img src="../../assets/images/image-20200425194213074.png" alt="image-20200425194213074" style="zoom:35%;" />

使用歧义文法后转化成

<img src="../../assets/images/image-20200425194042640.png" alt="image-20200425194042640" style="zoom:40%;" />

2. 用抽象语法树来代替语法树

<img src="../../assets/images/image-20200425194122636.png" alt="image-20200425194122636" style="zoom:40%;" />

<img src="../../assets/images/image-20200425194348302.png" alt="image-20200425194348302" style="zoom:40%;" />



# 2. Algorithms for attribute computation

## 2.1 Dependency Graphs and Evaluation Order

### A. dependency graph



$X_i.a_j=f_{ij}(\cdots,X_m.a_k,\cdots)$

An edge from each node $X_m.a_k$ to $X_i.a_j$ expressing the dependency of $X_i.a_j$ on $X_m.a_k$



<img src="../../assets/images/image-20200425232017467.png" alt="image-20200425232017467" style="zoom:50%;" />

Example:



### B. Directed acyclic graphs(DAG)

算法必须计算出dependency graph中每个node的属性值后才能计算后续属性

可以用拓扑排序来计算一下顺序，拓扑排序要求图无环



### C. find attribute values at roots

* Parse tree method:  construction of the dependency graph is based on the specific parse tree at compile time, add complexity and need circularity detective.
* Rule based method: fix an order for attribute evaluation at compiler construction time. It depends on an analysis of the attribute equations, or semantics rules.



## 2.2 Synthesized and inherited attributes

* #### ***synthesized attributes***

在parse tree中从孩子指向父亲的属性是synthesized attributes. 在分析树节点N上的非终结符A的综合属性是由N上的产生式所关联的语义规则来定义的。这个产生式的头一定是A。<u>节点N上的综合属性只能通过N的子节点或N本身的属性值来定义</u>

给定语法规则$A\rightarrow X_1X_2\cdots X_n$, the only associated attribute equation with an `a` on the left-hand side is of the form

$A.a=f(X_a.a_1,\cdots,X_1.a_k,\cdots,X_n.a_1,\cdots,X_n.a_k)$

我们称一个文法为S-attributed grammar, 如果所有属性都是synthesized attributes

***计算***：

可以通过bottom-up, post-order遍历的方式计算

```pascal
procedure postEval(T:treenode);
begin
	for each child C of T do
	postEval(C);
	compute all synthesized attributes of T;
end
```

* ***inherited attributes***

不是synthesized attributes就是inherited attributes

在分析树节点N上的非终结符B的继承属性是由N的父节点上的产生式所关联的语义规则来定义的。这个产生式的体中必含符号B。<u>节点N上的综合属性只能通过N的父节点, N本身和N的兄弟节点上的属性值来定义</u>

<img src="../../assets/images/image-20200425234201883.png" alt="image-20200425234201883" style="zoom:40%;" />

***计算***：

可以通过preorder, combined preorder/inorder 遍历的方式计算

```pascal
procedure preEval(T:treenode)
begin
	for each child C of T do
		computed all inherited attributes of C
	preEval(C)
end
```

Remark:

* The order in which the inherited attributes of the children are computed is important.

* It must adhere to any requirements of the dependencies

<br>

## 2.3 Attributes as parameters and returned values

* Inherited attributes: 经常是通过preorder遍历计算，所以将其作为调用的参数
* synthesized attributes: 经常通过postorder遍历计算，所以将其作为return values



# 3. The symbol table

为什么要有符号表：这是一个外部存储结构，存储了符号的相关信息

符号表包含的内容

* the name of an identifier
* additional information: kind, type, constant

| NAME | KIND | TYPE               | ATTRIBUTES |
| ---- | ---- | ------------------ | ---------- |
| foo  | fun  | int x int --> bool | extern     |
| m    | arg  | int                |            |
| n    | arg  | int                | const      |
| tmp  | var  | bool               | const      |

## 3.1 The structure of the symbol table

符号表可以有多种实现方式

### A. Linear List

* insert: constant time
* look: linear time
* delete: linear time

线性表实现简单，但是效率不高



### B. Various Search Tree Structures

binary search tree, AVL tree, B tree

这种实现效率不是最高的

删除操作非常复杂



### C. Hash Tables

insert, look, delete几乎都可以在常数时间完成

The size of the hash table: 几百到1000， 并且bucket array的长度应该选择一个素数



#### The process of the hash function

1. Converts a character string(the identifier name) into a nonnegative integer.
2. These integers are combined in someway to form a single integer
3. The result integer is scaled to the range 0,... ,size-1

<br>

#### The algorithm of the hash function

repeatedly use a constant number $\alpha$ as a multiplying factor when adding in the value of the next character.

$h_{i+1}=\alpha h_i+c_i,h_0=0$

final hash value $h=h_n\mod{size}$

n是这个string中字符的数量

$h=(\alpha^{n-1}c_1+\alpha^{n-2}c_2+\cdots+\alpha c_{n-1}+c_n)\mod {size}$

$\alpha$的选择会影响效率，一般选择$\alpha$为2的幂次，这样的话可以用移位实现乘法，效率更高。



## 3.2 Declarations

分类

* 常量声明
* 类型声明
* 变量声明
* procedure/function声明



### A. Constant Declarations

Value binding: associate values to names

The values that ca be bound determine how the compiler treats them

* Pascal, Modula-2: the values in a constant declaration be **static**, computable by the compiler
* C, Ada: permit constants to be **dynamic**, only computable during execution

### B. Type Declarations

Bind names to newly constructed types and may also create aliases for existing named types

Type names are used in conjunction with a type equivalence algorithm(perform type checking of a program)

### C. Variable Declarations

Bind names to data types

变量声明时可能绑定了其他隐藏属性，比如变量作用域

### D. Procedure/Function Declarations





### E. The strategies

* 用一张symbol table来存储所有类型的声明
* 用不同的symbol table来存储不同类型的声明
* symbol table对应程序的不同区域，并且将这些symbol table根据文法规则连接起来



## 3.3 Scope rules and block structure

two rules:

* Declarations before use
* The most closely nested rule for block structure

### A. Declaration Before Use

* 符号表在parsing过程中建立

* lookups to be performed as soon as a name reference is encountered in the code

  * If the lookup fails
    * a violation of declaration before use has occurred
    * the compiler will issue an appropriate error message

  ### B. The most closely nested rule for block structure

  Block: any construct that can contain declarations, such as procedure/function declarations.

  

  * Pascal
    * the blocks are the main program
    * pr

  

```cpp
int i, j;
int f(int size) 
{
  {
    char i, temp; ...
  }
  {
    double j; ...
  }
  {
    char* j; ...
  }
}
```



## 3.4 Interation of same-level declarations
