---
description: >-
  GAML语言是GAMA平台使用的一种面向代理的编程语言，其基本构架与面向对象的编程语言JAVA类似，两者有许多共通之处，GAML语言在JAVA的技术上实现了一系列方便进行模拟的代理功能，如族、代理技能等等
---

# GAML语言基础

### 基本数据类型

GAML的基本数据类型 有**整数int 、浮点数float、字符串string**和**布尔值bool**四种。除了基本数据类型，GAML还内置了一系列图形类型（如**geometry**、**point** 、**path** 、**topology**等\)、集合类型（如**pair**、**list**、**matrix**、**map**等）以及代理类型（如**agent**、**species**等\)。同时GAML还支持使用关键字type，在代理族**species**中自定义不同的类型。更多关于数据类型的内容可以参考[官方文档](https://gama-platform.github.io/wiki/DataTypes)。

### 变量

在GAML语言中，声明一个变量需要显式的声明变量的数据类型+数据名。

```text
string text;                // 在GAML中'//'是注释，表示其后的内容不会被认为是代码                                               
text -< 'Hello World';      // '-<'是赋值操作，
int a -< 3;                 // GAML每一行代码都以';'结束
float b -< 4;               // GAML会对基础数据类型进行自动转换
bool c -< a=b;              // GAML中'='是判定是否相等，不是赋值
write(c);                   // 不在交互式命令行窗口时，可用write来输出变量的值
```

### 列表、矩阵与字典

GAML中，关键字list表示列表，列表可以直接显式地声明列表中的数据类型，也可以声明列表中的数据类型来构建一个多数据类型的列表。

```text
 list<int>  l_1 <- [5,4,9,8];     //此时要求列表中所有元素都是整数
 list l_2 <- [4,5,'o','j',[1,2]]; //此时列表中元素有不同的数据类型 
 int i <- length(l_1);            //返回列表长度
 string s <- l_2[2];              //返回l_2列表第三个元素
 int i <- l_2 at 0;               //返回l_2列表第一个元素
 int index <- l_2 index_of 5;     //返回l_2列表中5的索引值
 remove from:l_2 index:1;         //删除l_2列表中索引值为1的元素(5)
 remove item:'j' from:l_2;        //删除l_2列表中的'j'
 add item: 9 to: l_2 at: 2;       //向l_2列表索引2的位置添加元素9
 add 0 to: l_2;                   //向l_2列表末尾添加0
 put 2 in: l_2 at:0;              //向l_2列表开始添加2
 put 3 in: l_2 key:2;             //向l_2列表索引2的位置添加元素3
```

GAML中，matrix表示一个二维矩阵或者一个一维向量。

```text
matrix mat1 <- matrix([1,2,3,4,5]);   //通过matrix函数将列表转换为一维向量
matrix mat2 <- 0.0 as_matrix({10,5}); //构造一个10列5行的二维数组，其初始值为0
matrix mat3 <- matrix([[1,2,3],[4,5,6]]); //构造一个2列3行的二维数组
int a <- mat3[0,0];                 //返回mat3矩阵第1列第1行的元素              
```

GAML中，map表示一个存贮键\(key\)-值\(value\)对的字典，在字典中的每一对都表示为`key::value` 。

```text
map<string,rgb> color_per_type <- ["T"::#gamared,"A"::#gamagreen];   //将'T'、'A'分别对应红色、绿色
map<string,int> scale_count <- ["L"::50, "M"::40, "S"::10];          //将'L'、'M'、'S'分别对应50、40、10
```

### 数学运算

GAML支持大部分的数学符号的直接运算，如加\(+\)、减\(-\)、乘\(\*\)、除\(/\)、乘幂\(^\)等，此外还有余弦\(cos\)、正弦\(sin\)、正切\(tan\)、平方根\(sqrt\)、四舍五入\(round\)等运算。更多计算操作可以查看[官方文档](https://gama-platform.github.io/wiki/Operators)

```text
int a <- 5*3;
int a <- 5^2;              
int a <- int(5/3);        //取整
int a <- round(5/2);      //四舍五入
int a <- 5 mod 3;         //取余
float a <- sin(90);       //sin函数
int a <- rnd(100);        //随机数
```

### 逻辑运算

GAML逻辑运算符主要有 **and、or** 以及 **！**\(表示not\)，**！** 在使用时应该放在表达式的前面。

```text
bool a <- true or false;
bool b <- true and false;
bool c <- ! true;
```

### 比较运算

GAML语言中需要注意的时**“=”**表示相等关系，而不是像python里面的赋值，**“!=”**表示不相等关系，此外**“&lt;”、“&gt;”、“&lt;=”、“&gt;=”**都是基本的比较运算符。

```text
bool a <- 3>5;
bool a <- 3!=5;
```

### 条件语句

GAML基本的条件语句结构为if/else结构，条件包含在小括号\(\)内，执行语句包含在大括号{}内。

```text
int index <- rnd(100);
if (index <10){
    write 'index is smaller than 10';
}
else if (index>=10 and index<80){
    write 'index is bigger than 10 but smaller than 80';
}
else {
    write 'index is bigger than 80';
}
```

GAML还提供一种使用**？**来简化表达的条件语句。

```text
string a <- (1>3) ? "this is true" : "this is false";
//此句等价于
if (1>3){
    string a <- "this is true";
}
else{
    string a <- "this is false";
}
```

### 循环语句

GAML使用关键字**loop**来实现循环语句。

```text
loop times: 2 { write 'hello world';}   //循环两次
loop while: true{write 'hello world';}  //无限循环
loop i from: 0 to: 5 step:2 {write i;}  //输出0-2-4
loop i over: [0,2,5]{write i;}          //输出0-2-5
```



