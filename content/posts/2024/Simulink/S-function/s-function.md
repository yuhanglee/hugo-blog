C/C++ 语言的 S-Function 函数说明

# S-Function 回调函数
## 输入输出

### 初始化
- mdlInitializeSizes  [R2006a]
> 执行 C-MEX 函数的输入、输出、状态、参数和其他特征的数量

使用 Simulink 引擎调用的第一个 S 函数回调方法。该方法执行以下任务：
1. 使用 ssSetNumSfcnParams 执行 S 函数支持的参数数量
  使用 ssSetSFcnParamTunalbe(S, paramIdx, 0)，当参数在模拟期间不能更改时，其中 paramIdx 从 0 开始。当参数被指定为不可调时，如果尝试修改承诺书，则引擎会在仿真期间发出错误。
2. 使用 ssSetNumContStates 和 ssSetNumDiscStates 指定此函数具有的状态数
3. 配置块的输入端口。
  使用 ssSetNumInputPorts 指定 S 函数拥有的输入端口数量
  指定输入端口的尺寸
  对于每个输入端口，使用 ssSetInputportDirectFeedThrough 指定是否具有直接馈通。
4. 配置块的输出端口
  使用 ssSetNumOutputPorts 指定块具有的输出端口数量
  指定输出端口的尺寸
  如果 S 函数是离散的，需要指定 SS_OPTION_DISCRETE_VALUED_OUTPUT。
5. 设置块运行时的采样次数
  有两种指定采样时间的方法
  - 基于端口的采样时间
  - 基于块的采样时间
  对于多速率 S 函数，建议通过基于端口的采样时间方法来设置采样时间。在创建多速率 S-Function 时，必须注意在抢占较慢的任务是否正确管理数据，以避免竟态条件。当指定基于端口的采样时间时，宽不能集成任何端口采样时间。设置块的工作向量大小，使用 ssSetNumRWork, ssSetNumIWork，ssSetNumPWork, ssSetNumModes, ssSetNumNonsampledZCs。
6. 使用 ssSetOptions 设置该块的仿真选项。
  所有选项的形式否是 SS_OPTION_XX 可以参考 【配置C/C++特性函数】，如： ssSetOptions(S, SS_OPTION_NAME1 | SS_OPTION_NAME2);

动态大小的块特征。可以设置参数 NumContStates, NumDiscStates, NumInputs, NumOutputs, NumRWork, NumIWord, NumPWord, NumModes, NumNonsampledZCs为一个固定的肺腑证书，以动态调整它们的大小。
将状态、工作向量等长度设置为从驱动块集成的值。可以使用 mdlSetWorkWidths 来设置宽度。宽度可以是0或整数。

示例函数
``` C
#define MDL_INITIAL_SIZES
#if defined(MATLAB_MEX_FILE)
static void mdlInitializeSizes(SimStruct *S)
{
  int_T nInputPorts = 1; /* number of input ports */ 
  int_T nOutputPorts = 1; /* number of output ports */ 
  int_T needsInput = 1; /* direct feedthrough */ 
  int_T inputPortIdx = 0; 
  int_T outputPortIdx = 0; 
  
  ssSetNumSFcnParams(S, 0); /* Number of expected parameters */ 
  if (ssGetNumSFcnParams(S) != ssGetSFcnParamsCount(S)) { 
    /* 
     * If the number of expected input parameters is not 
     * equal to the number of parameters entered in the 
     * dialog box, return. The Simulink engine generates an 
     * error indicating that there is aparameter mismatch. 
     **/ 
     return; 
  } else { 
    mdlCheckParameters(S); 
    if (ssGetErrorStatus(S) != NULL) return; 
  } 
  ssSetNumContStates( S, 0); 
  ssSetNumDiscStates( S, 0); 
  
  /* 
   * Configure the input ports. First set the number of input 
   * ports. 
   **/ 
  if (!ssSetNumInputPorts(S, nInputPorts)) return; 
  /* 
   * Set input port dimensions for each input port index 
   * starting at 0.
   */
  if(!ssSetInputPortDimensionInfo(S, inputPortIdx, DYNAMIC_DIMENSION)) return; 
  /* 
   * Set direct feedthrough flag (1=yes, 0=no). 
   **/ 
  ssSetInputPortDirectFeedThrough(S, inputPortIdx, needsInput); 
  /* 
   * Configure the output ports. First set the number of 
   * output ports. 
   **/ 
  if (!ssSetNumOutputPorts(S, nOutputPorts)) return; 
  /* 
   * Set output port dimensions for each output port index 
   * starting at 0. 
   **/ 
  if(!ssSetOutputPortDimensionInfo(S,outputPortIdx, DYNAMIC_DIMENSION)) return; 
  /* 
   * Set the number of sample times. 
   **/ 
  ssSetNumSampleTimes(S, 1); 
  /* 
   * Set size of the work vectors. 
   **/ 
  ssSetNumRWork(S, 0); /* real vector */ 
  ssSetNumIWork(S, 0); /* integer vector */ 
  ssSetNumPWork(S, 0); /* pointer vector */ 
  ssSetNumModes(S, 0); /* mode vector */ 
  ssSetNumNonsampledZCs(S, 0); /* zero crossings */ 
  ssSetOptions(S, 0); 
} /* end mdlInitializeSizes */
#endif
```

- mdlInitializeSampleTimes  [R2006a]
> 指定 C-MEX 函数运行的采样率

该方法应该指定每个采样率的采样时间和品阿姨时间，该S函数通过以下成对的宏定义进行操作：
```C
// sampleTimeIndex 从 0 开始运行
ssSetSampleTime(S, sampleTimeIndex, sampleTime)
ssSetOffsetTime(S, offsetTimeIndex, offsetTime)
```
如果 S 函数以一个或多个采样率执行，该方法可以为指定的采样时间指定以下任意采样时间和偏移值。
	- [CONTINUOUS_SAMPLE_TIME, 0.0]
	- [CONTINUOUS_SAMPLE_TIME, FIXED_IN_MINOR_STEP_OFFSET]
	- [discrete_sample_period, offset]
	- [VARIABLE_SAMPLE_TIME, 0.0]
以上的值，可以在 `sl_sample_time_defs.h` 中查看。
如果S函数以一个速率运行，该方法可以选择将采样时间设置为以下采样/偏移时间对之一。
	- [CONTINUOUS_SAMPLE_TIME, 0.0]
	- [CONTINUOUS_SAMPLE_TIME, FIXED_IN_MINOR_STEP_OFFSET]
如果采样次数为 0，则 Simulink 引擎嘉定 S 函数继承其从所链接的块开始的采样时间，即采样时间为 [INHERITED_SMPLE_TIME, 0]
因此，这个函数可以不做任何事情就返回。
在指定采样时间试用以下指导原则：
1. 在小的集成步骤中发生变化的连续函数应该将采样时间设置为 [CONTINNUOUS_SAMPLE_TIME, 0.0]
2. 在小的集成步骤中没有变化的连续函数应该将采样时间设置为 [CONTINUOUS_SAMPLE_TIME, FIXED_IN_MINOR_STEP_OFFSET]
3. 以指定速率变化的离散函数应将采样时间设置为 [discrete_sample_period, offset]，其中  discrete_sample_period > 0.0 并且 0 <= offset < discrete_sample_period
4. 以可变速率变化的离散函数应将采样时间设置为 [VARIABLE_SAMPLE_TIME, 0]，注意，VARIABLE_SAMPLE_TIME 需要可变步长求解器。
5. 为了在触发子系统或周期任务中正确运行，离散 S 函数应该
	1. 指定单个采样时间设置为 [INHERITED_SAMPLE_TIME, 0]
	2. 使用 ssSetOptions 在 SS_OPTION_DISALLOW_CONSTANT_SAMPLE_TIME 选项
	3. 验证在 mdlSetWorkWidths 分配李离散和触发的采样时间
				
		```C
		if (ssGetSampleTime(S, 0) == CONTINUOUS_SAMPLE_TIME) {
		  ssSetErrorStatus(S, "This block cannot be assigned a continuous sample time");
		}	
		```

在整个狂徒中传播采样时间后，引擎分配采样时间 [INHERITED_SAMPLE_TIME, INHERITED_SAMPLE_TIME] 到主流在被触发子系统的离散块。
如果此函数没有固有采样时间，则应按照以下准则，将其采样是按斤设置为 inherited:
1. 随着输入的变化而变化的函数，集市在叫嚣的积分步骤中，也应该将其采样时间设置为 [inherited_samople_time, 0] 随着输入的比那话而变化的函数，也应该在小的积分步骤中不会变化，采样时间应该设置为[INHERITED_SAMPLE_TIME, FIXED_IN_MINOR_STEP_OFFSET]。
  if (ssIsContinuousTask(S, tid))
  {
  }
如果有 Simulink Coder 在为包含此方法的非内敛函数生成代码。
``` C
#if defined(MATLAB_MEX_FILE)
#define MDL_INITIALIZE_SAMPLE_TIMES
static void mdlInitializeSampleTimes(SimStruct *S)
{

}
#endif
```
宏定义可以保证 mdlInitializeSampleTimes 仅仅对 mex 文件可用。


### 输出
- mdlOutputs  [R2006a]
> 计算输出 
> S： 块结构体
> tid:  任务 ID

Simulink 引擎在每个仿真时间步调用这个所需要的方法，该方法应该计算 S 函数在当前时间不愁的输出，并将结果存储在 S 函数的输出信号数组中。
tid 参数指定在调用 mdlOutputs 例程运行的任务。
如果 S 函数不包含特定的任务块，则使用 UNUSED_ARG 宏来说明 tid 输入参数是必须的，但是不在回调函数的主体中使用。需要在 mdlOutputs 中生声明 UNUSED_ARG(tid) 。

示例可以参考 `sfun_multiport.c` 文件。
``` C
#define MDL_OUTPUTS
#if defined(MATLAB_MEX_FILE)
static void mdlOutputs(SimStruct *S)
{
  // Add MdlOutputs code here.
}
#endif
```

- mdlUpdate
- mdlTerminate

### 信号规范
- mdlSetDefaultPortComplexSignals  [R2006a]
> 设置不能通过块连通性，确定端口的复杂信号类型

如果块具有无法从连接线确定数字类型的端口，则 Simulink 引擎调用此方法。这通常发生在块未连接时。此方法仅对仿真模式有效。
示例：
``` C
#define MDL_SET_DEFAULT_PORT_COMPLEX_SIGNALS
void mdlSetDefaultPortComplexSignals(SimStruct *S)
{

}
```

- mdlSetDefaultPortDataTypes  [R2006a]
> 设置不能通过块连通性，确定端口的数据类型

参考 mdlSetDefaultPortComplexSignals 函数。
默认端口的数据类型都是 double ，如果没有实现此方法，并且引擎无法确定端口的数据类型，则引擎将未知端口设置为最大的端口的数据类型。

示例：
``` C
#define MDL_SET_DEFAULT_PORT_DATA_TYPES
void mdlSetDefaultPortDataTypes(SimStruct *S)
{

}


if (ssGetInputPortDataType(S, 1) == DYNAMICALLY_TYPED) {
  ssSetInputPortDataType(S, 1, SS_UINT8 ); 
}
```

- mdlSetDefaultPortDimensionInfo
> 设置 C-MEX 函数的端口接受或发出的信号的默认尺寸

如果模型不能提供足够信息来确定输入或输出的信号维度是，Simuli 引擎将调用此方法。可以讲任何输入输出端口的位数设置为动态大小的值。

``` C
if (ssGetOutputPortWidth(S, 0) == DYNAMICALLY_SIZED) {
  ssSetOutputPortMatrixDimensions(S, 0, 1, 1 ); 
}
```

- mdlSetInputPortComplexSignal
> 设置输入端口接受的信号的复杂类型
> S:  块的参数结构体
> port： 端口索引，从0开始
> csig： 数据的类型， COMPLEX_ON（实数）或 COMPLE_YES（复合数）

例程： 参考 sdotproduct.c 文件

- mdlSetInputPortDataType
> 设置输入信号的端口数据类型
> S: 块的参数结构体
> port:  端口索引
> id： 数据类型 ID。可以zai simstruc_types.h 中获取

必须先检查指定的数据类型是否为指定端口的有效数据类型，如果是，则 C-MEX 函数使用 ssSetInputPortDataType 函数设置输入端口的数据类型。

- mdlSetInputPortSampleTime
- mdlSetOutputPortComplexSignal  [R2006a]
- mdlSetOutputPortDataType  [R2006a]
- mdlSetOutputPortSampleTime  [R2006a]

### 信号维度
- mdlSetInputPortDimensionInfo [R2006a]
> 设置输入端口接受的信号维度信息
> S:  块的参数结构
> port： 端口索引
> dimsInfo： 指定端口支持的信号信息



- mdlSetInputPortDimensionsModeFcn  [R2009a]
> 传播维度模式
> S:  块的参数结构
> port： 端口索引
> dimsMode： 当前维度模式。取值范围： INHERIT_DIMS_MODE, FIXED_DIMS_MODE, VARIABLE_DIMS_MODE.

可以调用此函数，来设置端口索引的维度模式。


- mdlSetInputPortWidth  [R2006a]
> 设置信号的输入端口维度
> S:  块的参数结构
> port:  端口索引
> width： 信号宽度

使用动态大小端口的候选宽度调用此方法。如果建议的宽度是可接受的，该方法应该使用 ssSetInputPortWidth 设置实际的端口宽度。


- mdlSetOutputPortDimensionInfo  [R2006a]
- mdlSetOutputPortWidth  [R2006a]


### 信号访问
- mdlSetWorkWidths
> 指定工作向量的大小，并创建 C-MEX 函数所需的运行时参数



## 采样时间
- mdlSetInputPortSampleTime
> 设置输入端口的采样时间，以继承所连接的端口采样时间。
> S:  块的参数结构
> port： 端口索引
> sampleTime： 采样时间
> offsetTime： 偏移时间

Simulink 引擎使用输入端口，从其连接的端口继承示例时间调用此方法。只对仿真有效。


- mdlSetOutputPortSampleTime

## 运行时参数
- mdlSetWorkWidths
- mdlCheckParameters  [ R2006a ]
> 检查 C-MEX 函数参数是否有效

在模拟过程种，需要更改参数或重新求值，验证新的参数设置。对于 C-MEX 函数，仅对模拟有效。
``` C
#define MDL_CHECK_PARAMETERS
// void mdlCheckParameters(SimStruct *S);

// 获取第一个参数
#define PARAM1(S) ssGetSFcnParam(S, 0)
#define MDL_CHECK_PARAMETERS
#if defined(MDL_CHECK_PARAMETERS) && defined(MTLAB_MEX_FILE)
static void mdlCheckParameters(SimStruct *S)
{
  if (mxGetNumberOfElements(PARAMS(S)) != 1) {
    ssSetErrorStatus(S, "Parameter to S-funtcion must be a scalar");
    return ;
  } else if (mxGetPr(PARAM1(S))[0] < 0) {
    ssSetErrorStatus(S, "Parameter to S-function must be nonnegative");
  }
}
#endif
```

除了前面的李例程之外，还必须从 mdlInitializeSizes 函数添加对该方法的调用，以便在初始化期间检查参数，因为 mdlCheckParameters 只在模拟运行时调用。要做到这一点，在使用 ssSetNumSFcnParams 设置 SFunction 中期望的参数数量之后，在 mdlInitializeSizes 中使用以下代码：
``` C
static void mdlInitializeSizes(SimStruct *S)
{
  // 设置期望参数个数
  ssSetNumSFcnParams(S, 1);

  #if defined(MTLAB_MEX_FILE) 
    // 判断实际参数和期望参数是否一致
    if (ssGetNumSFcnParams(S) == ssGetSFcnParamsCount(S)) {
      mdlCheckParameters(S);
      if (ssGetErrorStatus(S) != NULL) {
	    return ;
      }
    } else {
      return ;
    }
  #endif
}
```

示例可以参考 See sfun_errhdl.c 

- mdlProcessParameters
> 处理 C-MEX 函数的参数

这个是一个可选例程，在 mdlCheckParameters 更改并验证参数后， Simulink 引擎将调用它。 处理在仿真循环的顶部完成。此时处理更改的参数是安全的，此函数仅对仿真有效。
该函数目的是处理新更高的参数。一个例子是缓存向量中的参数变化。

例程：
处理一个字符向量参数 mdlCheckParameters 已经验证该参数的形式为 '+++' （其中可以有任意数量的 '+' 和 '-' 字符）。
``` C
#define MDL_PROCESS_PARAMETERS /* Change to #undef to remove function */ 
#if defined(MDL_PROCESS_PARAMETERS) && defined(MATLAB_MEX_FILE) 
static void mdlProcessParameters(SimStruct *S) 
{ 
  int_T i; 
  char_T *plusMinusStr;
  int_T nInputPorts = ssGetNumInputPorts(S); 
  int_T *iwork = ssGetIWork(S); 
  
  if ((plusMinusStr = (char_T*)malloc(nInputPorts+1)) == NULL) { 
    ssSetErrorStatus(S,"Memory allocation error in mdlStart"); 
    return; 
  } 
  
  if (mxGetString(SIGNS_PARAM(S), plusMinusStr, nInputPorts+1) != 0) { 
    free(plusMinusStr); 
    ssSetErrorStatus(S,"mxGetString error in mdlStart"); 
    return; 
  } 
  
  for (i = 0; i < nInputPorts; i++) { 
    iwork[i] = plusMinusStr[i] == '+'? 1: -1; 
  } 
  free(plusMinusStr); 
} 
#endif /* MDL_PROCESS_PARAMETERS */
```
在仿真循环开始之前，从 mdlStart 调用 mdlProcessParameters 来加载符号字符向量。
``` C
#define MDL_START 
#if defined(MDL_START) 
static void mdlStart(SimStruct *S) {
  mdlProcessParameters(S); 
} 
#endif /* MDL_START */
```

## 模型引用
- mdlStart
- mdlProcessParameters
- mdlSetWorkWidths


## 与 Simulink 引擎交互

### 仿真信息
- mdlGetOperatingPoing
- mdlSetOperatingPoing

### 错误处理
- mdlStart
- mdlTerminate

### 状态与工作向量
- mdlSetWorkWidths
- mdlZeroCrossings
- mdlInitializeConditions  [R2006a]
> 初始化 C-MEX 的状态向量

Simulink 引擎在模拟开始时，调用该方法。在初始化这个函数的连续和离散状态。在 C-MEX 函数中，可以使用 ssGetContStates 和 ssGetDisStates 来访问状态。此方法还可以执行此函数所需的任何其他初始化活动。
需要保证只初始化一次。

``` C
#define MDL_INITIALIZE_CONDITIONS
#if defined(MDL_INITIALIZE_CONDITIONS) && defined(MATLAB_MEX_FILE) 
static void mdlInitializeConditions(SimStruct *S) {
  // 设置 IWork 的标志。
  ssSetIWorkValue(S, 0, 1);
}
#endif
```

在计算 S 函数的输入信号之前，Simulink引擎调用 mdlInitalizConditions 函数。由于此时输入信号值还不可用，mdlInitalizConditions 不应该使用输入信号值来设置初始条件。如果 S 函数需要使用块的输入信号初始化内部数据，需要在 mdlOutputs 中执行。

例如： 在 C-MEX 函数中，使用该方法初始化 IWork 线路：
`ssSetNumIWork(S, 1);` 
IWork 向量持有一个标志。

检查 mdlOutputs 函数中的 IWork 标志的值，已确定是否需要重新设置。

``` C
static void mdlOutputs(SimStruct *S, int_T tid) 
{ 
  // Initialize values if the IWork vector flag is true. 
  // 
  if (ssGetIWorkValue(S, 0) == 1) 
  { 
    // Enter initialization code here // 
  } 
  // Remainder of mdlOutputs function // 
}
```

示例：
``` C
#define MDL_INITIALIZE_CONDITIONS 
/*Change to #undef to remove */ 
/*function*/ 
#if defined(MDL_INITIALIZE_CONDITIONS)
static void mdlInitializeConditions(SimStruct *S) 
{ 
  int i; 
  real_T *xcont = ssGetContStates(S); 
  int_T nCStates = ssGetNumContStates(S); 
  real_T *xdisc = ssGetRealDiscStates(S); 
  int_T nDStates = ssGetNumDiscStates(S); 

  for (i = 0; i < nCStates; i++) { 
    *xcont++ = 1.0; 
  } 
  for (i = 0; i < nDStates; i++) { 
    *xdisc++ = 1.0; 
  } 
} 
#endif 
/* MDL_INITIALIZE_CONDITIONS */
```

# SimStruct 函数


## 输入输出
### 信号规格
- ssSetNumInputPorts
- ssSetInputPortComplexSignal
- ssSetInputPortDataType
- ssSetInputPortDirectFeedThrough
- ssSetInputPortOffsetTime
- ssSetInputPortRequiredContiguous
- ssSetInputPortSampleTime
- ssRegisterUnitFromExpr
- ssSetInputPortUnit
- ssSetNumOutputPorts
- ssSetOutputPortComplexSignal
- ssSetOutputPortDataType
- ssSetOutputPortOffsetTime
- ssSetOutputPortSampleTime
- ssSetOutputPortUnit
- ssGetInputPortCompleSIgnal
- ssGetInputPortDataType
- ssGetInputPortDirectFeedThrough
- ssGetInputPortFrameData
- ssGetInputPortOffsetTime
- ssGetInputPortRequiredContiguous
- ssGetInputPortSampleTime
- ssGetInputPortSampleTimeIndex
- ssGetInputPortUnit
- ssGetOutputPortComplexSignal
- ssGetOutputPortDataType
- ssGetOutputPortFrameData
- ssGetOutputPortOffsetTime
- ssGetOutputPortUnit
- ssAllowSignalsWithMoreThan2D
- ssSetOneBasedIndexInputPort
- ssSetOneBasedIndexOutputPort
- ssSetZeroBasedIndexInputPort
- ssSetZeroBasedIndexOutputPort

### 信号访问
- ssGetNumIntputPorts
- ssGetNumOutputPorts
- ssGetInputPortBufferDstPort
- ssGetInputPortConnected
- ssGetInputPortOptimOpts
- ssGetInputPortOverWritable
- ssGetInputPortRealSignal
- ssGetInputPortRealSignalPtrs
- ssGetInputPortSignal
- ssGetOutputPortConnected
- ssGetOutputPortBeingMerged
- ssGetOutputPortOptimOpts
- ssGetOutputPortRealSignal
- ssGetOutputPortSignal
- ssSetInputPortOptimOpts
- ssSetInputPortOverWritable
- ssSetOutputPortOptimOpts
- ssSetoutputPortOverwitesInputPort
- ssSetInputPortAcceptExprInRTW
- ssGetInputPortAcceptExprInRTW
- ssSetOutputPortOutputExprInRTW
- ssGetOutputPortOutputExprInRTW

### 信号尺寸
- ssSetInputPortDimensionInfo
- ssSetInputPortDimensionsMode
- ssSetInputPortDimsSameAsOutputPortDims
- ssSetInputPortMatrixDimensions
- ssSetInputPortVectorDimension
- ssSetInputPortWidth
- ssPruneNDMatrixSigneltonDims
- ssSetCurrentOutputPortDimesions
- ssSetOutputPortDimensionInfo
- ssSetOutputPortDimesionsMode
- ssSetOutputPortMatrixDimensions
- ssSetOutputPortVectorDimension
- ssSetOutputPortWidth
- ssSetOutputPortMatrixDimensions
- ssAddOutputDimsDependencyRule
- ssAddVaiableSizeSignalIsRuntimeChecher
- ssAllowSignalsWithMoreThen2D
- ssRegMdlSetInputPortDimensionsModeFcn
- ssSetDWorkReguireResetForSignalSize
- ssSetSignalSizesComputeType
- ssSetVectorMode
- ssGetCurrentInputPortDimensions
- ssGetCurrentInputPortWidth
- ssGetInputPortDimensionSize
- ssGetInputPortDimensionsMode
- ssGetInputPortNumDimensions
- ssGetInputPortWidth
- ssGetCurrentOutputPortDimensions
- ssGetCurrentOutputPortWidth
- ssGetOutputPortDimensions
- ssGetOutputPortDimensionsSize
- ssGetOutputPortDimensionsMode
- ssGetOutputPortNumDimensions
- ssGetOutputPortWidth

### 信号区域
- ssCallSelectedSignalsFcn
- ssCallSigListCreateFcn
- ssCallSigListDestroyFcn
- ssCallSigListUnavailSigAlertFcn
- ssCallGenericDestroyFcn
- ssCallUnselectSigFcn
- gsl_FirstReg
- gsl_nSigRegions
- gsl_nSigs
- gsl_NumElements
- gsl_PortObj
- gsl_TimWrap
- gsr_Complex
- gsr_currDims
- gsr_data
- gsr_DataType
- gsr_DataTypeSize
- gsr_Dims
- gsr_nEls
- gsr_NextReg
- gsr_NumDims
- gsr_portObj
- gsr_startIdx
- gsr_status
- gsr_Unit
- gsr_M
- gsr_N

## 块对话框参数
- ssGetDTypeIdFromMxArray
- ssGetNumSFcnParams
- ssGetSFcnParam
- ssGetSFcnParamsCount
- ssSetNumSFcnParams
- ssSetSFcnParamTunable


## 数据类型和采样时间

### 采样时间

- ssSetSampleTime
- ssSetInputPortSampleTime
- ssSetModelReferenceSampleTimeDefaultInheritance
- ssSetModelReferenceSampleDisallowInheritance
- ssSetModelReferenceSampleTimeInheritanceRule
- ssSetNumSampleTimes
- ssSetTNext
- ssSetVariableDiscreteSampleTime
- ssSetNumTickstoNextHigForVariableDiscrete
- ssSetNeedAbsoluteTime
- ssSetTimeSource
- ssGetInputPortSampleTime
- ssGetInputPortSampleTimeIndex
- ssGetNumSampleTimes
- ssGetOutputPortSampleTime
- ssGetOutputPortSampleTimeIndex
- ssGetPortBasedSampleTimeBlockIsTriggered
- ssGetSampleTime
- ssGetTNext
- ssSetParameterTuningCompliance
- ssGetParameterTuningCompliance
- ssIsContinuousTask
- ssIsSampleHit
- ssIsSpecialSampleHig
- ssSampleAndOffsetAreTriggered
- ssSampleAndOffsetAreTriggeredOrAsync
- ssSetAsyncTimerDataType
- ssSetAsyncTaskPriorities

### 数据类型
- ssRegisterDataType
- ssRegisterTypeFromNamedObject
- ssSetDataTypeSize
- ssSetDataTypeZero
- ssSetInputPortDataType
- ssSetOutputPortDataType
- ssGetDataTypeId
- ssGetDataTypeIdAliasedThruTo
- ssGetDataTypeName
- ssGetDataTypeSize
- ssGetDataTypeZero
- ssGetInputPortDatatType
- ssGetNumDataTypes
- ssGetOutputPortDataType
- ssGetOutputPortSignal
- ssGetSFcnParamDataType

### 总线

- ssGetBusElementComplexSignal
- ssGetBusElementDataType
- ssGetBusElementDimensions
- ssGetElementName
- ssGetBusElementNumDimensions
- ssGetBusElementOffset
- ssGetNumBusElements
- ssGetSFcnParamName
- ssIsDataTypeABus
- ssRegisterTypeFromParameter
- ssRigisterTypeFromNamedObject
- ssSetBusInputAsStruct
- ssSetBusOuptutAsStruct
- ssSetBusOutputObjectName


## 运行时参数

- ssGetNumRunTimeParams
- ssGetRunTimeParamInfo
- ssRegAllTunableParamsAsRunTimeParams
- ssRegDigParamAsRunTimeParam
- ssSetNumRunTimeParams
- ssSetRunTimeParamInfo
- ssUpdateAllTunableParamsAsRunTimeParams
- ssUpdataDlgParamAsRunTimeParam
- ssUpdateRunTimeParamData
- ssUpdateRunTimeParamInfo


## 函数调用子系统，Simulink函数和模型参考

### 函数调用子系统

- ssCallSystemWithTid
- ssDisableSystemWithTid
- ssEnableSystemWithTid
- ssGetCallSystemNumFcnCallDestinations
- ssGetExplicitFCSSCtrl
- ssSetCallSystemOutput
- ssSetExplicitFCSSCtrl

### Simulink 函数

- ssDeclareSimulinkFunction
- ssDeclareFunctionCaller
- ssCallSimulinkFunction
- ssQuerySimulinkFunction
- ssGetSimulinkFunctionInput
- ssGetSimulinkFunctionOutput
- ssSetSimulinkFunctionArgComplexity
- ssSetSimulinkFunctionArgDataType
- ssSetSimulinkFunctionArgDimensions

### 模型引用

- ssRTWGenIsModelReferenceRTWTarget
- ssRTWGenIsModelReferenceSimTarget
- ssSetModelReferenceNormalModeSupport
- ssSetModelReferenceSampleTimeDefaultInheritance
- ssSetModelReferenceSampleTimeDisallowInheritance
- ssSetModelReferenceSampleTimeInheritanceRule

## 与 Simulink 引擎交互

### 仿真信息

- ssSetStateAbsTol
- ssSetBlockReduction
- ssSetOperatingPointCompliance
- ssSetOperatingPointVisibility
- ssSetSolverNeedsReset
- ssSetSkipContStatesConsistencyCheck
- ssSetStopRequested
- ssGetBlockReduction
- ssGetErrorStatus
- ssGetFixedStepSize
- ssGetMaxStepSize
- ssGetMinStepSize
- ssGetSimMode
- ssGetSimStatus
- ssGetSolverMode
- ssGetSolverName
- ssGetStateAbsTol
- ssGetStopRequested
- ssGetT
- ssGetTaskTime
- ssGetTFinal
- ssGetTNext
- ssGetTStart
- ssIsExternalSim
- ssIsFirstInitCond
- ssIsMajorTimeStep
- ssIsMinorTimeStep
- ssIsVariableStepSolver
- ssRTWGenIsAccelerator
- ssIsRapidAcceleratorActive

### 错误处理

- ssGetErrorStatus
- ssGetLocalErrorStatus
- ssPrintf
- ssSetErrorStatus
- ssSetLocalErrorStatus
- ssWarning

### 信息与选项

- ssSetOptions
- ssCallExternalModeFcn
- ssGetModelName
- ssGetParentSS
- ssGetPath
- ssGetRootSS
- ssSetExternalModeFcn
- ssSetPlacementGroup
- ssSetUserData
- ssSupportsMultipleExecInstances

## 状态与工作向量

- ssSetNumDWork
- ssSetDWorkComplexSignal
- ssSetDWorkDataType
- ssSetDWorkName
- ssSetDWorkUsageType
- ssSetDWorkUsedAsDState
- ssSetDWorkWidth

- ssGetNumDWork
- ssGetDWork
- ssGetDWorkComplexSignal
- ssGetDWorkDataType
- ssGetDWorkName
- ssGetDWorkUsageType
- ssGetDWorkUsedAsDState
- ssGetDWorkWidth
- ssGetdX


- ssSetNumContStates
- ssSetNumDiscStates
- ssSetNumNonsampledZCs

- ssGetContStates
- ssGetDiscStates
- ssGetRealDiscStates
- ssGetNonsmpledZCs
- ssGetNumContStates
- ssGetNumDiscStates
- ssGetNumNorsampledZCs

- ssSetNumRWork
- ssGetNumRWork
- ssGetRWork
- ssGetRWorkValue
- ssGetRWOrkValue
- ssSetRWorkValue
- ssSetNumIWork
- ssGetNumIWork
- ssGetIWork
- ssGetIWorkValue
- ssSetIWorkValue
- ssSetNumPWork
- ssGetNUmPWork
- ssGetPWork
- ssGetPWorkValue
- ssSetPWorkValue
- ssSetNumModes
- ssGetNumModes
- ssGetModeVector
- ssGetmodeVectorValue
- ssSetModeVectorValue


## 代码生成

- ssGetDWorkRTWIdentifier
- ssGetDWOrkRTWIdentifierMustResolveToSignalObject
- ssGetDWorkRTWStorageClass
- ssGetDWorkRTWTypeQualifier
- ssGetPlacementGroup
- ssRTWGenIsCodeGen

- ssSetArrayLayoutForCodeGen
- ssSetDWorkRTWIdentifier
- ssSetDWorkRTWIdentifierMustResolveToSignalObject
- ssSetDWorkRTWStorageClass
- ssSetDWorkRTWTypeQualifier
- ssSetPlacementGroup

- ssWriteRTW2dMatParam
- ssWriteRTWMx2dMatParam
- ssWriteRTWMxVectParam
- ssWriteRTWParameters
- ssWriteRTWParamSettings
- ssWriteRTWScalarParam
- ssWriteRTWStr
- ssWriteRTWStrParam
- ssWriteRTWStrVectParam
- ssWriteRTWVectParam
- ssWriteRTWWorkVect


## 入口函数

- mdlDerivatives [R2006a]
> 计算 C-MEX 的函数的导数

Simulink 引擎在每个时间步调用这几个方法来计算 S 函数连续状态的导数，这种方法应该将倒数存储在 S 函数的状态导数向量中，在 C-MEX 文件中，可以使用 ssGetdX 来获取指向导数向量的指针。
每次调用此函数，必须显示设置所有导数的值。导数向量不保存上次调用此例程的值，只在分配给导数向量的内存执行期间发生变化。

``` C
#define MDL_DERIVATIVES
#if defined(MDL_DERIVATIVES) && defined(MATLAB_MEX_FILE)
static void mdlDerivatives(SimStruct *S)
{

}
#endif
```

- mdlDisable/mdlEnable [R2006a]
> 包含此块的已启用系统的禁用回复

如果此块驻留在已启用的子系统中，则 Simulink 引擎调用此方法，启用的子系统在当前时间步，从启用状态变为禁用状态。

- mdlGetOperatingPoint  [R2019a]
> 返回 C-MEX 函数的模拟操作点作为 MATLAB 的数据结构。

Simulink 引擎可以调用这个自定义方法来获取包含 S 模型的模拟操作点。该方法调用发生在 mdlStart 之后和 mdlTerminate 之前，以确保所有的 S 函数数据结构都是可用的。

``` C
/* Function: mdlGetOperatingPoint 
 * Abstract: 
 * Package the RunTimeData structure as a MATLAB structure 
 * and return it. 
 */ 
static mxArray* mdlGetOperatingPoint(SimStruct* S) 
{ 
  RunTimeData_T* rtd = (RunTimeData_T*)ssGetPWorkValue(S, 0); 
  const char* fieldNames[] = {"Count"}; 
  /* Create a MATLAB structure to hold the run-time data */ 
  mxArray* simSnap = mxCreateStructMatrix(1, 1, 1, fieldNames); 
  mxSetField(simSnap, 0, fieldNames[0], mxCreateDoubleScalar(rtd->cnt)); 
  return simSnap; 
}
```


- mdlGetSimState [R2009a]
> 不推荐使用。可以使用 mdlGetOperatingPoint 代替
> 将 C-MEX 函数模拟状态范围为有效的 MTATLAB 数据结构。

Simulink 引擎调用该方法来获取 S 模型的模拟状态。

``` C
/* Function: mdlGetSimState 
 * Abstract: 
 * Package the RunTimeData structure as a MATLAB structure 
 * and return it. 
 */ 
#define MDL_SIM_STATE
static mxArray* mdlGetSimState(SimStruct* S)
{
  RunTimeData_T *rtd = (RunTimeData_T*) ssGetPWorkValue(S, 0);
  const char * fieldNames[] = {"Count"};
  /* Create a MATLAB structure to hold the run-time data */
  mxArray * simSnap = mxCreateStructMatrix(1, 1, 1, fieldNames);
  mxSetField(simSnap, 0, fieldNames[0], mxCreateDoubleScalar(rtd->cnt));
  return simSnap;
}
```

- mdlGetTimeOfNextVarHit  [R2006a]
> 确定下一个样本时间命中的时间

在 S-Funtcion 注册的变量样本时间达到最大值后， Simulink 引擎在主要时间步调用这个可选方法。

``` C
#define MDL_GET_TIME_OF_NEXT_VAR_HIT
static void mdlGetTimeOfNextVarHit(SimStruct *S) 
{ 
  time_T offset = getOffset(); 
  time_T timeOfNextHit = ssGetT(S) + offset;
  ssSetTNext(S, timeOfNextHit);
}
```


- mdlProjection [R2006a]
> 对系统的解进行摄动，以更好的满足定常解关系。

需要时，查看文档。


- mdlRTW
> 为 C-MEX 函数生成代码

该函数在 Simulink Coder 产品生成模型RTW文件时调用。可以用以下函数，来向模型RTW文件添加字段。
	- ssWriteRTWParameters 
	- ssWriteRTWParamSettings 
	- ssWriteRTWWorkVect 
	- ssWriteRTWStr 
	- ssWriteRTWStrParam 
	- ssWriteRTWScalarParam 
	- ssWriteRTWStrVectParam 
	- ssWriteRTWVectParam 
	- ssWriteRTW2dMatParam 
	- ssWriteRTWMxVectParam 
	- ssWriteRTWMx2dMatParam

- mdlSetOperatingPoint  [R2019a]
> 设置 C-MEX 的恢复工作点
> S： 块的参数结构体
> const mxArraay* in： 由 mdlGetOperatingPoing 创建的 S 函数的操作点

Simulink 引擎在包含 S 模型的模拟开始调用这个自定义方法。 mdlSetOperatingPoint 将 S 函数的初始模拟状态设置为模型的工作点。

例程：
``` C
/* Function: mdlSetOperatingPoint 
 * Abstract: 
 * Unpack the MATLAB structure passed and restore it to 
 * the RunTimeData structure 
 **/ 
static void mdlOperatingPoint(SimStruct* S, const mxArray* simSnap) 
{ 
  RunTimeData_T* rtd = (RunTimeData_T*)ssGetPWorkValue(S, 0); 
  
  /* Check and load the count value */ 
  { 
    const mxArray* cnt = mxGetField(simSnap, 0, fieldNames[0]); 
    ERROR_IF_NULL(S,cnt,"Count field not found in simulation state"); 
    if ( mxIsComplex(cnt) || !mxIsUint64(cnt) || mxGetNumberOfElements(cnt) != 1 ) { 
      ssSetErrorStatus(S, "Count field is invalid"); 
      return; 
    } 
    rtd->cnt = ((uint64_T*)(mxGetData(cnt)))[0]; 
  } 
}
```


- mdlSetSimState
> 不推荐
> 通过恢复 SimState 设置 C-MEX 的仿真状态


