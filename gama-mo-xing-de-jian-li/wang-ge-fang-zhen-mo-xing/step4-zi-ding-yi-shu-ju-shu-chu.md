# Step5:  自定义数据输出

除了在运行过程中呈现动态模拟窗口，GAMA还支持以其他形式输出模拟数据，包括代理检查器（Inspector）、数据监管器（Monitor）、图表、文件、图像等

### 数据检查器\(Inspector\)

GAMA内置了数据检查器，无需进行额外的设置。数据检查器包含两种检索信息的方式： **代理族浏览器（Species browser）** 和  **代理检查器（** **Agent inspector）**。

* **代理族浏览器（Species browser）**可通过点击模拟界面右上角的图标打开，代理族浏览器中，包含了同一代理族中各代理的所有属性信息。

![4.5.1 &#x4EE3;&#x7406;&#x6D4F;&#x89C8;&#x5668;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2810%29.png)

![4.5.2 &#x4EE3;&#x7406;&#x6D4F;&#x89C8;&#x5668;](../../.gitbook/assets/image%20%2811%29.png)

* **代理检查器（** **Agent inspector）**可通过在模拟界面右键选择某个代理，点击**Inspect** 查看某一代理的所有属性信息，并可以进行实时修改。

![4.5.3 &#x4EE3;&#x7406;&#x68C0;&#x67E5;&#x5668;](../../.gitbook/assets/image%20%2812%29.png)

### 数据监管器\(Monitor\)

数据监管器可以用来实时查看指定数据的输出，比如我们希望在运行过程中实时查看所有食草动物和食肉动物的数量，可以通过以下操作实现：

首先，在全局代理中添加全局变量分别表示食草动物数量和食肉动物数量：

```text
global {
	...
	//添加返回食草与食肉动物数量的全局参数
	int nb_preys -> {length (prey)};
	int nb_predators -> {length (predator)};
	...
}
```

> **length\(list\)** : 使用length函数将会返回列表的长度，这里分别返回当前食草动物与食肉动物族中代理的数量。

然后在实验设置的output中，添加数据监管器\(Monitor\)，监管nb\_preys、nb\_predators两个参数的变化。

```text
experiment prey_predator type: gui {

	output {
		//主窗口
		display main_display {
		...
		}
		//新增一个信息展示窗口
		display info_display {
		...
    }
    //添加数据监管器
    monitor "Number of preys" value: nb_preys;
    monitor "Number of predators" value: nb_predators;
    }
}
```

再次运行模拟，模拟界面将会出现Monitors窗口。

![4.5.4 &#x6570;&#x636E;&#x76D1;&#x7BA1;&#x5668;](../../.gitbook/assets/image%20%2819%29.png)

### 图表输出

如果希望通过更加直观的方式输出可视化的数据，GAMA支持输出折线图、拼图、直方图的输出。图标的输出定义在**实验设置\(experiment\)**中，使用**chart**关键字来构建不同的图表，接下来我们分别为捕食者模型添加折线图、饼图和直方图：

```text
    output {
        display main_display {
            ...
        }

        display info_display {
            ...
        }
        //创建一个新窗口显示图表，每五个循环刷新一次
        display Population_information refresh: every(5#cycles) {
            //创建名为‘Species evolution’的图表，类型为折线图，图表大小为宽1高0.5，图表左上角位置为0,0
            chart "Species evolution" type: series size: {1,0.5} position: {0, 0} {
                //填充数据，数据名为number_of_preys，数据值为nb_preys，显示为蓝色
                data "number_of_preys" value: nb_preys color: #blue;
                //填充数据，数据名为number_of_predator，数据值为nb_predators，显示为红色
                data "number_of_predator" value: nb_predators color: #red;
            }
            ...
        }
        ...
    }
}
```

> **cycles**: cycle是gama仿真模型中的时间度量，1个cycle表示仿真模型运行了一步，通常通过指定一天或者一个小时对应多少cycle来将仿真模型中模拟的时间和现实时间对应。

> **size:{1,0.5}**: 这里大小的表示，相当于将图表所在窗口的宽和高设为1，size:{1,0.5}表示图表的宽等于窗口宽，图表高等于窗口的一半。这种设定方式使得窗口改变时图表也会适应窗口跟着变化。同理，位置的表示也是一样的。

```text
//创建名为‘Prey Energy Distribution’的图表，类型为直方图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
chart "Prey Energy Distribution" type: histogram background: #lightgray size: {0.5,0.5} position: {0, 0.5} {
    //数据区间0-0.025 的值是食草动物中能量值小于等于0.25的数量,颜色为红色
    data "0-0.25" value: prey count (each.energy <= 0.25) color:#red;
    //数据区间0.25-0.5 的值是食草动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    data "0.25-0.5" value: prey count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    //数据区间0.5-0.75 的值是食草动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    data "0.5-0.75" value: prey count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    //数据区间0.75-1 的值是食草动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    data "0.75-1" value: prey count (each.energy > 0.75) color:#yellow;
    }
```

> list **count** condition**:**  计算列表list中符合condition条件的个体的数量。

```text
//创建名为‘Predator Energy Distribution’的图表，类型为饼图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
chart "Predator Energy Distribution" type: pie background: #lightgray size: {0.5,0.5} position: {0, 0.5} {
    //数据区间0-0.025 的值是食肉动物中能量值小于等于0.25的数量,颜色为红色
    data "0-0.25" value: predator count (each.energy <= 0.25) color:#red;
    //数据区间0.25-0.5 的值是食肉动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    data "0.25-0.5" value: predator count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    //数据区间0.5-0.75 的值是食肉动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    data "0.5-0.75" value: predator count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    //数据区间0.75-1 的值是食肉动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    data "0.75-1" value: predator count (each.energy > 0.75) color:#yellow;
    }
```

最终结果如下图所示：

![4.5.5 &#x56FE;&#x8868;&#x8F93;&#x51FA;](../../.gitbook/assets/image%20%2816%29.png)

### 文件输出

### 图像输出



