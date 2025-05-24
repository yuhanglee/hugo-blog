## 1. 错误
你应该使用以下技术来报告s函数中遇到的错误:
```matlab
ssSetErrorStatus(S,"error encountered due to ...");
return;
```


```ad-note
注意， ssSetErrorStatus 的第二个参数必须是持久内存。
它不能是过程中的局部变量。例如，以下操作将导致不可预测的错误:
```matlab
mdlOutputs()
{
	char msg[256];  {ILLEGAL: to fix use "static char msg[256];"}
	sprintf(msg,"Error due to %s", string);
	ssSetErrorStatus(S,msg);
	return;
}
```

## 2. 


