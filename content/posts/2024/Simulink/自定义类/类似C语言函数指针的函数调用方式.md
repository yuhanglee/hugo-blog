

```matlab 
c1 = class.type1.class1;
c1.def_class1_f1;  % def_class1_f1

cc.class1 = c1.def_class1_f1;
cc.class1;         % def_class1_f1
```

```matlab
c1 = class.type1.type3.class3;
c1.def_class1_f1;  % def_type3_class1_f1

cc.class1 = c1.def_class1_f1;
cc.class1;         % def_type3_class1_f1
```

