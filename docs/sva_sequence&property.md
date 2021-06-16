## 文章目录
### 一、断言简介
1.1.断言分类——立即断言/并发断言
1.2.断言的语法结构层次
### 二、并发断言序列sequence
2.1. 关键字(sequence、property)与操作符( |=>、|->)
2.2. sequence的重复操作符——连续[*n]、非连续[=n]、跟随[->n]
2.2. sequence序列采样函数——$ rose、$ fell、$ past、$ stable、$ sampled
2.3. sequence序列操作符——and、intersect、or、first_match、within、throughout、ended
### 三、并发断言属性property
3.1. assert、assume、cover
3.2. disable iff 与not用法
3.3. 多时钟属性
### 四、代码示例

### 一、断言简介
   断言是对设计行为属性的描述。它使用描述性语言来描述了设计必须满足的属性。在仿真过程中，如果一个被描述的属性不是期望的那样，那么断言就将失败，或者在仿真过程中，出现了一个不该出现的属性，断言也将失败。
  使用断言的原因：原有的Verilog语言是一种过程性设计语言，在硬件设计过程中不能很好的描述时序行为。而SVA是一种描述性语言，可以很好的描述和控制复杂设计的时序的相关问题，代码简洁，易于维护。基于断言的验证方法（ABV）是对验证的增补，通过白盒验证的方式可以获取设计的意图，提供设计的可观察性，有利于指导验证工作，加快验证进度，保证验证的完备性。 除此之外，断言还可以对总线协议进行定义和验证（AHB/APB）等，相当于一个RTL代码的监视器。

#### 1.1.断言分类——立即断言/并发断言
  断言分为立即断言和并发断言。立即断言是非事件性的，断言属性类似if语句中的条件表达式，用于检查表达式的值是否为真，断言失败时必须采用相应的仿真行为，如下语法：

assert_info：assert(expression)
     $display("passed");       //expression为真时，执行语句
  else
     $display("failed");      //expression为假时，执行语句
1
2
3
4
  相比于立即断言，并发断言更为常用，并发断言基于时钟周期，用于描述一个跨时钟周期的行为，并发断言使用关键字property…endproperty描述事件，如下：

property  p_shakehand;                 //当行为属性p_shakehand中的条件request为真时，结果序列必须为真，否者序列失败
   @(posedge  clk)                     //符号“|=>”左侧的为原因序列，右侧为结果序列
   request |=> acknowledge ##1 data_enable ##1 done;      //2. 当request为1时，启动线程序列检查
endproperty
apshakehand:assert property(p_shakehand);       //1. assert关键字启动断言检查
1

#### 1.2.断言的语法结构层次
  SVA语法主要分为五个结构层次：

最底层——布尔表达式；
第二层——序列（sequence），其中可包含一些操作符，如##时隙延迟、重复操作符、序列操作符等，序列是一个封装格式，可在不同地方使用；
第三层——属性（property），重要的封装方式，可封装sequence，内部可定义蕴涵操作符（|->，|=>）；
第四层——断言指示层，即采用assert对特定的属性或者序列做行为检查，或者采用cover做覆盖率统计；
第五层——断言的最后封装，这一层可以通过module、program、interface来进行封装。
二、并发断言序列sequence
  并发断言基于时钟周期的，因而只有利用时钟周期采样的值才有效。下面以并发断言为主进行讲解。需要说明的是断言属性有七种，序列sequence只是其中之一（属性property包含有序列sequence、否定Negation、分离Disjunction、联合Conjunction、if…else…、implication、instantation）。

2.1. 关键字(sequence、property)与操作符( |=>、|->)
thread：线程是一组相关的事件序列（原因序列与结果序列），表示一种设计属性；
sequence：序列是描述一种信号时序关系的基本语句块；
property：属性用于封装各种sequence，可作为检查器、假设条件和覆盖率，对应关键字为assert、assume、cover；
  
|-> ： 表示起因序列与结果序列处于同一个时钟周期；只有当起因序列为真时才进行结果序列的检查，避免虚假错误的产生。
|=> ：表示结果序列处于起因序列的下一个时钟周期；等效于##1。
断言的建立过程与格式：
1. 编写序列sequence
2. 编写属性property
3. 编写断言assert或覆盖cover语句


sequence s1;       //sequence主要描述信号与信号之间的时序关系
  a ##1 b ##1 c;    //a为高，下一拍b为高，在下一拍c为高
endsequence

property p1;       //property主要将各种sequence进行封装
  s1;
endproperty

a1:assert property(@(posedge clk) a |-> p1 );    //关键字assert启动断言， |-> :表示起因序列和结果序列在同一个周期
1
2
3
4
5
6
7
8
9
#### 2.2. sequence的重复操作符——连续[*n]、非连续[=n]、跟随[->n]


重复操作符（连续）—— [*n]、 [*min：max] ，用法示例如下：
示例1：       a ##1 b[*3] ##1 c     等价于    a ##1 b ##1 b ##1 b ##1 c
示例2：       a ##1 b[*3：4] ##1 c     等价于    a ##1 b ##1 b ##1 b ##1 c 或者 a ##1 b ##1 b ##1 b ##1 b ##1 c
1
2


重复操作符1（非连续） —— [=n]、 [=min：max]， 示例如下
示例1：  a ##1 b[=2] ##1 c     含义：第一拍a为1，最后一排c为1，a与c之间b为1要出现2次，非连续出现。
示例1：  a ##1 b[=2：5] ##1 c     含义：第一拍a为1，最后一拍c为1，a与c之间b为1最少出现2次，最多出现5次，非连续出现。
1
2
重复操作符2（非连续） —— [->n]、 [->min：max]， 示例如下
示例1：a ##1 b[->2] ##1 c  含义：第一拍a为1，最后一排c为1，且*倒数第二拍b也为1*，a与c之间b为1要出现2次，非连续出现。
示例1：a ##1 b[->2：5] ##1 c  含义：第一拍a为1，最后一拍c为1，*倒数第二拍b也为1，a与c之间b为1最少出现2次，最多出现5次，非连续出现。
1
2
2.2. sequence序列采样函数——$ rose、$ fell、$ past、$ stable、$ sampled
$ rose：判断上升沿。如果表达式的值跳变为1，$rose函数返回真，否则返回假
$ fell：判断下降沿。如果表达式的值跳变为0，$fell函数返回真，否则返回假
$ past：返回当前时钟周期之前的一个时钟周期的表达式的采样值
$ stable：判断信号的稳定不变，如果表达式的值保持稳定不变，$ stable函数返回真，否则返回假
$ sampled：在时钟事件的最后时刻采样表达式的值
`define  data_end_exp  (data_phase && ((irdy==0)||($fell(stop))))         //声明宏定义
property  data_end_rule;
   @(posedge  mclk)
   `data_end_exp |-> ##[1:2] $rose(frame) ##1 $rose(irdy);
endproperty
1
2
3
4
5
    

#### 2.3. sequence序列操作符——and、intersect、or、first_match、within、throughout、ended


and操作符：表示两个序列是匹配的，具有相同的起点，但是他们的结束时间不一致，由长序列的结束点为准。

intersect交互操作符：用于描述两个序列匹配，长度一样，且结束时间必须一致。相当于and操作符的一个特例。

or操作符：用于描述两个序列至少有一个是匹配的，即至少有个序列为真，则操作为真。
first_match操作符：如果有多个匹配序列时，first_match操作符只返回第一个操作序列
throughout跨越序列：在假设条件为真时，一个序列才会发生正确的行为
sequence burst_rule1;
  @(posedge mclk)
  $fell(burst_mode) ##0
  (!burst_mode) throughout (##2 ((trdy==0)&&(irdy==0)) [*7]);
endsequence



ended关键字：可以测试任意序列是否到达结束点。用法：“序列名.ended”
### 三、并发断言属性property
3.1. assert、assume、cover
assert关键字：属性检查，相当于检查器
assume关键字：将属性当做假设条件，仿真时，验证环境必须满足属性要求，否者必须报错
cover关键字：收集设计行为的覆盖率。
   功能覆盖率分为面向数据覆盖率和面向控制的覆盖率，面向数据覆盖率可以通过编写覆盖组covergroup收集覆盖率，而面向控制的覆盖率用于检查行为序列，通过SVA编写行为序列，使用关键字cover收集覆盖率。

3.2. disable iff 与not用法
disable iff(expression)：当expression为真时，关闭property的序列检查，否则进行检查
not： 表示not后的序列不能出现
```
property  abc(a, b ,c);     //参数化
   disable iff(a==1);    //当a为1时，关闭属性检查
   not @clk (b ##1 c);   //not后的序列不能出现
endproperty
env_property：assert property(abc(rst, in, out)) pass_statement; else fail_statement;    
//statement语句是可选的，当属性为真，执行pass_statement，否则执行fail_statement
```
1
2
3
4
5
6
3.3. 多时钟属性
|-> ： 表示起因序列与结果序列起始点处于同一个时钟周期；
@(posedge clk0) s0 |-> @(posedge clk1) s1        //语法序列不合法，存在不同的时钟
@(posedge clk0) s0 |-> @(posedge clk0) s1        //正确
1
2
|=> ：表示结果序列处于起因序列的下一个时钟周期；
@(posedge clk0) sig0 |=> @(posedge clk1) sig1     //操作符"|=>"用于同步clk0与clk1的上升沿
1


### 四、代码示例
```
module sequence_demo();

  bit rst_n;
  bit clk;
  reg a,b,c;
  event e1;

initial begin
   forever #10 clk = ~clk;
end

initial begin
   rst_n = 1;
   #5;
   rst_n = 0;     //复位处理
   #5;
   rst_n = 1;     //撤销复位
   ->e1;   
end

initial begin
   $vcdpluson();     //该系统函数是为了使代码运行完毕之后，生成相应的波形文件
   @e1;
   a <= 0;
   b <= 0;
   c <= 0;
   repeat(3)begin
      @(posedge clk);
      a <= 1;
      @(posedge clk);
      a <= 0;
      b <= 1;
      @(posedge clk);
      a <= 0;
      b <= 0;
      c <= 1;
      @(posedge clk);
      c <= 0;
   end
      @(posedge clk);               
      @(posedge clk);
      @(posedge clk);
end

sequence s1;       //sequence主要描述信号与信号之间的时序关系
  a ##1 b ##1 c;    //a为高，下一拍b为高，在下一拍c为高
endsequence

property p1;       //property主要将各种sequence进行封装
  s1;
endproperty

a1:assert property(@(posedge clk) a |-> p1 );    //启动断言， |-> :表示起因序列和结果序列在同一个周期

endmodule

```

启动脚本编译后：使用Makefile实例三，略微修改all选项即可。


————————————————
版权声明：本文为CSDN博主「Mr.Marc」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_46022434/article/details/105469623
