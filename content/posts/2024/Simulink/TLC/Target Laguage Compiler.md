
model.rtw 结构体如下：

![](Target%20Laguage%20Compiler.png)

对于模型中的每个给定块的出现，模型中存在一个相应的块记录.rtw文件，系统目标文件TLC代码循环遍历块记录，并未该块类型调用相应块目标中的函数。

有一种方法可以将特定于块的信息（内部块信息，与输入，输出，参数相反）。mdlRTW还允许写参数设置（ParamSettings），与此块有关的唯一信息，对于块TLC文件的参数设置，可以从块TLC代码直接访问这些字段，并且可以根据需要修改生成的代码。

当非内联S-Function执行时，需要调用MEX文件。调用过程如下；

![](Target%20Laguage%20Compiler-1.png)


输出代码如下：
![](Target%20Laguage%20Compiler-2.png)

完全内联函数可以将代码完全嵌入到生成代码中，不需要多余的调用关系。

![](Target%20Laguage%20Compiler-3.png)


并且可以生成自定义代码，包括可以调用自定义C语言函数。
![](Target%20Laguage%20Compiler-4.png)


生成的代码如下：
![](Target%20Laguage%20Compiler-5.png)


## 3. 目标编译器教程

### 3.1 使用TLC读取记录

记录的格式为 `recordName {itemName  itemValue}`，其中 `itemName` 可以 是数字或者字符串，数字可以是标量、向量、矩阵。可以有多个项目，以空格、TAB、或者返回字符分割。



在模型的RTW文件中，最顶部的记录名称是 `CompiledModel` 每个块由其中的自己录表示。可以参考 `matlabroot/toolbox/rtw/rtwdemos/tlctutorial/guide` 。

- 解释记录
例如有一个如下的记录文件

```matlab
Top {                   # Outermost Record, called Top 
  Date "21-Aug-2008"    # Name/Value pair named Top.Date 
  Employee {            # Nested record within the Top record 
    FirstName "Arthur"  # Alpha field Top.Employee.FirstName 
    LastName "Dent"     # Alpha field Top.Employee.LastName 
    Overhead 1.78       # Numeric field Top.Employee.Overhead 
    PayRate 11.50       # Numeric field Top.Employee.PayRate 
    GrossRate 0.0       # Numeric Field Top.Employee.GrossRate 
  }                     # End of Employee record 
  NumProject 3          # Indicates length of following list 
  Project {             # First list item, called Top.Project[0] 
    Name "Tea"          # Alpha field Name, Top.Project[0].Name 
    Difficulty 3        # Numeric field Top.Project[0].Difficulty 
  }                     # End of first list item 
  Project {             # Second list item, called Top.Project[1] 
    Name "Gillian"      # Alpha field Name, Top.Project[1].Name 
    Difficulty 8        # Numeric field Top.Project[1].Difficulty 
  }                     # End of second list item 
  Project {             # Third list item, called Top.Project[2] 
    Name "Zaphod"       # Alpha field Name, Top.Project[2].Name 
    Difficulty 10       # Numeric field Top.Project[2].Difficulty 
  }                     # End of third list item 
}                       # End of Top record and of file
```

下面读取文件内容：

```matlab
# 读取字符串
%<Top.Date>     # "21-Aug-2008"
# 局部变量
%assign worker = Top.Employee.FirstName    # "Arthur"
# 全局变量
::global_worker = worker + " " + Top.Employee.LastName   # "Authur Dent"
# 设置值
Top.Employee.GrossRate = 10.0
# 跨行语句，使用 ... 连接
%assin projects = Top.Project[0].Name + ', ' + Top.Project[1].Name ...
                 + ", " + Top.Project[2].Name   # Tea, Gillian, Zaphod"

```


- TLC脚本的解剖

下面介绍TLC脚本的每个部分：
1)  `Codinng Conventions`  编码约定
2)  `File Header` 头文件和格式化指令
3)  `Tokee Expasion`  计算字段和变量标识符
4)  `General Assignment`  使用 %assign 指令
5)  `Arithmetic Operations` 字段和变量的计算
6)  `Modify Records` 更改、复制、追加记录
7)  `Index Lists` 引用带有下标的列表元素
8)  `Loop Over Lists`  循环结构和行为的详细信息。

下面来详细解释每个部分

1)  `Codinng Conventions`  编码约定

下面是一些基本的TLC语法和编码约定：
```
%% comment               TLC注释，用%%来隔开，不会输出到文件
/* comment */            注释，可以输出到文件
%keyword                 TLC 变量，以 '%' 开头
%<expr>                  计算语句，可以作为变量执行运算
.                        作用域操作符，用来选择内部变量
...                      语句继续。不输出换行符。可以连接下一行，必须在行尾，后不可以有字符.
\                        语句继续。输出换行符。参考 `...`
locl_var_identifier      局部变量
global_var_identifier    全局变量
record_identifier        记录标识符，全部大写
EXISTS                   内置函数，全部大写
```


2)  `File Header` 头文件和格式化指令

```matlab
%% 关键字 readformat 控制后续文件浮点数如何格式化
%realformat "CONCISE"
```

3)  `Tokee Expasion`  计算字段和变量标识符

```matlab
%% 定义一个变量。一般定义变量的方式为 %assign ::variable=exp
%assign td = "%" + "<Top.Date>" 

%% 显示。  td是一个字符串，%<var>可以取变量的值。 td的值为字符串： "%<Top.Date>"
%% 变量不支持尖括号双重嵌套 %<foo%<bar>>
%<td> -- evaluates to: 
"%<Top.Date>"
```

4)  `General Assignment`  使用 %assign 指令

```matlab
%% 创建一个局部变量 worker 并为其赋值
%assign worker = Top.Employee.FirstName

%% 输出字符串，并不会创建变量
"%assign worker = Top.Employee.FirstName"

%% 使用变量
worker expands to Top.Employee.FirstName = %<worker>
```

5)  `Arithmetic Operations` 字段和变量的计算

```matlab
%% 创建变量
%assign wageCost = Top.Employee.PayRate * Top.Employee.Overhead
%% 回显
"%assign wageCost = Top.Employee.PayRate * Top.Employee.Overhead"
%% 字符串使用变量，需要使用 %<var>
wageCost expands to Top.Employee.PayRate * Top.Employee.Overhead ...
%<Top.Employee.PayRate> * %<Top.Employee.Overhead> = %<wageCost>
```

6)  `Modify Records` 更改、复制、追加记录

记录一旦被读入到内存，就可以像修改变量一样，进行读、写。

| 符号          | 描述                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------- |
| %createrecord | 创建新的顶级记录，还可以在其中指定子记录，包括名称，值对                                  |
| %addtorecord  | 向现有记录添加字段。西字段可以是名称/值对或下你有记录的别名                               |
| %mergerecord  | 合并一条或几条记录，第一个记录按照顺序包含其本身以及命令指定的其他记录内容的副本          |
| %copyrecord   | 像 %createrecord 一样创建一条新纪录。                                                     |
| %undef var    | 从作用域中移除 var （变量或者记录)。如果是变量/记录，则直接删除。如果是列表，则删除第一个 |



7)  `Index Lists` 引用带有下标的列表元素

记录文件可以包含具有相同标识符的列表或记录序列。

8)  `Loop Over Lists`  循环结构和行为的详细信息。

一般情况下，列表除了有记录 record 外，还有一个列表尺寸的变量。
```matlab
NumProject 3
Project {
  Name "Tea"
  Diff  3
}
Porject {
  Name "Gillian"
  Diff  8
}
Project {
  Name  "Zaphod"
  Diff  10
}
```
同样，可以使用TLC内置函数 SIZE() 来确定指定范围的记录数量，从而确定列表的长度。
循环读取Project，并输出显示。
```matlab
%assign diffSum = 0.0 
%foreach i = Top.NumProject 
  - At top of Loop, Project = %<Top.Project[i].Name>; Difficulty =... 
    %<Top.Project[i].Diff> 
  %assign diffSum = diffSum + Top.Project[i].Diff
  - Bottom of Loop, i = %<i>; diffSum = %<diffSum> 
%endforeach 
%assign avgDiff = diffSum / Top.NumProject 
Average Project Difficulty expands to diffSum / Top.NumProject = %<diffSum> ...
/ %<Top.NumProject> = %<i>


%% 显示的结果如下
- At top of Loop, Project = Tea; Difficulty = 3 
  - Bottom of Loop, i = 0; diffSum = 3.0 
  - At top of Loop, Project = Gillian; Difficulty = 8 
  - Bottom of Loop, i = 1; diffSum = 11.0 
  - At top of Loop, Project = Zaphod; Difficulty = 10 
  - Bottom of Loop, i = 2; diffSum = 21.0 
Average Project Difficulty expands to diffSum / Top.NumProjects = 21.0 / 3 = 7.0
```

foreach 在变量列表时，下标从0开始。


## 修改 read-guide.tlc

### 传递和使用参数

可以使用TLC命令和内置函数将参数从命令行传递到正在在执行的TLC文件，最常用的命令开关是 `-a`，它可以分配任意变量。

`tlc -r input.rtw -avar=1 -afoo="abc" vars.tlc`

通过 `-a` 传递字符串变量方式与TLC文件内部声明和初始化局部变量的结果相同。
`%assign var=1   %assign foo="abc"`

如果不需要在TLC文件中声明这样的变量，当使用 `-a` 设置时，参数依旧可用。但是如果代码中没有定义，且调用TLC文件也没有传递参数，则会导致错误。


### 审查

以下几个指令，在TLC文件中也比较常用

| 指令                 | 描述                                                                           |
| -------------------- | ------------------------------------------------------------------------------ |
| %addincludepath      | 使能TLC的寻找头文件路径                                                        |
| %addtorecord         | 向现有记录添加字段。新字段可以时名称/值对或现有记录的别名                      |
| %assign              | 创建或修改变量                                                                |
| %copyrecord          | 拷贝记录                                                                       |
| %createrecord        | 创建新的TOP记录，如果适用，在记录中指定子记录，包括名称/值对。                 |
| %foreach/%endforeach | 从0到上限迭代循环变量                                                          |
| %if/%endif           | 控制代码是否执行，如在C中                                                      |
| %mergerecord         | 合并一条或多条记录。第一个记录按顺序包含其本身以及命令指定的其他记录内容的副本 |
| %selectfile          | 直接输出到流或者文件                                                         |
| %undef var           | 从作用域删除变量                                                               |
| %with/%endwith       | 添加作用域以简化引用块。                                                                                |


### 带TLC的内联S-Function


- 什么是内联S-Fuction

- 为什么要使用TLC内联函数


代码生成器包含一个通用API，可以调用用户编写的算法和驱动程序，该API包括各种回调函数，用于初始化、输出、派生、中止等等。
内联S-Function使用MEX编译后，可以加快API调用的速度，并且可以通过代码生成方式，嵌入到yoghurt编写的算法模块中。


- 创建一个内联哈数

只要TLC见到到与S函数同名的TLC文件，它就会创建一个内联的S-Function。通过以下步骤来了解这个过程是如何工作的。

1. 复制 `tlctutorial/timesN/rename_timesN.tlc`，并重命名为 `timesN.tlc`。内容如下：
```tlc
%implements "timesN" "C" 
%% Function: Outputs =========================================================== 
%% 
%function Outputs(block, system) Output 
  %assign gain = SFcnParamSettings.myGain 
  /* %<Type> Block: %<Name> */
  %% 
  /* Multiply input by %<gain> */ 
  %assign rollVars = ["U", "Y"] 
  %roll idx = RollRegions, lcv = RollThreshold, block, "Roller", rollVars 
    %<LibBlockOutputSignal(0, "", lcv, idx)>  = \
    %<LibBlockInnputSignal(0, "", lcv, idx)> * %<gain>; 
  %endroll 
%endfunction
```

2. 配置sfun_xN_ilp.模型，生成代码。文件 一般被放在 `model_grt_rtw` 内，代码如下：
``` C
/* Model output function */ 
static void sfun_xN_ilp_output(int_T tid) {
  /* Sin: '<Root>/Sin' */ 
  sfun_xN_ilp_B.Sin = sin(sfun_xN_ilp_M->Timing.t[0]); 
  /* S-Function Block: <Root>/S-Function */ 
  /* Multiply input by 3.0 */ 
  sfun_xN_ilp_B.timesN_output = sfun_xN_ilp_B.Sin * 3.0; 
  /* Outport: '<Root>/Out' */ 
  sfun_xN_ilp_Y.Out = sfun_xN_ilp_B.timesN_output; 
  UNUSED_PARAMETER(tid); 
}
```

如果输入的参数是一个向量，那么生成的代码可能需要循环结构。下面来介绍一种适用于循环结构的关键字 `%roll`

> %roll  适用于需要循环处理输入信号的方式
格式： %roll sigIdx = RollRegions, lcv = RollThreshold, block, "Roller", rollVars

| 参数     | 描述                                                                                                               |
| -------- | ------------------------------------------------------------------------------------------------------------------ |
| sigIdx   | 向量的索引，从0开始                                                                                                |
| lcv      | 通常在 %roll 指令中，指定的控制变量，如 lcv=RollThreshold；一般RollThreshold默认值为5，超过5个循环，则使用滚动方式 |
| block    | 告诉TLC正在对块对象进行操作                                                                                        |
| “Roller" | 在 `rtw/c/tlc/roller` 中指定                                                                                       |
| rollVars | 告诉TLC什么了欸行的项目应该循环                                                                                    | 


## 调试TLC代码

## 用TLC包装自定义代码

Simulink 用户想要好构架包含他们已经用高级语言编码、实现和测试过的算法模型。可以采用以下几种方式：
- 将用户代码转成S-Function，并于泛型API挂钩。这是最简单的方法，但是会牺牲效率，并且需要对S-Function比较熟悉。
- 可以内联 S-Function，将其重新实现为TLC文件。但是需要花费时间和经历，且可能会引入错误到生产代码，并且每个算法文件都需要维护两套。
- 可以通过TLC包装器内联S-Function，这样，只需要少量的TLC代码，且算法可以保持原有语言不动。


下图中，说明了 S-Function 的工作原理
![](Target%20Laguage%20Compiler-6.png)


要为外部函数 `my_alg.c` 创建包装器，可以构造一个包含其调用函数 warpsfcn.c 的 TLC 文件。TLC文件必须可以生成C代码：
- 返回double类型，并输入double类型的外部函数的函数原型声明
- 对代码outpus部分添加 my_alg() 调用。

要为 my_alg() 创建包装器，请执行以下操作：
1. 在 matlab 中打开 change_wrapsfcn.tlc 文件，在有注释标记的地方添加代码行，以创建可用的包装器。
2. 将编辑后的文件另存为 `wrapsfcn.tlc` ，注意: 必须与使用它的文件名称保持一致。
3. 打开 `externalcode` 模型，可以验证结果。

wrapsfcn.tlc 文件内容如下：

``` tlc
%% File : wrapsfcn.tlc 
%% Abstract: 
%%   Example tlc file for S-function wrapsfcn.c 
%% 
%% Copyright 1994-2002 The MathWorks, Inc. 
%% 
%% 
%implements "wrapsfcn" "C" 
%% Function: BlockTypeSetup ================================ 
%% Abstract: 
%%   Create function prototype in model.h as: 
%%   "extern double my_alg(double u);" 
%% 
%function BlockTypeSetup(block, system) void 
  %openfile buffer 
    %% ASSIGNMENT: PROVIDE A LINE OF CODE AS A FUNCTION PROTOTYPE 
    %% FOR "my_alg" AS DESCRIBED IN THE WRAPPER TLC ASSIGNMENT 
    extern double my_alg(double u); 
  %closefile buffer 
  %<LibCacheFunctionPrototype(buffer)>
%endfunction 
%% BlockTypeSetup 
%% Function: Outputs ======================================= 
%% Abstract: 
%%   y = my_alg( u ); 
%% 
%function Outputs(block, system) Output 
  /* %<Type> Block: %<Name> */ 
  %assign u = LibBlockInputSignal(0, "", "", 0) 
  %assign y = LibBlockOutputSignal(0, "", "", 0) 
  %% PROVIDE THE CALLING STATEMENT FOR "wrapfcn" 
  %<y> = my_alg( %<u> ); 
%endfunction %% Outputs
```



## 代码生成架构


### 构建过程

TLC是以编译目标语言来编写的文件。目标 语言是一种解释型语言，编译器每次执行（不需要重新编译）都会对源文件进行操作。因此，如果TLC文件中分支语句，只有在执行到，才会有报错信息。例如：

```tlc
%if 1
  Hello
%else
  <Invalid_function_call()>
%endif
```

尽管上面代码中第4行是一个无效函数调用，但是由于分支无法到达，所以执行TLC文件时，永远不会出错。


### 创建和使用 TLC 文件

1. 创建以下模型，并保存为 basic.mdl
![](Target%20Laguage%20Compiler-7.png)

2. 配置模型参数为固定步长-0.1，仿真时间为10秒
3. 点击生成代码按钮，等待代码生成。

完成代码构建后，会生成 basic_grt_rtw 的文件夹，模型生成的C语言源文件保存的位置。

如果不想打开模型，也可以通过 slbuild("basic) 来生成代码。

### 查看 basic.rtw 文件

basic.rtw 文件包含已标记的记录和字段，每条记录用方括号风格，并包含从属记录和字段，标签说明了每个记录和字段的用途。

接下来尝试创建一个名为 basic.tlc  的文件，其内容包括：

```tlc
%with CompiledModel
  My model is called %<Name>
  It was generated on %<GeneratedOn>
  It has %<NumModelOutputs> output(s) and %<NumContStates> continuuous state(s).
%endwith
```

在 matlab 窗口执行 `slbuild('basic');tlc -r basic_grt_rtw/basic.rtw basic.tlc -v`，可以输出以下内容

```
My model is called basic. 
It was generated on Wed Jun 12 20:51:11 2024. 
It has 1 output(s) and 0 continuous state(s).
```



### 配置TLC

可以使用 `set_param` 命令，模型参数 `TLCOptions` 和 TLC 选项 `-a` 从MATLAB命令行输入TCL命令行参数。例如，要输入TLC命令行字符串 `-amyConfiggVariable=1`，请使用以下命令：
`set_param(modelName, "TLEOptions", "-amyConfigVariable=1");`

TLC选项 `-amyConfigVariable` 就等于在TLC文件中创建变量 `%assign myConfigVariable = 1;`


### 代码生成概念

TLC作为一种语言，目的只有一个：将model.rtw的内容完全转化为目标可以使用的代码。因此，TLC提供了许多对完成代码转换相关的函数功能。


#### 输出
输出 ”Hello, world!" 是我们作为了解一门语法的第一个用例。

```tlc
%selectfile STDOUT
Hello, world!
```

执行 `tlc hello.tlc` 可以看到命令行输出 "Hello, world!" 的语句。

STDOUT表示输出缓冲区位于标准输出（显示器）。同样，我们可以使用 `selectfile` 执行其他文件作为缓冲区输出。例如：

```tlc
%openfile hello = "hello.txt"

%selectfile hello

Hello, world!

%closefile hello
```


#### 记录

与从 model.rtw 文件生成代码最相关的结构之一是 record。记录类似于C中的结构或Pascal 中的记录。

记录声明的语法是：

```tlc
%createrecord recVar {...
   field1 value1 ...
   field2 value2 ...
   field3 value3 ...
}
```

`recVar` 作为记录的名称，`field` 作为字符串，`value` 作为对应的 TLC 值。


记录中可以包含嵌套记录或子记录。`model.rtw` 文件，实质上是一条名为 `CompiledModel` 的记录，其中包含各种子记录。

需要注意，model.rtw 可能会由于 matlab版本的不同，而生成文件不同。

TLC提供了一个 %with 指令，以便于使用record。


记录变量使用方式； 1. 使用 `.` 来选择子记录。  2. 使用下标来调用数组内的数据。

在TLC中，可以创建记录的别名。别名类似于C语言的指向结构体的指针，可以作为单个记录创建多个别名。对别名的修改，在每个记录中都会同步。

例如：
```tlc
%createrecord var1 {field 1}
%createrecord a {}
%createrecord b {}
%createrecord c {}

%addtorecord a var1 var1
%addtorecord b var1 var1
%addtorecord c var1 {field 1}

%assign var1.field = 2

%selectfile STDOUT

ISALIAS(a.var1) = %<ISALIAS(a.var1)>
ISALIAS(b.var1) = %<ISALIAS(b.var1)>
ISALIAS(c.var1) = %<ISALIAS(c.var1)>

a.var1.field = %<a.var1.field>
b.var1.field = %<b.var1.field>
c.var1.field = %<c.var1.field>



%% 输出结果
%%
%% >> tlc ./hello.tlc
%%
%% ISALIAS(a.var1) = 1
%% ISALIAS(b.var1) = 1
%% ISALIAS(c.var1) = 0
%% 
%% a.var1.field = 2
%% b.var1.field = 2
%% c.var1.field = 1
```

在函数内部时，可以创建函数内记录的别名。如果别名是从函数返回的，则即使在退出函数后，别名仍有效。

```tlc
%selectfile STDOUT

%function func (value) Output
  %createrecord var { field value }
  %createrecord a { var var }

  ISALIAS(a.var) = %<ISALIAS(a.var)>

  %return a.var
%endfunction

%assign x = func(2)
ISALIAS(x) = %<ISALIAS(x)>
x = %<x>
x.field = %<x.field>


%% 输出
%%  
%% ISALIAS(a.var) = 1
%%
%% ISALIAS(x) = 1
%% x = { field 2 }
%% x.field = 2
```

只要通过别名对记录进行一些引用，就不是删除记录。这允许将记录作为函数的返回值。


在代码生成的上下文中，有两种类型的目标文件：
- 系统目标文件
  系统目标文件确定代码生成的总体框架，何时执行数据块，如何记录数据等
- 组织目标文件
  每个块都有一个目标文件，用于确定应该为该块生成什么代码。代码可能会有所不同，剧吐取决于块的参数与连接类型，输入输出参数。

#### 模型目标文件和系统目标文件


系统目标文件是TLC的入口。类似与C语言的 `main` 函数。系统会监控整个代码生成过程。例如，系统目标文件`grt.tlc` 就为 `codegenertyry.tlc` 设置了一些变量。

#### 系统目标文件

整个代码生成过程，从在 `Code Generation` 窗口的 `Configuration Parameters` 对话框中指定的单个系统目标文件开始。

```tlc
%selectfile NULL_FILE
%assign TargetTyep = "RT"
%assign Language = "C"
%assign MatFileLogging = 1

%include "codegenentry.tlc"
```

`Language, TargetType, MatFileLogging` 三个变量是其他函数使用的TLC全局变量，通过调用 `codegenentry.tlc` 来启动代码生成。


如果要修改整体代码生成，则必须更改系统目标文件。初始设置后，可以调用自定义的 TLC 文件，而不直接调用  `codegenentry.tlc` 。

```tlc
%% 设置变量，等待加载其他函数
%include "mylib.tlc"
%include "codegenentry.tlc"
```

在 `mylib.tlc` 内，可以定义自己的TLC变量（全局变量），供自定义模块来使用。



## model.rtw 文件和编写 S-Function 和数据对象

### model.rtw 文件和范围

代码生成软件会从 Simulink 模型创建 `model.rtw` 文件，该文件是由构建过程生成的模型部分形式，供 Target Language Compiler 使用，它描述了相应模型文件的输入、输出、模块、参数、状态、存储及其他属性。

`model.rtw` 文件是作为存储在记录层次结构中的【参数对】的ASCII字符，参数名称/值固定为 【参数 值】的形式。

妹妹个记录都会创建一个新范围， model.rtw 文件使用大括号 `{, }` 来标记记录的范围。本实例的文件以 `CompiledModel` 作为第一个记录，形式如下：

```rtw
CompiledModel { 
  Name "modelname"         -- Example of a parameter-value 
  ...                         pair (record field). 
  System {                 -- There is one system for each nonvirtual subsystem. 
    Block {                -- Block records for each 
      Type "S-Function"       nonvirtual block in the system. 
      Name "S-Function"   
      ... 
      Parameter { 
        Name "P1" 
        Value Matrix(1,2) [[1, 2];] 
      } 
      ... 
      Block {
      } 
    } 
    ... 
    System {               -- The last system is for the root of your model.
    } 
}
```


访问记录内容的方式为 `.`，例如： `CompiledModel.Name`。多条同名记录作为列表使用，使用下表访问，例如： `CompiledModel.System[0]`，下标从0开始。

```ad-note
可以使用 %with 来简化记录的使用
%with CompilerModel
  %% var = "S-Function"
  %assign var = Name
%endwith
```

### model.rtw 中的数据对象信息

在模型构建代码过程中，会将和Simulink信号与参数相关的 信息写入 `model.rtw` 内。为满足特定条件的每个参数或信号写入带有 `CoderInfo` 属性信息的 `Object` 记录。

对于某些数据，自定义存储类可能是一种有用的方式。对于该部分，可以参考 《通过为存储类编写TLC代码来惊喜控制数据表示》章节。

对于满足以下条件的每个参数，model.rtw 文件的 ModelParameters 部分中都包含带有 CoderInfo 属性信息的 Object 记录。

- 参数解析为 Simulink.Parameter 数据对象
- 参数符号保留在生成代码中。当数据对象的 CoderInfo.StorageClass 属性未设置为 Auto 时，或者在 Code Mapping Editor 中，将相对应数据类型设置为 Default，则保留该符号。

### 信号的对象记录

具有 CoderInfo 属性信息的 Object 记录包含在 model.rtw 文件中的 ExternalOutputs、ExternalInputs、BlockOutputs部分。对于其符号保留在生成代码中的每个符号（包括root-level的inport与Outport模块），当信号使用为非Auto存储类时，将保留该原件。生成的代码其有效性和唯一性被强制执行，并且其符号保留。

```rtw
ExternalOutputs { 
   ... 
   NumExternalOutputs 1 
   ... 
   ExternalOutput { 
     ArgSrc Y0 
     Block [1,3] 
     BlockName "/Out1" 
     Identifier "output" 
     OrigIdentifier "output" 
     StorageClass "ExportedGlobal" 
     ResolvedToSignalObject embedded 
     HasObject 1 
     Object { 
       Package Simulink 
       Class Signal 
       ObjectProperties { 
         CoderInfo { 
           Object { 
             Package Simulink 
             Class CoderInfo 
             ObjectProperties { 
               StorageClass "ExportedGlobal" 
               TypeQualifier "" 
               Alias "" 
               Alignment -1 
               CSCPackageName "Simulink" 
               ParameterOrSignal "Signal" 
               CustomStorageClass "Default" 
               CustomAttributes { 
                 Object { 
                   Package SimulinkCSC 
                   Class AttribClass_Simulink_Default 
                   ObjectProperties { 
                   }
                 } 
              } 
            } 
          }
        }
      }
    }
  }
}
```

### 通过TLC访问数据对象信息

以下代码片段迭代 model.rtw 文件的 ModelParameters 部分的 Parameter 结构，并从遇到的参数 Object 中提取信息。

```tlc
%with CompiledModel.ModelParameters 
  %foreach modelParamIdx = NumParameters
    %% 因为 %<> 无法嵌套，所以需要创建变量形式来去数组内数据
    %assign thisModelParam = Parameter[modelParamIdx] 
    %assign paramName = thisModelParam.Identifier
    %% 判断是否带有该属性，防止出现调用错误
    %if EXISTS("thisModelParam.Object.ObjectProperties") 
      %with thisModelParam.Object.ObjectProperties 
        %assign valueInObject = Value 
        %with CoderInfo.Object.ObjectProperties 
          %assign storageClassInObject = StorageClass 
        %endwith 
        %% *********************************** 
        %% Access user-defined properties here 
        %% *********************************** 
        %if EXISTS("MY_PROPERTY_NAME") 
          %assign userDefinedPropertyName = MY_PROPERTY_NAME 
        %endif 
        %% *********************************** 
      %endwith 
    %endif 
  %endforeach 
%endwith
```

### 访问信号对象记录

以下代码片段迭代 model.rtw 文件的 BlockOutputs 部分中的 ExternalBlockOutput 结构，并从遇到的信号对象记录中提取信息。

```tlc
%with CompiledModel.BlockOutputs 
  %foreach blockOutputIdx = NumExternalBlockOutputs 
    %assign thisBlockOutput = ExternalBlockOutput[blockOutputIdx] 
    %assign signalName = thisBlockOutput.Identifier 
    %if EXISTS("thisBlockOutput.Object.ObjectProperties") 
      %with thisBlockOutput.Object.ObjectProperties 
        %with CoderInfo.Object.ObjectProperties 
          %assign storageClassInObject = StorageClass 
        %endwith  
        %% *********************************** 
        %% Access user-defined properties here 
        %% *********************************** 
        %if EXISTS("MY_PROPERTY_NAME") 
          %assign userDefinedPropertyName = MY_PROPERTY_NAME 
        %endif 
        %% *********************************** 
      %endwith 
    %endif 
  %endforeach 
%endwith
```

### 数据引用

model.rtw 文件中的某些记录（例如，对于参数和常量IO的记录）可以嵌入非常大的数据值向量。此类向量可能会在代码生成期间，占用大量内存开销，因此在此过程中，这些值必须作为文本保留在内存中。

为了避免此类开销，默认情况下，Simulink软件不会将真个数值向量写到 model.rtw 中。相反，它会编写一个成为“数据引用”的 key ，该键可以在代码生成期间使用，一边直接从Simulink访问数据。如果数据在代码生成过程总没有改变，则在写包含数据值的实际代码时，会有效的流式传输到磁盘，进行保存。

默认情况下，Simulink会写入 model.rtw 的数据引用，以代替长度大于10的数据向量。如需更改，执行以下语句：
`set_param(0, 'RTWDataReferencesMinSize`, maxlen)`

### 访问 model.rtw 的库函数异常


有几个库函数可以提供对模块输入、输出、参数、采样时间和其他信息的访问。建议使用这些库函数来访问和配置库记录的参数值对，而不是通过TLC代码，直接读写参数名称/值对。


C MEX S-Function 中的 mdlRTW 函数可以来访问和设置RTW的参数值对。

```C
static void mdlRTW(SimStruct *S) 
{ 
  // 写入一个参数
  // 操作符 - &&
  if (!ssWriteRTWParamSettings(S, 1, 
                            // 类型                键（字符串形式）   值
                              SSWRITE_VALUE_QSTR, "Operator", "AND")) 
  { 
    ssSetErrorStatus(S,"Error writing parameter data to .rtw file"); 
    return; 
  } 
}
```

model.rtw 中生成的块记录为：

```rtw
Block { 
  Type "S-Function" 
  Name "<Root>/S-Function" .
  .. 
  SFcnParamSettings { 
    Operator "AND"
  }
}
```

可以使用TLC文件来访问参数，辅助生成代码：

```tlc
%function Outputs(block, system) Output 
  %% 
  %% Select Operator 
  %switch(SFcnParamSettings.Operator) 
    %case "AND" 
      %assign LogicOp = "&" 
      %break 
    ... 
  %endswitch 
%endfunction
```

当调用块目标文件中的参数时，它们将作为参数传递给实例的块记录和系统记录。第一个参数 block 参数，在 scope 内，这意味着实例的块记录的变量名可以通过名称直接访问，如： SfcnParamSettings.Operator。



## 6. 指令和内置函数

可以通过编写或修改应用TLC指令和内置函数脚本来控制从模型生成代码的方式，使用以下部分作为目标语言机构的语法和格式，以及MATLAB tlc命令本身的参考。

### 目标语言编译器指令

目标语言指令必须时一行中的第一个非空字符，并以 `%` 开头。

`%` :  TLC语法
`%%` : TLC注释

其余都会传递给输出流。


### 语法

TLC 文件一般有以下任意形式语句组成：
- [ text | % < expression > ] *    文本或者语句

文本将以原样格式传递到输出流，包括空行、空格、tab。  在输出之前，会对 表达式 进行计算。


- %keyword 参数

%keyword 表示TLC的指令之一，参数表示后续表达式。例如： %assign var = 1



```tlc
%% 单行注释

/% 
	多
	行
	注
	释
%/

%% 执行matlab语句，并返回结果
%matlab disp(2.8)

%% 语句表达式
%% 如果是到输出流，则会对表达式进行计算
%<expr> 
%% 表达式，不支持嵌套计算，例如
%assign a = [1, 2, 3]
%assign b = 2
%aasign c = 3
%% 不被允许
%<c> = %<a[%<b>]>
%% 可以在定义时进行赋值
%assign c = a[b]
%% 语句表达式，对字符串会先进行计算操作
%assign c = a["%<b>"]
%% 避免在字符串外使用表达式
%assign c = a[%<b>]  %% 会出现错误

/% 
  %if expr 
  %else if expr
  %else
  %endif
%/
%if EXISTS(a)
a = %<a>
%endif

%if EXISTS("%<b>")
b = %<b>
%endif

%if !EXISTS(d)
d not exist.
%endif

%if 1 || EXISTS(d)
d 不存在，但是逻辑计算会有"短路"运算。
%endif


/%
  %switch expr
    %case case1
      %break

    %case case2
      %continue

	%default
	  %break

  %endswitch

TLC 的 %switch 语句类似于C语言的 switch 语句，expr和case语句只可以是标量（整数）,
%/


/%
  %with expr
  %endwith

%/


/%
  %setcommandswitch string

  更改参数字符串只当的命令行开关的值。仅支持以下开关：
  v, m, p, O, d, r, I, a

  
%/


/%
  %assert expr

  执行 expr 表达式的值，如果结果为 false ，TLC就会发出错误消息、堆栈跟踪并退出，否则继续执行程序。
  如果需要使用断言，需要执行 set_param(model, "TLCAssertion", "on|off"); 或者获取设置 get_param(model, "TLCAssertion");
%/


/%
  错误
  %error tokens

  警告
  %warning tokens

%/


/%
  %assign
  创建创建变量。变量一般由字符构成，以字母、'_'为开始，可以包括数字、字母、'_'。 
  :: 可以将变量修改为全局变量，跨越函数和文件。

%/


/%
  %createrecord
  %addtorecord  添加记录，作为浅拷贝，不进行深度拷贝。 可以理解为指针。
  %copyrecord   拷贝记录，作为深拷贝。
  %mergerecord  将一个或多个记录添加到另一个记录中，第二条记录被深度复制
%/


/%
  %realformat    指定如何设置实数变量的格式
  %language      必须出现在第一次执行 GENERATE 或 GENERATE_TYPE 函数之前。一般设置为 %language "C"
  %implements    通过%generatefile 时，需要放在特定的TLC文件内。 %implements "Type" "Language"
                 一般C语言作为 %implements "s-function" "C"
  %generatefile  提供记录Type和文件中包含的函数之间的映射，每条记录都可以同名，但映射到不同内容的函数。
                 一般是同名文件
  %filescope     将变量的范围限定为当前文件。文件中任意位置添加 %filescope 都可以将百年来限定到但钱文件内，在节省内存方面会有一些作用
  %include       在文件中插入指定的 tlc 目标文件。可以是绝对路径或者相对路径（和搜索路径有关）
  %addincludepath增加搜索路径。只对当前文件起作用 

  %openfile f = "name.txt", "w"  打开文件，参数为文件和模式。模式为： "r", "w", "a"
  %closefile f                   关闭文件
  %selectfile f                  选择输出流执行文件  文件可以是： NULL_FILE, STDOUT
  %openfile buffer           %% 数据被放在 buffer 缓冲区，不存放文件
  hello, world!
  %closefile buffer
  %% buffer = hello, world!

  %generate  生成

  %undef var   从某个范围内，删除一个变量。也可以删除记录内的变量，如果变量是数组，则数组内容全部删除。
  
%/


/%
  %roll %endroll
%/


/%
  %breakpoint    断点，在调试时需要。
%/


/%
  %function func_name(可选参数) [void | Output]
    %return 
  %endfunction
%/


/%
  \  作为行延续，必须是行的结尾，后续不可以有字符。
%/
```

## TLC 内置函数和值

> CAST(expr, expr)   第一个表达式必须是数据类型的字符串，第二个表达式会强制转为该类型。例如：   CAST("Real", var)


> EXISTS(var) 判断 var 是否在当前作用域内。 作用域包括 %include 范围

> FEVAL(matlab command, TLC-expressions)  在 Matlab 中执行表达式，并返回该结果。如果结果是多个值，则只接受第一个值。

> FILE_EXISTS(file_name)  file_name 必须是一个字符串。如果 file_name 不存在当前路径上，则返回 TLC_FALSE。路径包括 Matlab 设置路径，也包括 %addtoinclude 添加的路径。

> FORMAT(readvalue， format)  格式化readvalue。  

> FIELDNAMES(record)  返回包含记录字段名称的字符串数组

> GETFIELD(record, "record name")  如果记录名称和记录相关，则返回记录的内容。 时间复杂度为 O(1)

> GENERATE(record, fiunction-name, ... )  执行映射到特定记录类型的函数调用。例如，来执行内置块的 tlc 文件中的函数。

> GNERATE-FILENAME(type)  对于指定的记录类型，是否存在 .tlc 文件。

> GENERATE_FORMATTED_VALUE(expr, string, expand) 返回一个潜在的多行字符串。该字符串可以用在但钱目标语言中声明 expr 的值啊啊啊；第二个参数是一个字符串，用来描述返回字符串第一行的变量名称。如果为空，则不i会在输出字符串中放入任何描述性注释。第三个参数是一个布尔值；  TLC_TRUE，在输出之前展开为原始文本，但是内存会被占用很多。
> For example,  
> `static const unsigned char contents[] =`
> `%assign value = GENERATE_FORMATTED_VALUE(SFcnParamSettings.CONTENTS, "", TLC_FALSE)`
> `;`
> yields this C code:
> `static const unsigned char contents[] = { 0U, 1U, 2U, 3U, 4U };`

> GENERATE_FUNCTION_EXISTS(record, func_name) 确定给定的块函数是否存在。

> GENERATE_TYPE(record, func_name, type, ...)  

> GENERATE_TYPE_FUNCTION_EXISTS(record, function-name, type) 和 GENERATE_FUNCTION_EXISTS 不同的是，带有 type 类型。

> IDNUM(string)  第一个值为字符串之前的内容，第二个值是数字字符串转为数值

> IMAG(Real)   返回复数的虚部
> REAL(Real)    返回复数的实部

> ISALIAS(record) 如果是指向其他记录的引用（符号，浅拷贝）

> ISEQUAL(exp1, exp2) 如果两个都是数字，返回 TLC_TRUE。否则必须相同的数据类型，相同的内容

> ISEMPTY(expr)  空字符串，空向量，空记录。   0 不属于EMPTY。

> ISFIELD(record, "field-name") 如果字段名称和记录关联，则返回 TLC_TRUE。

> ISINF(expr)   ISNAN(expr)  IFFINITE(expr)

> ISSLPRMREF(param.value) 返回一个布尔值。该值反应参数是否为引用 Simulink 参数。此函数支持与 Simulink 共享参数，以便在生成代码过程中节省内存和时间。
> %if !ISSLPRMREF(param.Value) 
>     assign param.Value = CAST("Real", param.Value) 
>  %endif

> NULL_FILE 一个无效的输出流。  可以认为是黑洞
> STDOUT    标准输出，一般是显示设备

> NUMLTLCFILES 目前的作用域内，TLC文件的个数。【包括 %addtoinclude 和 %include 的文件】

> OUTPUT_LINES 返回已写入到当前文件或缓冲区的字符串

> REMOVEFIELD(record, "field-name")  删除记录内的字段

> SETFIELD(record, "field-name", value) 设置记录内的字段值。如果已经有字段，则返回 TLC_TRUE

> SIZE(expr)  计算表达式的大小

> SPRINTF(format, var, ... )  类似 sprintf() 

> STRING(expr)  将表达式转为字符串
> STRINGOF(expr)  将每个元素视为单个字符来构造


> SYSNAME(expr) 用于解析子系统名称。  `%< sysname("<sub>/Gain") >` 会被解析为 ['sub', 'Gain']

> TLCFILES 返回包含目标文件名称的向量
> TLC_TRUE  TLC_FALSE  
> TLC_TIME 编译时间
> TLC_VERSION  目标语言编译器的版本和日期

> TYPE(expr)  表达式的结果类型
> WHITE_SPACE(expr)  如果只包含空白字符(空格，TAB， 回车，换行)，则返回 TLC_TRUE
> WILL_ROLL(expr1, expr2)  第一个变量是滚动变量。 第二个表达式是范围。


### 命令行参数

命令行参数，可以辅助我们来调试 tlc 文件。

###  TLC 命令行选项

`tlc [switch expr1 switch2 expr2] file.tlc`

如果多次指定一个switch，则最后一个有效。

> -r file   
> 读取数据库( file.rtw ) 文件

> -v [number] 
> 内部详细基本设置为 number   默认为1

> -Ipath
> 添加搜索文件夹

> -Opath
> 指定生成的输出文件夹，包括 %openfile 打开的文件以及调试输出的文件。默认为 model_name_rtw

> -m[num]
> 指定最大错误数  没有-m 默认值为5   -m 默认为1

> -x0  
> 仅解析 TLC 文件，但是不执行

> -lint 
> 对性能和过时的功能执行一些简单的见擦汗

> -p[number] 
> 打印一个 点 (.) 表示执行的每个 TLC 原语操作的进度。

> -d[a | c | f | n | o ] 
> TLC 的调试模式
> -da 使能 TLC 的 %assert 命令
> -dc 调用 TLC 的命令行调试器
> -df filename  调试 TLC 命令行调试器，并运行 filename 指定的脚本
> -dn 生成 TLC 日志文件
> -do 禁用 TLC 调试行为


> -dr 
> 检查循环嵌套的记录

> -a[ident]#expr
> 为标识符 ident 赋值 expr   类似与  gcc 编译的 -D 功能

> -shadow[0 | 1]
> 覆盖局部变量

### 文件名和搜索路径

默认情况下，块级文件的名称与它们所在的块的 Type 相同。

搜索路径次序如下：
1. 当前文件夹
2. %addincludepath 指定的路径，从上到下依次搜索
3. 包含命令行中 -I 指定的路径。从右到左计算

注意： TLC 不会搜索 matlab 路径。


## 7. 调试 TLC 文件


## 8. 内联 S-Function

LEVEL1  的 C MEX S-Function 将导致生成的代码调用这些函数，即使例程为空。


> mdlInitializeSizes
> 初始化数组大小

> mdlInitializeSampleTimes 
> 初始化采样时间

> mdlInitializeConditions
> 初始化状态

> mdlOutputs
> 计算输出

> mdlUpdate
> 更新状态

> mdlDerivatives
> 计算连续状态的导数

> mdlTerminate
> 模型结束


未内联的 2 级 C MEX S-Function 调用上述函数。以下情况除外：
- 仅当定义了 MDL_INITIALIZE_CONDITIONS 时，才会调用 mdlInitializeConditions 函数
- 仅当 mdlStart 和 MDL_STTART 才会调用 mdlStart 。
- 仅当 mdlUpdate 和 MDL_UPDATE 一起，才会调用 mdlUpdate。
- 仅当 mdlDerivatives 和 MDL_DERIVATIVES 一起，才会调用 mdlDerivatives。

通过内联 S-Function ，可以消除仿真循环中对这些可能为空的函数的调用，可以大大提高生成代码的效率。

### S-Function 参数

S-Function 可以将两种不同参数写入到 model.rtw 中，以供 TLC 文件访问：
- 参数设置： 对于不可优化参数，可以通过 ssWriteRTWParamSettings 写入到 model.rtw 中，并在 TLC 文件中，通过 SFcnParamSettings 来访问
- 可调参数： 当此类参数在 S-Function 注册为运行参数时，可以访问此类函数。请注意，可调参数会自动写入到 model.rtw 文件内，在TLC文件中，可以使用 LibBlockParameter 函数访问运行时参数及其属性。

### S-Function 示例代码

设置一个模拟 Gain 模块，去油一个输入，一个输出，一个增益参数。MEX C文件示例如下：

```C
#define S_FUNCTION_NAME foogain 
#define S_FUNCTION_LEVEL 2 

#include "simstruc.h" 

#define GAIN mxGetPr(ssGetSFcnParam(S,0))[0] 

static void mdlInitializeSizes(SimStruct *S) 
{ 
  ssSetNumContStates (S, 0); 
  ssSetNumDiscStates (S, 0); 
  
  if (!ssSetNumInputPorts(S, 1)) return; 
  ssSetInputPortWidth (S, 0, 1); 
  ssSetInputPortDirectFeedThrough(S, 0, 1); 
  
  if (!ssSetNumOutputPorts(S, 1)) return; 
  ssSetOutputPortWidth (S, 0, 1); 
  ssSetNumSFcnParams (S, 1); 
  ssSetNumSampleTimes (S, 0); 
  ssSetNumIWork (S, 0); 
  ssSetNumRWork (S, 0); 
  ssSetNumPWork (S, 0); 
}

static void mdlOutputs(SimStruct *S, int_T tid) 
{ 
  real_T *y = ssGetOutputPortRealSignal(S, 0); 
  const InputRealPtrsType u = ssGetInputPortRealSignalPtrs(S, 0); 
  y[0] = (*u)[0] * GAIN; 
} 

static void mdlInitializeSampleTimes(SimStruct *S){} 

static void mdlTerminate(SimStruct *S) {} 

#define MDL_RTW /* Change to #undef to remove function */ 
#if defined(MDL_RTW)&&(defined(MATLAB_MEX_FILE)||defined(NRT)) 
static void mdlRTW (SimStruct *S) 
{ 
  if (!ssWriteRTWParameters(S, 1,SSWRITE_VALUE_VECT,"Gain","", mxGetPr(ssGetSFcnParam(S,0)),1)) 
  { return; } 
} 

#endif 

#ifdef MATLAB_MEX_FILE 
#include "simulink.c" 
#else 
#include "cg_sfun.h" 
#endif
```

model.c 的内联和非内联版本比较

如果没有 TLC 文件来定义 S-Function 细节，代码生成器必须通过 S-Function API 调用 MEX 文件。以下代码是非内联 S-Function 的 model.c 文件（不存在对应的 TLC 文件）

```C
/* 
 * model.c 
 * 
 */ 
real_T untitled_RGND = 0.0; 
 
/* real_T ground */ 
/* Start the model */ 
void MdlStart(void) 
{ 
   /* (no start code required) */ 
} 

/* Compute block outputs */ 
void MdlOutputs(int_T tid) 
{ 
  /* Level2 S-Function Block: /S-Function (foogain) */ 
  { 
    SimStruct *rts = ssGetSFunction(rtS, 0); 
    sfcnOutputs(rts, tid); 
  } 
} 

/* Perform model update */ 
void MdlUpdate(int_T tid) 
{ 
  /* (no update code required) */ 
} 

/* Terminate function */ 
void MdlTerminate(void) 
{ 
  /* Level2 S-Function Block: /S-Function (foogain) */ 
  { 
    SimStruct *rts = ssGetSFunction(rtS, 0); sfcnTerminate(rts); 
  } 
} 

#include "model_reg.h" 
/* [EOF] model.c */
```

内联 S-Function 文件


```C
/* 
 * model.c
 */  
/* Start the model */ 
void MdlStart(void) 
{ 
  /* (no start code required) */ 
} 

/* Compute block outputs */ 
void MdlOutputs(int_T tid) 
{
  /* S-Function block: /S-Function */ 
  /* NOTE: There are no calls to the S-function API in the inlined version of model.c. */
  rtB.S_Function = 0.0 * rtP.S_Function_Gain; 
} 

/* Perform model update */ 
void MdlUpdate(int_T tid) 
{ 
  /* (no update code required) */ 
} 

/* Terminate function */ 
void MdlTerminate(void) 
{ 
  /* (no terminate code required) */ 
} 
#include "model_reg.h" 
/* [EOF] model.c */
```

如果为此 S-Function 模块包含此目标文件，则生成的代码 model.c 为：

```C
rtB.S_Function = 0.0 * rtP.S_Function_Gain;
```

包含 TLC 文件不仅大大减少了代码大小，并提高了生成代码的效率。
- TLC 指令中 `%implements` 是块文件中必须的，并且必须是该文件的第一个可执行语句（前面可以有注释）。



### 管理块实例数据，着眼于代码生成

实例数据是 Simulink 模型中模块的每个实例所独有的额外数据和工作内存。这不包括参数或状态数据（分别存储在模型参数和状态向量中），二十用于缓存参数和模式的中间结果或派生表示。在 LEVEL 2 的 S-Function 中，可以通过多种方式逐个实例分配和使用内存：通过 ssSetUserData， 工作向量（ssSetRWorkValue，ssSetIWorkValue)。为了以最少的工作量编写 S-Function 和块目标文件，并自动符合目标（如grt）上的静态和动态分配实例数据，请在使用实例数据编写 S-Function 时使用数据类型工作变量。

优点是双重的，首先，编写 S-Function 更直接，因为内存分配和使用都是有 Simulink 处理。其次，DWork 向量会自动写入 model.rtw 文件，包括 DWork名称，数据类型和大小。这使得编写块目标文件更加容易。

此外，如果要将 DWork 向量组捆绑到结构体中传递给函数，可以在 S-Function mdlStart 函数和块目标文件的 Start 方法中使用执行 DWork数组的指针，填充结构，从而实现 S-Function 和生成代码对数据的处理之间的一致性。


### 目标文件方法

每一个块，都有一个同名的TLC文件，用于确定应该为哪个块来生成代码。代码可能有所不同，取决于块的参数和连接类型。

```C
BlockInstanceSetup(block, system)
BlockTypeSetup(block, system)
Enable(block, system)
Start(block, system)
InitializeConditions(block, system)
Outputs(block, system)
Update(block, system)
Derivatives(block, system)
Terminate(block, system)
```



在面对对象编程术语中，这些函数本质上是多态的。因为每个Block目标文件都可以包含相同的函数。TLC 在运行时，根据块的类型动态确定要执行的块函数。


> BlockInstanceSetup( block, system)

BlockInstanceSetup 对于模型的target文件定义了此函数的模块执行。例如，如果模型包含10个 From WorkSpace 模块，则 formwks.tlc 中的 BlockInstanceSetup 函数将执行10次，每个 From WorkSpace 模块实例都执行一次。


语法：
```tlc
BlockInstanceSetup(block, system) void 
  block = Reference to a Simulink block 
  system = Reference to a nonvirtual Simulink subsystem
```


``` tlc
%function BlockInstanceSetup(block, system) void 
  %if (block.InMask == "yes") 
    %assign blockName = LibParentMaskBlockName(block) 
  %else 
    %assign blockName = LibGetFormattedBlockPath(block) 
  %endif 
  %if (CodeFormat == "Embedded-C") 
    %if !(ParamSettings.ColZeroTechnique == "NormalInterp" && ... 
          ParamSettings.RowZeroTechnique == "NormalInterp") 
          %selectfile STDOUT
          open_system('%<blockName>') 
          at the MATLAB command prompt. 
          %selectfile NULL_FILE 
    %endif 
  %endif 
%endfunction
```


> BlockTypeSetup ( block, system )


在代码生成开始之前，BlockTypeSetup 为每个块类型执行一次，也就是说，如果模型中存在10个Lookup Table模块，则 loop_up.tlc 中的 BlockTypeSetup 函数仅调用一次。

语法：

```tlc
BlockInstanceSetup(block, system) void 
  block = Reference to a Simulink block 
  system = Reference to a nonvirtual Simulink subsyste
```


例如，给一个函数内，头文件需要有一个 `#define` 和 函数声明，则可以：
```tlc
%function BlockTypeSetup(block, system) void 
  %% Place a #define in the model's header file 
  %openfile buffer 
    #define A2D_CHANNEL 0 
  %closefile buffer 
  
  %LibCacheDefine(buffer) 
  %% Place function prototypes in the model's header file 
  %openfile buffer 
    void start_a2d(void); 
    void reset_a2d(void); 
  %closefile buffer 
  %LibCacheFunctionPrototype(buffer)
%endfunction
```


> Enable( block, system )
> Disable (block, system )


每当 Simulink 子系统包含具有 Enable 函数的模块时，代码生成器就会为废墟你子系统创建 Enable 函数。在模块的目标文件中包含 Enable 函数会将模块的特定使能代码至于此子系统 Enable 函数中。

```tlc
%% Function: Enable ============================================ 
%% Abstract: 
%% Subsystem Enable code is required only for the discrete form 
%% of the Sine Block. Setting the Boolean to TRUE causes the 
%% Output function to resync its last values of cos(wt) and 
%% sin(wt). 

%% %function Enable(block, system) Output 
  %if LibIsDiscrete(TID) 
    /* %<Type> Block: %<Name> */ 
    %<LibBlockIWork(SystemEnable, "", "", 0)> = (int_T) TRUE; 
  %endif 
%endfunction
```


> Start (block, system )

Start 函数只执行一次。

``` tlc
%% Function: Start ============================================ 
%% Abstract: 
%% Set the output to the constant parameter value if the block 
%% output is visible in the model's start function scope, i.e., 
%% it is in the global rtB structure. 

%% %function Start(block, system) Output 
  %if LibBlockOutputSignalIsInBlockIO(0) 
    /* %<Type> Block: %<Name> */ 
    %assign rollVars = ["Y", "P"] 
    %roll idx = RollRegions, lcv = RollThreshold, block, ... 
        "Roller", rollVars 
      %assign yr = LibBlockOutputSignal(0,"", lcv, ... "%<tRealPart>%<idx>") 
      %assign pr = LibBlockParameter(Value, "", lcv, ... "%<tRealPart>%<idx>") 
      %<yr> = %<pr>; 
      %if LibBlockOutputSignalIsComplex(0) 
        %assign yi = LibBlockOutputSignal(0, "", lcv, ... "%<tImagPart>%<idx>") 
        %assign pi = LibBlockParameter(Value, "", lcv, ... "%<tImagPart>%<idx>") 
        %<yi> = %<pi>; 
      %endif 
    %endroll 
  %endif 
%endfunction %% Start
```

> InitialsizeConditions ( block, system )

从块的 InitalsizeConditions 函数生成的 TLC 代码代码显示在以下两个位置之一。当非虚拟子系统配置为在启用时重置状态时，包含了一个 Initialize 函数。在这种情况下，此模块函数生成的 TLC 代码放置在子系统 Initialize 函数中，并且 start 函数调用此子系统 Initialize 函数。但是，如果 Simulink 模块位于跟系统或不需要 Initialize 函数的非虚拟子系统中，则此系统函数生成的代码将直接放入 start 函数中。

块函数 Start 和 InitializeConditions 中间存在细微差别。通常，您可以包含一个 Start 函数来执行代码，该代码在启用它所在的子系统时，不需要重新执行。您包含一个 InitializeConfitions 函数来执行代码，该代码必须在启用它所在的子系统时重新执行。例如：

``` tlc
%% Function: InitializeConditions ============================= 
%% 
%% Abstract: Invalidate the stored output and input in 
%% rwork[1 2*blockWidth] by setting the time stamp stored 
%% in rwork[0]) to rtInf. 

%% 
%function InitializeConditions(block, system) Output 
  /* %<Type> Block: %<Name> */ 
  %<LibBlockRWork(PrevT, "", "", 0)>= %<LibRealNonFinite(inf)>; 
%endfunction
```


> Outputs (block, system)

块通常需要包含 Outputs 函数，有区块的 Outpus 函数生成的 TLC 代码，位于两个位置之一。如果模块不位于虚拟子系统找那个，则代码直接防止在模型的 Outputs 函数。如果模块位于非虚拟子系统中，则代码直接位于子系统的Outputs函数中。例如：

```tlc
%% Function: Outputs ========================================== 
%% Abstract: 
%%     Y[i] = fabs(U[i]) if U[i] is real or 
%%     Y[i] = sqrt(U[i].re^2 + U[i].im^2) if U[i] is complex. 
%% 

%function Outputs(block, system) Output 
  /* %<Type> Block: %<Name> */ 
  %% 
  %assign inputIsComplex = LibBlockInputSignalIsComplex(0) 
  %assign RT_SQUARE = "RT_SQUARE" 
  %% 
  %assign rollVars = ["U", "Y"] 
  %if inputIsComplex 
    %roll sigIdx = RollRegions, lcv = RollThreshold, ... 
          block, "Roller", rollVars 
    %% 
      %assign ur = LibBlockInputSignal( 0, "", lcv, ... 
                   "%<tRealPart>%<sigIdx>")
      %assign ui = LibBlockInputSignal( 0, "", lcv, ... 
                   "%<tImagPart>%<sigIdx>") 
      %% 
      %assign y = LibBlockOutputSignal(0, "", lcv, sigIdx) 
      %<y> = sqrt( %<RT_SQUARE>( %<ur> ) + %<RT_SQUARE>( %<ui> ) ); 
    %endroll 
  %else 
    %roll sigIdx = RollRegions, lcv = RollThreshold, ... 
                   block, "Roller", rollVars 
      %assign u = LibBlockInputSignal (0, "", lcv, sigIdx) 
      %assign y = LibBlockOutputSignal(0, "", lcv, sigIdx) 
      %<y> = fabs(%<u>); 
    %endroll 
  %endif 
%endfunction
```
注意： 过零检查代码，也在 Outputs 函数中。

如果编写 TLC 代码以从 S-Function 生成内联代码，并且 TLC 代码包含 Outputs 函数。如果满足以下条件：
- Outputs port 使用或继承恒定残阳时间，outpus端口具有常量值。
- S-Function 是多速率 S-Function 或使用基于端口的采样时间。

在这种情况下，TLC 代码必须使用函数 output for TID ，而不是函数 Outputs 为常数值输出代码。


> Update ( block, system )

如果模块具有需要在每个主要时间步更新的代码，则包括 Update 函数。从此函数生成的代码将放置在模型的 Update 函数中。具体取决于是否位于非虚拟子系统中。例如：

``` tlc
%% Function: Update ============================================ 
%% Abstract: 
%% X[i] = U[i]
%% 

%function Update(block, system) Output 
  /* % Block: % */ 
  %assign rollVars = ["U", "Xd"] 
  %roll idx = RollRegions, lcv = RollThreshold, block, ... 
              "Roller", rollVars 
    %assign u = LibBlockInputSignal(0, "", lcv, idx) 
    %assign x = LibBlockDiscreteState("", lcv, idx) 
    %<x> = %<u>; 
  %endroll 
%endfunction %% Update
```


> Derivatives (block, system )

在生成代码以计算区块的连续状态时，包含一个 Derivatives 函数。  从此函数生成的代码将放置在 Derivatives 函数中，具体取决于模块是否位于非虚拟子系统中。


> Terminate ( block, system )

包括 Terminate 函数以将代码放置在 mdlTerminate 中。可以来保存数据、释放内存、重启硬件等操作。




## 9. TLC函数参考

### 描述

有几个 TLC 库函数提供对模块输入、输出、参数、采样时间和其他信息的访问，建议使用这些库函数来访问块记录中的许多参数名称/参数值，而不是直接使用 TLC 代码方位参数名称/参数值对。


库函数简化++了 TLC 代码，并为循环、数据类型、复杂数据提供函数支持，并且还提供了一个layer，防止直接操作model.rtw 内容。

使用这些函数的一个例外情况是，当访问块的参数时，可以使用 C MEX S-Function 的 mdlRTW 函数设置参数。该函数可以包含字符串、标量、向量、矩阵等数据。可以用作传递固定值和信息，用于更改块的代码，或者直接作为块的结果来生成代码。

### TLC 函数约定

可以在 matlabroot/toolbox/simulink 中找到这些函数的示例。

常见函数参数

| 参数          | 描述                                                                                                                                                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| portIdx       | 输入或输出端口索引，从0开始。                                                                                                                                                                                          |
| ucv           | 用户控制变量。这是一个高级功能，可以覆盖 lcv 和 sigIdx 函数。在内联函数中，通常指定为 ""                                                                                                                               |
| lcv           | Loop 控制变量。一般作为 %roll 指令的第二个承诺书使用。                                                                                                                                                                 |
| sigIdx or idx | 信号索引。有时也称为信号元素索引。什么时候直接访问 input 和 output 信号的特定元素，对各种 library 函数的调用应具有 ucv="", lcv="" 和 sigIdx 等于从0开始的所需证书信号索引。对于复杂信号，sigIdx 可以是重载的证书索引。 |
| paramIdx      | 参数索引。有时称为 parameter 元素索引。这个和上面的 sigIdx 很像。                                                                                                                                                      |
| stateIdx      | 状态索引。有时称为状态向量元素索引。计算结果必须为第一个元素从0开始的整数                                                                                                                                              |


大多数采样 sigIdx 参数的函数都已重载形式，其中 sigIdx 可以是：
- 一个整数，例如 3 。 如果引用的信号是 complex，则这是指 complex 容器的标识符。
- id-num ，通常采用以下格式。
	- `%<tRealPart>%<idx>` 信号的实部信息。
	- `%<tImagPart?%<idx>` 信号的虚部信息。


重载sigIdx
信号索引在传递给库函数时，大部分都会重载。

| sigIdx | Complex | 函数返回       | 示例     | 数据类型 |
| ------ | ------- | -------------- | -------- | -------- |
| "re3"  | Yes     | 元素的实部     | u0[2].re | real_T   |
| "im3"  | Yes     | 元素的虚部     | u0[2].im | real_T   |
| "3"    | Yes     | 元素的复杂容器 | u0[2]    | creal_T  |
| 3      | Yes     | 元素的复杂容器 | u0[2]    | creal_T  |
| "re3"  | No      | 元素           | u0[2]    | real_T   |
| "im3"  | No      | ""             | N/A      | N/A   |
| "3"    | No      | 元素           | u0[2]    | real_T  |
| 3      | No      | 元素           | u0[2]    | real_T  |

### 输入信号函数

> LibBlockInputPortIndexMode ( block, pidx )   确定块的 input 端口索引形式
> block 块记录    pidx 端口索引
> 返回：  “” 非索引端口 ，否则是 “Zero-based” 或 “One-based"
> 如果块的输入端口设置为索引端口，并且其索引基标记为 zero-based或one-based。则此信息写入model.rtw文件。LibBlockInputPortIndexMode 根据输入端口的所以哦呢，查询不同到的代码分支。
> 例如：  可以参阅 blocklib.tlc 中的 LibBlockInputPortIndexMode 函数。
```tlc
# See LibBlockInputPortIndexMode in blkiolib.tlc.
%if LibBlockInputPortIndexMode(block, pidx) == "Zero-based" 
   ... 
%elseif LibBlockInputPortIndexMode(block, pidx) == "One-based" 
   ... 
%else 
   ... 
%endif 
```


> LibBlockInputSignal( portIdx, ucv, lcv, sigIdx )
> 根据输入端口号（portIdx）、用户控制变量（ucv， user contorl var）、循环控制变量（lcv， loop control var）、信号索引（sigIdx）以及此输入信号的来源。LibBlockInputSignal返回对模块输入信号的引用。
> 返回的字符串值是表达式的有效右值。模块输入信号可以来自另一个模块、状态向量或外部输入、文字常量。
> 由于返回的值可以是文字常量，因此不应该使用 LibBlockInputSignal 来访问输入信号的地址。要访问输入信号的地址，请使用 LibBlockInputSignalAddr 函数。 通过 LibBlockInputSignal 访问信号的地址可能回导致对文本常量的引用。
> 例如，`%assign u = LibBlockInputSignal(0, "", lcv, sigIdx)    x = &%<u>;` 语句不起作用。
> 如果 u 代表的是常数(5)，则生成的代码为 `x = &5;` 是一个无效语句。
> 可以使用 `%assign uAddr = LibBlockInputSignalAddr(0, "", lcv, sigIdx)   x = %<uAddr>;` ，
> 直接索引：  如果 ucv=“” 且 lcv=“” 则 LibBlockInputSignal 返回 sigIdx 指定多元素的索引表达式。
> 循环滚动/展开  在这种情况下， lcv和 sigIdx 由 %roll 指令生成，并且 ucv 必须为 “”。 仅当由 %roll 指令生成并且使用 Roller TLC 文件（或用户提供的符合相同变量/信号偏移处理的 Roller TLC 文件），才允许 lcv 的非空值。此外，只有当 “U” 或特定输入端口（例如， “u0”）通过 roll variables 参数传递给 %roll 指令时，才应该使用 lcv 调用 LibBlockInputSignal。
``` tlc
%assign rollVars = ["U", "Y", "P"] 
%roll sigIdx=RollRegions, lcv=RollThreshold, block, ... 
      "Roller", rollVars 
  %assign u = LibBlockInputSignal( 0, "", lcv, sigIdx) 
  %assign y = LibBlockOutputSignal(0, "", lcv, sigIdx) 
  %assign p = LibBlockParameter( 0, "", lcv, sigIdx) 
  %<y> = %<p> * %<u>; 
%endroll
```

使用 %roll 指令，sigIdx 时当前滚动区域的起始索引，lcv 时 “” 或索引变量。以下是有效值的示例；
``` tlc
LibBlockInputSignal(0, "", lcv, sigIdx)    rtB.blockname[0]
LibBlockInputSignal(0, "", lcv, sigIdx)    u[i]
```
第一个例子中，当 input port 连接到另一个模块的 output，并且：
- %roll 指令生成的控制变量 lcv 为空，表示当前滚动区域低于滚动阈值，sigIdx为0.
- input port的宽度为1，表示该端口正在标量扩展。如果 sigIdx 不为0，则返回 rtB.blockname[0]。例如 sigIdx 为3 则返回 rtB.blockname[3]。

第二个事例中，如果当前滚动区域高于滚动阈值，且输入端口宽度为非标量时，LibBlockInputSignal 返回 u[0]。 在这种情况下，Roller TLC 文件设置一个局部变量 u 来指向输入信号，并且当前 %roll 指令中的代码被放置在 for 循环中。



> LibBlockInputSignalAddr (portIdx, ucv, lcv, sigIdx )
> 返回一个字符串，提供指定块输入信号的内存地址。
> 当需要输入信号地址时，必须使用 LibBlockInputSignalAddr，而不是在 LibBlockInputSignal 返回的的字符串后附件 '&'。例如：LibBlockInputSignal 可以返回一个 Literal 常量，比如 5（不可修改的常量）。代码生成器跟踪何时在不变信号上调用 LibBlockInputSignalAddr，并将该信号声明为 const数据，而不是作为文本常量防止在生成的代码中。
> 请注意，最后一个输入参数 sigIdx 没有重载，它在 LibBlockInputSignal 中。因此，如果 input 信号是复数，则返回复数容器的地址。
> 要返回宽输入信号的地址并将地址传递给用户函数进行处理，可以使用
> ``` tlc
> %assign uAddr = LibBlockInputSignalAddr(0, "", "", 0) 
> %assign y = LibBlockOutputSignal(0, "", "", 0) 
> % < y > = myfcn(%< uAddr >);
> ```
> 详细用例，可以参考 blocklib.tlc 中的 LibBlockInputSignalAddr 函数。


> LibBlockInputSignalAliasedThruDataTypeName( portIdx, reim )
> 返回别名 thru 数据类型的名称（例如，int_t, ... creal_T），对应于指定的模块输入端口。如果需要完成的信号类型名称，则 reim 参数指定为 "" 。
> 例如： reim 参数为 "" ，并且第一个输出端口是 real 和 complex，则防止在 dtname 中的数据类型名称为 creal_T。
> `%assign dtname = LibBlockInputSignalDataTypeName(0, "")`
> `dtname: creal_T`
> 如果需要原始元素类型名称，请将 reim 指定为 tRealPart。
> `%assign dtname = LibBlockInputSignalDataTypeName(0, tRealPart)`
> `dtname: real_T`
> 详细用例，可以参考 blocklib.tlc 中的 LibBlockInputSignalDataTypeName 函数。


> LibBlockInputSignalConnected(portIdx)
> 如果指定的输入端口连接到 GND 模块以外的模块，则返回1， 否则返回0
> 详细用例，可以参考 blocklib.tlc 中的 LibBlockInputSignalConnected 函数。

> LibBlockInputSignalDataTypeId(portId)
> 返回与指定模块输入端口的数据类型对应的数组标识符（id）
> 如果输入端口信号为复数，则 LibBlockInputSignalDataTypeId 返回信号的实部的数据类型
> 详细用例，可以参考 blocklib.tlc 中的 LibBlockInputSignalDataTypeId 函数。

> LIbBlockInputSignalDataTypeName(portIdx, reim)
> 返回指定的模块输入端口的数据类型的名称（例如，int_T，... creal_T）。
> 如果需要完成的信号类型名称，请将 reim 的参数指定为 "" 。
> %assign dtname = LIbBlockInputSignalDataTypeName(0, "")
> 详细用例，可以参考 blocklib.tlc 中的 LIbBlockInputSignalDataTypeName 函数。


> LibBlockInputSignalDimensions(portIdx) 返回制定模块输入端口的维度向量，例如 [1, 3]
> LibBlockInputSignalIsComplex(portIdx)  输入端口是否为复数
> LibBlockInputSignalIsFrameData(portIdx)  输入端口是否基于 frame
> LibBlockInputSignalLocalSampleTimeIndex(portIdx) 返回与指定模块输入端口对应的本地采样时间索引
> LibBlockInputSignalNumDimensions(portIdx) 返回指定模块输入端口的维度数字
> LibBlockInputSignalOffsetTime(portIdx) 返回指定模块输入端口对应的偏移时间
> LibBlockInputSignalSampleTime(portIdx) 返回指定模块输入端口对应的采样时间
> LibBlockInputSignalSampleTimeIndex(portIdx) 返回指定模块输入端口对应的采样时间索引
> LibBlockInputSignalSymbolicDimensions(portIdx) 返回指定模块输入端口的维度数
> LibBlockInputSignalSymbolicWidth(portIdx) 返回制定模块输入端口的符号宽度
> LibBlockNumInputPorts(block) 返回模块的数据输入端口数（不包含控制端口）


### 输出信号功能

可以参考 `blkiolib.tlc` 文件。

> LibBlockAssignOutputSignal(portIdx, ucv, lcv, sigIdx, rhs) 根据输出端口号（portIdx）、用户控制滨安路（ucv）、循环控制变量（lcv）、信号索引（sigIdx）、输出信号目标，LibBlockAssignOutputSignal 将模块是的输出分配给指定的右侧值
> 可以参考 customstoragelib.tlc 中的 LibBlockAssignOutputSignal
> LibBlockNumOutputPorts(block)  返回模块的数据输出端口数
> LibBlockOutputPortIndexMode(block, pidx) 确定块输出端口的索引模式   分为 "Zero-based"" 或 “Zero-based"
> 可以参考 blkiolib.tlc 中的 LIbBlockOutputPortIndexMode 函数
> LibBlockOutputSignal(portIdx, ucv, lcv, sigIdx) 返回对模块输出信号的引用
> LibBlockOutputSignalAddr(portIdx, ucv, lcv, sigIdx) 返回一个字符串，指定模块输出端口的内存地址。
> LibBlockOutputSignalAliasedThruDataTypeName(portIdx, reim) 返回类型名称字符串的别名数据类型
> LibBlockOutputSignalBeingMerged(portIdx) 返回指定的输出端口是否链接到 Merge 块。
> LibBlockOutputSignalConnected(portIdx) 指定的输出端口连接到 GND 以外的块。
> LibBlockOutputSignalDataTypeId(portIdx) 指定数据块输出端口的数据类型对应的数字ID
> LibBlockOutputSignalDataTypeData(portIdx, reim) 返回类型名称字符串的数据类型 
> LibBlockOutputSignalDimensions(portIdx)  返回指定模块输出端口的维度
> LibBlockOutputSignalIsComplex(portIdx)  输出端口是否为 complex 类型
> LibBlockOutputSignalIsExpr(portIdx) 输出信号是否是表达式
> LibBlockOutputSignalIsFrameData(portIdx) 宽输出端口是基于 frame 的
> LibBlockOutputSignalLocalSampleTimeIndex(portIdx) 返回与指定模块输出端口对应的本地采样时间索引
> LibBlockOutputSignalNumDImensions(portIdx) 返回指定的输出端口的维度
> LibBlockOutputSignalOffsetTime(portIdx)  返回指定模块输出端口对应的偏移时间
> LibBlockOutputSignalSampleTime(portIdx)  返回与指定模块输出端口对应的采样时间
> LibBlockOutputSignalSymbolicDimensions(portIdx) 返回指定块输出端口的符号维度
> LibBlockOutputSignalSymbolicWidth(portIdx)  返回指定块输出端口的符号宽度
> LibBlockOutputSignalSampleTimeIndex(portIdx)  返回与指定模块输出端口对应的采样时间素银
> LibBlockOutputSignalWidth(portIdx) 返回指定宽输出端口的宽度


### 参数函数
可以参考 paramlib.tlc 文件，查看实现过程。
> LibBlockMatrixPrarmeter(param, rucv, rlcv, ridx, cucv, clcv, cidx) 给定行列用户控制变量（rucv, cucv）、循环控制变量（rlcv, ulcv）和索引（ridx，uidx），返回块的 matrix 参数。通常，块应使用 LibBlockParameter 。
> LibBlockMatrixParameterAddr(param,  rucv, rlcv, ridx, cucv, clcv, cidx)  返回矩阵参数的地址
> LibBlockMatrixParameterBaseAddr(param) 返回矩阵参数的基地址
> LibBlockParamSetting(bType, psType) 返回指定参数的字符串。如果传入空的block类型参数，则参数设定在该block的ParamSettings的记录中。
> LibBlockParameter(param, ucv, lcv, sigIdx) 根据param， ucv， lcv， sigIdx和参数内联状态，返回对块参数的引用。
> LibBlockParameterAddr(param, ucv, lcv, idx) 返回块参数的地址。  当全局 InlineParams 变量等于1时，使用 LibBlockParameterAddr 访问参数将导致该变量在 RAM 被声明为 const 而不是内联。
> LibBlockParameterBaseAddr(param)  返回块参数的基地址。
> LibBlockParameterDataTypeId(param) 返回知道你更快参数的数据类型对应的数字ID
> LibBlockParameterDataTypeName(param, reim)  返回指定块参数对应的数据类型的名称。
> LibBlockParameterDimensions(param) 返回一个长度为N的行向量，给出参数数据的维度
> ``` tlc
> %assign dims = LibBlockParameterDimensions("paramName")
> %assign nDims = SIZE(dims, 1)
> %foreach i = nDims
>     %< i+1> = %< dims[i]>
>  %endforeach
> ```
> LibBlockParameterIsComplex(param) 返回指定的块参数为 complex。
> LibBlockParameterSize(param)  返回大小为2的向量，格式为【nRows, nCols】。
> LibBlockParameterString(param) 根据快参数引用，返回解释为字符串的指定块参数
> LibBlockParameterValue(param, elIdx) 根据块参数引用（param）和数组的索引元素（elIdx）中。
> ``` tlc
> %assign mode = LibBlockParameterValue(Integrator, 0) 
> %switch (mode) 
>    %case 1 
>     %< CodeForIntegrator1 >
>     %break 
>    %case 2 
>     %< CodeForIntegrator2 >
>     %break 
>    %default 
>     Error: Unrecognized integrator value. 
>     %break 
>  %endswitch
> ```
> LibBlockParameterWidth(param)  返回参数的元素数目。



### 块状态和工作向量函数
可以参考 `blocklib.tlc`。
> LibBlockAssginDWork(dwork, ucv, lcv, sigIdx, rhs)  根据块的 dwork 索引或记录、ucv、lcv、sigIdx，LibBlockAssginDWork将块的 dwork 分配给 rhs。
> LibBlockContinuousState(ucv, lcv, idx)  返回与指定的块连续状态元素对应的字符串。
> LibBlockContimuousStateDerivative(ucv, lcv, idx)  返回与指定的块连续状态元素对应的字符串
> LibBlockContStateDisabled(ucv, lcv, idx)  返回与指定连续状态对应的字符串
> LibBlockDWork(dwork, ucv, lcv, idx)  返回与指定的块 dwork 元素响应的字符串。最有一个参数被重载以处理复杂的dworks
> LibBlockDWorkDataTypeId(dwork) 返回制定快 dwork 的数据类型ID
> LibBlockDWorkDataTypeName(dwork, reim) 返回指定快 dwork的数据类型名称
> LibBlockDWorkIsComplex(dwork) 返回指定块 dwork 是否是复数
> LibBlockDWorkName(dwork) 返回指定 dwork 的名称
> LibBlockDWorkStorageClass(dwork) 返回指定块 dwork 的类
> LibBlockDWorkStorageTypeQualifier(dword) 返回指定块 dowrk 的存储类型限定符
> LibBlockDWorkUsedAsDiscreteState(dwork) 返回dwork是否是离散状态
> LibBlockDWorkWidth(dwork) 返回指定块 dowrk 的宽度
> LibBlockDiscreateState(ucv, lcv, idx) 返回与指定块离散状态元素对应的字符串
> LibBlockIWork(definediwork, ucv, lcv, idx) 返回与指定块 IWORK 对应的I字符串
> LibBlockMode(ucv, lcv, idx) 返回与指定块 MODE 元素对应的字符串
> LibBlockNonSampledZC(ucv, lcv, NSZCIdx)  返回与指定块 NSZC 对应的字符串。
> LibBlockPWork(definedPWork, ucv, lcv, idx) 返回与指定块 PWORK 元素对应的字符串
> LibBlockRWork(definedRWork, ucv, lcv, idx) 返回与制定快 RWORK 元素对应的字符串。第一个符号，是在 MEX C文件中定义的符号： `ssWriteRTWWorkVect([...], 'RWork", [...], "MyRWorkName", [...])`
> LibBlockZCSignalValue(ucv, lcv, zcsIdx, zcElIdx) 返回与指定区块 ZCSignalValue 对应的字符串


### 阻止路径和错误报告功能

可以在 utillib.tlc 中查看函数。

> LibBlockReportError(block, errorString) 可以在没有块的记录范围内调用，则block需要传递 [] 空。例如：`LibBlockReportError([], "error");   LibBlockReportError(blockrecord, "error")`
> LibBlockReportFatalError(block errorString) 在报告块的致命错误时，可以使用 LibBlockReportFatalError 进行防御性编程。
> LibBlockReportWarning(block warnString) 参考 LibBlockReportError
> LibGetBlockName(block) 获取块记录的路径名字，不包括回车和其他特殊字符串
> LibGetBlockPath(block) 返回块的完成路径，包括回车和特殊字符。
> LibGetFormattedBlockPath(block) 返回不带特殊字符的完成路径名称字符串。


### 代码配置函数

可以在 codetempatelib.tlc 中查看函数

> LibAddSourceFileCustomSection(file, buildInSection, newSection)  将自定义部分添加到源文件。 但是 buildInSection 必须与内置部分相关联： Includes、Defines、Types、Enums、Definitions、Declarations、Functions、Documentation。如果该部分不存在，则创建该区域；如果存在，则不会发生任何事情。
> LibAddToCommonIncludes(incFileName) 将项目添加到 `#include/package` 规范项目列表中。列表中的每个成员都是唯一的。尝试添加重复成员不会有任何操作。应该从TLC方法调用 LibAddToCommonIncludes 以指定在 moodel.h 中生成 `#include` 语句。在尖括号内的包含路径上指定文件名。例如： `LibAddToCommonIncludes("tpu332lib.h")`
> LibAddToModelSources(newFile)  将文件添加到源文件列表中。
> LibCacheDefine(buffer)  每次调用 LibCacheDefine(buffer) 都会将缓冲区追加到现有缓冲缓冲区。例如：
> ```
> %openfile buffer
>   # define NTERP(X, Y)  (X+Y)
>   # define THIS THAT
> %closefile buffer
> %< LibCacheDefine(buffer) >
> ```
> LibCacheExtern(buffer) 参考 LibCacheDefine
> LibCacheFunctionPrototype(buffer) 参考 LibCacheDefine
> LibCacheTypedefs(buffer) 参考 LibCacheDefine
> LibCallModelInitalize() 返回用于调试模型的初始化代码
> LibCallModelStep(tid) 返回用于调用模块的步长函数的代码
> LibCallModelTerminate() 返回终止函数的代码
> LibCallSetEventForThisBaseStep(bufferName) 返回设置事件函数的代码
> LibClearFileSectionContents(fileIdx, attrib) 将文件写入磁盘之前，清除自定义文件部分。 fileIdx 文件索引， attrib 模型属性名称
> LibCreateSourceFile(type, creater, name)  创建新的 C 或 C++ 文件，并返回引用。如果文件存在，则返回当前文件的引用。
> ```
>    %assign file = LibCreateSourceFile("Source", "Custom", "foofile")
> ```
> LibGetFileRecordName(file)  返回不带文件爱你扩展名的模型文件名（包括路径）。
> LibGetMdlPrvHdrBaseName() 返回模型的私有头文件的名称，例如 model_private.h
> LibGetMdlPubHdrBaseName() 返回模型公共头文件的名称，例如 model.h
> LibGetMdlSrcBaseName() 返回模型主源文件的名称，例如 model.c
> LibGetMdlDataSrcBaseName() 返回模型数据文件的名称，例如 model_data.c
> LibGetMdlTypesHdrBaseName() 返回模型类型文件的名称，例如 model_types.h
> LibGetMdlCapiHdrBaseName()/LibGetMdlCapiSrcBaseName()/LibGetMdlCapiHostHdrBaseName() 返回模型 capi 够文件的名称，例如 model_capi.h/model_capi.c/model_host_capi.h
> LibGetMdlTestIfHdrBaseName()/LibGetMdlTestIfSrcBaseName  返回模型 testinterface 文件的名称，例如 model_testinterface.h model_testinterface.c
> LibGetDataTypeTransHdrBaseName() 返回数据类型转换文件的基本名称
> LibGetModelDotCFile()/LibGetModelDotHFile() 返回对 model.c或 model.cpp 文件的引用。然后可以使用 LibSetSourceFileSection 缓存其他代码。例如： `%assign srcFile = LibGetModelDotCFile() %<LibSetSourceFileSection(srcFile, "Functions", mybuf>` 返回对文件的引用。
> LibGetModelName() 返回模型的名称（不带扩展名称）
> LibGetNumSourceFiles() 返回已经创建的源文件(.c/.cpp/.h) 的数量
> LibGetRTModelErrorStatus() 获取模型错误状态  `%<LibGetRTModelErrorStatus()>`
> LibGetSourceFileAttribute(fileIdx, attrib)  返回文件的指定属性
> attrib:  Name, SystemsInFile,  IsEmpty, SharedType, BaseName, RequiredIncludes, Indent, CodeTemplate, Type, utilityIncludes, WrittenIncludes, WrittenToDisk, OutputDirectory, Creator, Filter, Shared, Group
> LibGetSourceFIleFromIdx(fileIdx) 根据模型文件的索引，返回模型文件引用。次饮用对于所有文件的常见操作非常有用。例如设置所有文件的前导文件标题。 
> ```
> %assign fileH = LibGetSourceFileFromIdx(fileIdx)
> ```
> LibGetSourceFileSection(fileIdx, section) 返回文件的内容
> LibGetSourceFileTag(fileIdx) 分别返回头文件和源文件的 fileName_h和 fileName_c 文件。其中 fileName 是模型的名称
> LibMdlRegCustomCode(buffer, location) 将生成语句和可执行代码放在 model_initialize 函数中
>     location:  “header" 函数顶部   "declaration": 函数顶部  "execution" 函数顶部，header之后  "trailer" 函数底部
>  LibMdlStartCustomCode(buffer, location)  声明语句和可执行语句放在 start 函数中，启动代码只执行一次。
>  LibMdlTerminateCustomCode(buffer, location)  将声明语句和可执行语句放在 terminate 函数内
>  LibSetRTModelErrorStatus(str) 返回设置模块错误状态所需的代码 。 `LibSetRTModelErrorStatus("\"Overrun"\")`
>  LibSetSourceFileCodeTemplate(opFile, name) 可以更改文件的模板。 `%assgin tag = LibSetSourceFileCodeTemplate(opFile, name)` name: 所需模板的名称，模板路径需要添加到matlab路径内。
>  LibSetSourceFileCustomSection(file, attrib, value) 添加到 LibAddSourceFileCustomSection 创建的自定义部分的内容。
>  ``` tlc
>  %openfile buffer
>    int a = 0;
>    for (a = 0; a < 10; a+)
>    {
>    }
>  LibSetSourceFileCustomSection(file, "Custom", buffer)
>  %closefile buffer
>  ```
>  LibSetSourceFileOoutputDirectory(opFile, name) 允许更改生成指定源文件的文件夹，文件夹需要存在。 dirName : 文件夹路径
>  LibSetSourceFileSection(fileH, section, value) 添加到指定文件的指定节的内容。 section： Banner, Includes, Defines, IntrinsicTypes, PrimitiveTypedefs, UserTop, Enums, Definitions, ExternData, ExternFcns, FcnPrototypes, Functions, CompilerErrors, CompilerWarnings, Documentation, UserBottom.。代码生成器按照上述名称排序。
>  ``` tlc
>  %% 遍历文件
>  %openfile tmpBuff
>  %% 内容无所谓
>  %closefile tmpBuff
>  %foreach fileIdx = LibGetNumSourceFIles() 
>    %assign fileH = LibGetSourceFileFromIdx(fileIdx)
>    %<LibSetSourceFileSection(fileH, "SectionOfInterest", tmpBuff) 
>  %endforeach
>  
>  %assign fileH = LibCreateSourceFile("Header", "Custom", "foofile")
>  %<LibSetSourceFileSection(fileH, "Defines", "#deine FOO 5 \n")>
>  ```
>  LibSystemDerivativeCustomCode(system, buffer, location) 声明语句和可执行代码放在子系统的派生函数中
>  LibSystemDisableCustomCode(system, buffer, location)/LibSystemDisableCustomCode(system, buffer, location)/LibSystemInitalizeCustomCode(system, buffer, location)/LibSystemOutputCustomCode(system, buffer, location)/LibSystemUpdataCustomCode(system, buffer, location)  声明语句和可执行代码放在子系统的禁用/使能/初始化/输出/更新函数中
>  LibWriteModelData() 返回模型的数据
>  LibWriteModelInput(tid, rollThreshold)/LibWriteModelOutpus(tid, rollThreshold)  返回用于写入指定根写入（模型输入端口块）的代码。对引用模型无效。
>  LibWriteModelInputs()/LibWriteModelOutputs() 返回写入所有输入/输出的代码。



### 采样时间函数

可以在 utillib.tlc 中查看函数实现。

TID  taskId

> libAsynchronousTriggeredTID(tid)  返回 TID 是否对应异步触发速率
> LibAsyncTaskAccessTimeInFcn(tid, fcnType) 如果指定的异步任务标识符（TID）为函数类型执行，则返回1
> LibBlockSampleTime(block) 返回块的采样时间。> 0： 离散； =0.0： 连续； =-1.0： 触发； =-2.0： 常数
> LibGetClockTick(tid) 返回时间，数据类型为整数。
> LibGetClockTickDataTypeId(tid) 返回 clock tick 数据类型ID。
> LibGetClockTickHigh(tid) 返回任务时间的高字。
> LibGetClockTickStepSize(tid) 返回整数任务时间的分辨率
> LibGetElapseTime(system) 返回上致辞子系统开始执行以来经过的时间
> LibGetElapseTimeCounter(system) 返回一个整数运行时间。 字上次系统启动依赖经过的 clock ticks 数字。
> LibGetElapseTimeCounterDTypeId(system) 返回 LibGetElapseTimeCounter 的证书时间的日期类型ID
> LibGetElapseTimeResolution(system) 返回 LibGetElapseTimeCounter 的运行时间分辨率
> LibGetGlobalTIDFromLocalSFcnTID(sfcnTID) 返回与指定的本地 S-Funtion 任务标识符或端口采样时间对应的模型任务标识符。 
> ``` tlc
> %assign globalTID = libGetGlobalTIDFromLocalSFcnTID(2)
> %% 或
> %% %assign globalTID = LibGetGlobalTIDFromLocalSFcnTID("InputPortIDx4")
> %assign period = CompiledModel.SampleTime[GlobalTID].PeriodAndOffset[0]
> %assign offset = CompiledModel.SampleTime[GlobalTID].PeriodAndOffset[1]
> ```
> 继承的采样时间
> ``` tlc
> %switch (LibGetSFcnTIDType(0)) 
>   %case "discrete" 
>   %case "continuous" 
>     %assign globalTID = LibGetGlobalTIDFromLocalSFcnTID(2) 
>     %assign period = CompiledModel.SampleTime[globalTID].PeriodAndOffset[0] 
>     %assign offset = ... CompiledModel.SampleTime[globalTID].PeriodAndOffset[1] 
>     %break 
>     
>   %case "triggered" 
>     %assign period = -1 
>     %assign offset = -1 
>     %break 
>   %case "constant" 
>     %assign period = rtInf 
>     %assign offset = 0  
>     %break 
>    %default 
>    %<LibBlockReportFatalError([], "Unknown tid type">)
> %endswitch
> ```
> LibGetNumAsyncTasks()  返回生成的代码中异步任务的数量
> LibGetNumSFcnSampleTimes(block) 返回块的 S-Function 采样时间数
> LibGetNumSyncPeriodicTasks() 返回生成代码中周期性任务数
> LibGetNumTasks()  返回生产的代码中任务数
> LibGetSampoleTimePeriodAndOffset(tid, idx) 返回指定任务的采样时间段值或偏移值
> ``` tlc
> %% Get sample time period and offset for task 0 
> %assign sampleTime = LibGetSampleTimePeriodAndOffset(0,0) 
> %assign offsetTime = LibGetSampleTimePeriodAndOffset(0,1) 
> %% Get sample time periods for tasks 0 and 1 
> %assign periodTID0 = LibGetSampleTimePeriodAndOffset(0,0) 
> %assign periodTID1 = LibGetSampleTimePeriodAndOffset(1,0)
> ```
> LibGetSFcnTIDType(sfcnTid) 返回指定的 S-Function 任务标识符的类型。"continuous" 连续   "discrete" 离散   "triggered"  触发   "contstant"  常数
> LibGetTaskTime(tid) 返回一个字符串以访问任务的绝对时间。
> LibGetTaskTimeFromTID(block) 返回一个字符串，用于访问与数据块关联的任务的绝对时间。
> LibGetTID01eq()  返回TID01EQ标志的值。如果连续任务与第一个离散任务采样率相等，则返回1，否则返回0
> LibIsContinuous(TID) 如果指定的任务标识符 (TID) 是连续的，则返回1，否则返回0.
> LibIsDiscrete(TID) 如果指定的任务标识符(TID) 会离散的，则返回1
> LibIsSFcnSampleHig(sfcnTID) 如果指定的本地 S-Function TID 发生样本点击，返回1
> LibIsSFcnSingleRate(block) 只是 S-Function是单速率（1）还是多速率（0）
> LibIsSFcnSpecialSampleHig(sfcnSTI, sfcnTID)  将慢速任务升级到快速任务
> LibIsSingleRateModel() 单速率，则返回1
> LibIsSingleTasking() 模型为单任务执行，则返回1
> LibIsZOHContinuous(TID)  如果指定的TID为0，则返回1
> LibNumAsynchronousSampleTImes() 返回模型中的异步采样时间数
> LibNumDiscreteSampleTimes() 返回模型中离散采样时间的数目
> LibNumSynchronousSampleTimes() 返回模型中的同步采样时间数目
> LibPortBasedSampleTimeBlockIsTriggered(block) 确定是否触发基于端口的 S-Function 块
> LibSetVarNextHitTime(block, tNext) 生成代码以设置下一个变量命中时间。
> LibTriggeredTID(tid) 返回TID是否对应触发的速率


### 其他功能

> LibBlockExecuteFcnCall(block, callIdx)  具有函数调用的 S-Function 使用。
> LibBlockExexuteFcnDisable(block, callIdx) 调用 func-call 系统的 diasble 函数。
> LibBlockExecuteFcnEnable(block, callIdx)
> LibBlockInputSignalAliasedThruDataTypeId(idx) 返回输入信号别名的数据类型ID
> LibBlockOutputSignalAliasedThruDataTypeId(idx) 返回输出信号别名的数据类型ID
> LibGenConstVectWithInit(data, typeId, varId) 返回初始化静态常量变量字符串
> LibGetBlockAttribute(block, attr) 获取块记录中的字段值
> LibGetCallerClockTickCounter(sfcnBlock) 异步任务可以管理自己的时间
> LibGetCallerClockTickCounterHigh(sfcnBlock) 异步任务可以管理自己的时间
> LibGetDataTypeComplexNameFromId(id) 返回与数据类型ID对应的复杂数据类型的名称
> LibGetDataTypeEnumFromId(id) 返回与数据类型ID对应的数据类型枚举
> LibGetDataTypeIdAliasedThruToFromId(id) 返回与数据类型对应的别名
> LibGetDataTypeIdAliasedToFromId(id) 返回与数据类型ID对应的别名
> LibGetDataTypeNameFromId(id) 返回与数据类型ID对应数据类型名称
> LibGetDataTypeSLSizeFromId(id) 翻译与数据类型ID对应的大小
> LibGetDataTypeStorageIdFromId(id) 返回与数据类型 ID 对应的数据类型 StorageId
> LibGetFcnCallBlock(sfcnblock, callIdx) 返回下游的 function-call subsystem 模块的模块记录
> LIbGetRecordDatTypeId(record) 返回指定记录的数据类型标志符
> LibGetRecordDimensions(record) 返回指定记录的维度
> LIbGetRecordIsComplex(record) 返回指定记录是否是 complex类型
> LibGetRecordWidth(record) 返回记录的宽度
> LibGetT() 访问绝对时间
> LibIsComplex(arg) 查看参数是否是 complex
> LibIsFirstInitCond() 返回生成的代码。
> LibIsMajorTimeStep() 返回一个字符串，以访问当前模拟步骤是否为主要时间步长
> LibIsMinorTimeStep()
> LibManageAsyncCounter(sfcnBlock, callIdx) 异步任务可以管理自己的时间
> LibMaxIntValue(dtype) 返回数据类型的最大值
> LIbMinIntValue(dtype)
> LibNeedAsyncCounter(sfcnBlock, callIdx) 是否用异步计数器
> LibSetAsuncClockTicks(sfcnBlock, callIdx, buf1, buf2) 
> LibSetAsyncCounter(sfcnBlock, callIdx, buf)
> LibSetAsyncCounterHigh(sfcnBlock, callIdx, buf)
> LibTIDInSystem(system, fcnType) TID是否位于子系统函数的范围内。
> LibIsRowMajor 当前模块是否优先行数组布局


### 高级功能

可以在 modelrefutil.tlc 中查看函数的实现。

> LibAppendToModelReferenceUserData(data) 将给定的数据对象附加到当前正在构建的模型的binfo问价脑袋用户数据。
> LibBlockInputSignalBufferDstPort(portIdx) 返回共享相同内存的输入端口对应的输出端口，否则返回 -1
> LibBlockInputSignalStorageClass(portIdx, sigIdx) 返回指定 block input port 信号的存储类。返回： "Auto"、"ExportedSignal"、"ImportedExtern"、"ImportedExternPointer"。
> LibBlockInputSignalStorageTypeQualifier(portIdx, sigIdx) 
> LibBlockOutputSignalIsGlobal(portIdx) 如果在全局范围内生命了指定的模块信号，则返回1
> LibBlockOutputSignalIsInBlockIO(portIdx) 如果指定的块输出端口存在与全局块 I/O 数据结构中，则返回1
> LibBlockOutputSignalIsValidLValue(portIdx)
> LibBlockOutputSignalStorageClass(portIdx)
> LibBlockOutputSignalStorageTypeQualifier(portIdx)
> LibBlockSrcSignalBlock(portIdx, sigIdx) 返回对指定 block input 端口元素的block的引用。
> LibBlockSrcSignalIsDiscrete(portIdx, sigIdx) 如果与指定模块输入端口元素对应的源信号是离散的，则返回1
> LibBlockSrcSignalIsGlobalAndModifiable(portIdx, sigIdx) 如果源信号可以满足以下三个条件，则返回1
> - 可以在认可地方都是可读的
> - 可以由其地址引用
> - 值可以更改
> LibBlockSrcSignalIsInvariant(portIdx, sigIdx)  如果与指定的模块输入端口元素对应的源信号不变，则返回1
> LibGetModelReferenceUserData(modelName)  获取给定模型的用户数据
> LibGetReferencedModelNames() 返回正在构建的模型名称
> - NumReferencedModels  模型数量，包含引用模块
> - ReferencedModes  结构体数组，其中每个结构体都包含一个字段(name) 指向引用模块
> LibIsModelReferenceRTWTarget() 如果构建过程正在为模块引用目标生成代码，则返回 1 
> LibIsModelReferenceSimTarget() 如果要为模型引用目标生成代码，则翻译 1 
> LibIsModelReferenceTarget() 如果我们要为模型引用目标生成代码，则返回1

 
