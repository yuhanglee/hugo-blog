Level 2 S-function methods

- => 是必须的方法
- [] 是可选的方法

## 1. Sinulink模型初始化方法

^0a2e74

> => mdlInitializeSizes   必须定义的方法 Initialize SimStruct sizes array

注意： S哈数不能同时使用 mdlSetInput(Output)PortWidth 和 mdlSetInput(Output)PortDimensionInnfo 两个方法，可以单独使用宽度或者维度，不可以同时使用。


>  => mdlInitializeSampleTimes 必须定义的方法

初始化采样时间和配置函数调用连接



> 【mdlSetInputPortWidth】   可选方法

检查和设置输入或者可配置的其他端口宽度。



> 【mdlSetOutputPortWidth】  可选方法

检查和设置输出和可配置的其他端口宽度



> 【mdlSetInputPortDimensionInfo】  可选方法

检查和设置输入或者可配置的其他端口维度。



> 【mdlSetOutputPortDimensionInfo】  可选方法

检查和设置输出或者可配置的其他端口维度。



> 【mdlSetDefaultPortDimensionInfo】  可选方法

设置所有未知维度的输入输出端口维度。



> 【mdlSetInputPortSampleTime】 可选方法

检查和设置输入端口 采样时间和可配置的其他端口采样时间。 

> 【mdlSetOutputPortSampleTime】 可选方法

检查和设置输出端口，采样时间和可配置的其他端口采样时间

> 【mdlSetInputPortDataType】 可选方法
> 【mdlSetOutputPortDataType】 可选方法
> 【mdlSetDefaultPortDataTypes】 可选方法

检查和设置输入端口类型。 有关内置数据类型，请参阅simstruc_types.h中的SS_DOUBLE到SS_BOOLEAN。


>   【mdlInputPortComplexSignal】 
>   【mdlOutputPortComplexSignal】
>   【mdlSetDefaultPortComplexSignals】

检查和设置输入端口复杂度属性(COMPLEX_YES, COMPLEX_NO)。


>  【mdlSetWorkWidths】  

设置状态，iwork, rwork, pwork, dword 等等宽度。

>  【mdlStart】

执行诸如分配内存和附加到pwork元素之类的操作。


>  【mdlInitializeConditions】

初始化模型参数(通常是状态)。如果s函数没有初始化条件方法，则不会被调用。

>  【'constant' mdlOutputs】

以恒定的采样时间执行块。这些在这里只执行一次。

>  【mdlSetSimState】

加载此块的完整模拟状态，当从初始模拟状态启动模拟时调用该块，并且此s-函数已将其sssetsimstatecomplcompliance设置为USE_CUSTOM_SIM_STATE。参见mdlGetSimState。


## 2. 模型仿真方法，可循环调用

>  【mdlCheckParameters】

将在仿真循环期间参数改变时随时调用。

``` C
#define MDL_CHECK_PARAMETERS
#if defined(MDL_CHECK_PARAMETERS) && defined(MATLAB_MEX_FILE)
/****
 * mdlCheckParameters 在模拟过程中参数更改或重新求值时验证新参数设置，在仿真时，可以在仿真循环期间的任何时间能更改 S 函数参数。
 * 
 * 可以在 mdlInitializeSizes 之后的任意时刻调用。一般是在 mdlInitializeSizes 中添加该方法的调用，以检查参数。
 * 通过 ssSetNumSFcnParams(S, n); 设置 S 函数中期望的参数数量之后：
 * #if defined(MATLAB_MEX_FILE)
 * if (ssGetNumSFcnParams(S) == ssGetSFcnParamsCount(S)) {
 *     mdlCheckParameters(S);
 *     if (ssGetErrorStatus(S) != NULL) return;
 * } else {
 *     return;     Simulink will report a parameter mismatch error
 * }
 * #endif
 * 
 * *
 ****/
static void mdlCheckParameters(SimStruct *S)
{

}
#endif
```






### SimulationLoop


>  【mdlGetSimState】

如果模型被配置为保存其最终模拟状态，并且此S-Function已将其sssetsimstatecomplcompliance设置为USE_CUSTOM_SIM_STATE，则调用该函数以获取此块的完整模拟状态。参见mdlSetSimState


## 3. 代码生成的模型初始化

初始化部分，可以参考  [Simulink 初始化部分](#^0a2e74) 

>  【mdlRTW】 

仅在生成代码以向模型添加信息时调用。Real-Time Workshop使用的rtw文件。

mdlTerminate 
模型的最后，释放内存等等。


## 4. 非内联s函数在实时工作环境中的执行

1） 大多数初始化防范的结果被“编译”到生成的代码中，许多方法不能被调用
2） 非内联 S 函数在几个方面收到限制，例如：参数必须是实数（非复数），双精度向量或者字符串。更多的功能通过目标语言编译器提供。


> => mdlInitializeSizes

初始化 SimStruct 结构体

> => mdlInitializeSampleTimes

初始化采样点和配置函数调用连接。


>  【mdlInitializeConditions】

初始化模型参数（通常是状态），如果s函数没有初始化条件方法，则不会被调用。

>  ‘【mdlStart】 

执行诸如分配内存和附加到 pwork 元素之类的操作。



### 循环执行
> => mdlOutputs

主要输出调用函数通常用来更新输出信号。

>  【mdlUpdate】

更新离散状态

``` matlab
Integration (Minor time step)
	[mdlDerivatives]         -  Compute the derivatives.
		 Do
			[mdlOutputs]
			[mdlDerivatives]
		EndDo - number of iterations depends on solver

		Do
			[mdlOutputs]
			[mdlZeroCrossings]
		EndDo - number of iterations depends on zero crossings signals
```


> mdlTerminate

模型结束函数，用来释放内存呢等操作。





## 4. 模型配置

> list = Simulink.sfunction.analyzer.findSfunctions(model)  2017a 开始支持
> model:  一个模型或者模型路径
> list: 符合条件的所有 SFunction 的列表

```matlab
sfuns = Simulink.sfunction.analyzer.findSfunctions("test")

sfuns =
  1×1 cell 数组
    {'sfun_fcncallgen'}

```


> sFunSettings = Simulink.SFunctionBuilder.getSettings(blk)   2022a开始支持
> blk  ： 模块路径或者句柄
> sFunSettings : 模块设置


> Simulink.SFunctionBuilder.generateCodeOnly(blk)  2022a开始支持
> 只编译S-Function，不执行MEX编译二进制文件
> 如果需要使用MEX ，则执行 `Simulink.SFunctionBuilder.build` 函数





> h = getSimulinkBlockHandle(path, openModel)
> path:  模块路径
> openModel:   是否打开模型，然后读取

```matlab
get_param(getSimulinkBlockHandle("test/S-Function"), "Name")

ans =
    'S-Function'
```