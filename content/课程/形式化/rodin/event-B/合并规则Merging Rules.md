1. M_IF
   把共同的条件`P`放在一起, 用`if else`分别处理两种对立情况
   ### progress1与progress2结合后得到progress_1_2
   ![[assets/条件分支转换图.png]]
2. Serialisation串行化
   将并行的赋值语句转化为串行执行的程序语言
   给语句添加`;`
   如果只有一条赋值语句就不用进行串行化
   ### init转化为init_ser, progress_1_2转化为progress_1_2_ser
   ![[assets/初始化序列化对比.png]]
3. M_WHILE
   `P`必须是不会被S影响的不变式
   左侧与M_IF一样, 根据程序进行主观判断使用M_WHILE还是M_IF. 
   ### progress_1_2_ser与final结合后得到progress_1_2_ser_final
   ![[assets/循环转换示意图.png]]
4. M_INIT
   初始化变量的初始值
   写在最上方
   ### init_ser与progress_1_2_ser_final结合得到search_in_matrix(程序名)
   ![[assets/矩阵搜索算法流程.png]]